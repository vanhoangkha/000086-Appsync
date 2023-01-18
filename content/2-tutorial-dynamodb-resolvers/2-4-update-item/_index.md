---
title : "Updating data"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 2.4. </b> "
---
We’ll set up a new mutation to allow us to update object. We’ll do this using the UpdateItem DynamoDB operation.

#### Updating a post

1. Choose the **Schema** tab
- Add the below code to **Mutation** type to add a new **updatePost** mutation
- Then click **Save Schema**
```
    updatePost(
        id: ID!,
        author: String!,
        title: String!,
        content: String!,
        url: String!
    ): Post
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-1.png?featherlight=false&width=90pc)

2. In **Resolvers** pane on the right, find **updatePost** field on the Mutation type and then choose **Attach**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-2.png?featherlight=false&width=90pc)

3. Select **PostDynamoDBTable** for **Data source name**
- Paste the following content to **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "UpdateItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
    },
    "update" : {
        "expression" : "SET author = :author, title = :title, content = :content, #url = :url ADD version :one",
        "expressionNames": {
            "#url" : "url"
        },
        "expressionValues": {
            ":author" : $util.dynamodb.toDynamoDBJson($context.arguments.author),
            ":title" : $util.dynamodb.toDynamoDBJson($context.arguments.title),
            ":content" : $util.dynamodb.toDynamoDBJson($context.arguments.content),
            ":url" : $util.dynamodb.toDynamoDBJson($context.arguments.url),
            ":one" : { "N": 1 }
        }
    }
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-3.png?featherlight=false&width=90pc)

{{% notice note %}}
This resolver is using the DynamoDB UpdateItem, which is significantly different from the PutItem operation. Instead of writing the entire item, you’re just asking DynamoDB to update certain attributes. This is done using DynamoDB Update Expressions. You can see the "expression" section under "update".  It says to set the author, title, content and url attributes, and then increment the version field. The expression has placeholders that have names starting with a colon, which are then defined in the expressionValues field. Finally, DynamoDB has reserved words that cannot appear in the expression. For example, url is a reserved word, so to update the url field you can use name placeholders and define them in the expressionNames field.
{{% /notice %}}

4. Scroll down, paste the following content to **Configure the response mapping template**
```
$utils.toJson($context.result)
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-4.png?featherlight=false&width=90pc)

4. Scroll up, click **Save Resolver**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-5.png?featherlight=false&width=90pc)

5. Choose the **Queries** tab to call API to add new post
- Paste the following mutation to **Queries** pane, then click **Execute query** (the orange play button)
```
mutation updatePost {
  updatePost(
    id:"123"
    author: "A new author"
    title: "An updated author!"
    content: "Now with updated content!"
    url: "https://aws.amazon.com/appsync/"
  ) {
    id
    author
    title
    content
    url
    ups
    downs
    version
  }
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-6.png?featherlight=false&width=90pc)

6. The return result should look similar to the following
```
{
  "data": {
    "updatePost": {
      "id": "123",
      "author": "A new author",
      "title": "An updated author!",
      "content": "Now with updated content!",
      "url": "https://aws.amazon.com/appsync/",
      "ups": 1,
      "downs": 0,
      "version": 2
    }
  }
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-7.png?featherlight=false&width=90pc)

Once the updating did, the **version** attribute is incremented by one unit because we asked AWS AppSync and DynamoDB to add a unit to the **version** attribute each time we update.

When you use the updatePost mutation as above, there will be 2 problems:
- If you want to update just a single field, you have to update all of the fields.
- If two people are modifying the object, you could potentially lose information.
Follow the next steps to fix the above problem

8. Choose **Schema** tab
- Replace the code of the **updatePost** mutation with the following content:
- Then click **Save Schema**
```
    updatePost(
        id: ID!,
        author: String,
        title: String,
        content: String,
        url: String,
        expectedVersion: Int!
    ): Post
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-8.png?featherlight=false&width=90pc)

9. In the **Resolvers** pane on the right, find the **addPost** field on the Mutation tyoe, and then choose **Attach**

10. Paste the following content to **Configure the request mapping template**
- Then click **Save Resolver**
```
{
    "version" : "2017-02-28",
    "operation" : "UpdateItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
    },

    ## Set up some space to keep track of things you're updating **
    #set( $expNames  = {} )
    #set( $expValues = {} )
    #set( $expSet = {} )
    #set( $expAdd = {} )
    #set( $expRemove = [] )

    ## Increment "version" by 1 **
    $!{expAdd.put("version", ":one")}
    $!{expValues.put(":one", { "N" : 1 })}

    ## Iterate through each argument, skipping "id" and "expectedVersion" **
    #foreach( $entry in $context.arguments.entrySet() )
        #if( $entry.key != "id" && $entry.key != "expectedVersion" )
            #if( (!$entry.value) && ("$!{entry.value}" == "") )
                ## If the argument is set to "null", then remove that attribute from the item in DynamoDB **

                #set( $discard = ${expRemove.add("#${entry.key}")} )
                $!{expNames.put("#${entry.key}", "$entry.key")}
            #else
                ## Otherwise set (or update) the attribute on the item in DynamoDB **

                $!{expSet.put("#${entry.key}", ":${entry.key}")}
                $!{expNames.put("#${entry.key}", "$entry.key")}
                $!{expValues.put(":${entry.key}", { "S" : "${entry.value}" })}
            #end
        #end
    #end

    ## Start building the update expression, starting with attributes you're going to SET **
    #set( $expression = "" )
    #if( !${expSet.isEmpty()} )
        #set( $expression = "SET" )
        #foreach( $entry in $expSet.entrySet() )
            #set( $expression = "${expression} ${entry.key} = ${entry.value}" )
            #if ( $foreach.hasNext )
                #set( $expression = "${expression}," )
            #end
        #end
    #end

    ## Continue building the update expression, adding attributes you're going to ADD **
    #if( !${expAdd.isEmpty()} )
        #set( $expression = "${expression} ADD" )
        #foreach( $entry in $expAdd.entrySet() )
            #set( $expression = "${expression} ${entry.key} ${entry.value}" )
            #if ( $foreach.hasNext )
                #set( $expression = "${expression}," )
            #end
        #end
    #end

    ## Continue building the update expression, adding attributes you're going to REMOVE **
    #if( !${expRemove.isEmpty()} )
        #set( $expression = "${expression} REMOVE" )

        #foreach( $entry in $expRemove )
            #set( $expression = "${expression} ${entry}" )
            #if ( $foreach.hasNext )
                #set( $expression = "${expression}," )
            #end
        #end
    #end

    ## Finally, write the update expression into the document, along with any expressionNames and expressionValues **
    "update" : {
        "expression" : "${expression}"
        #if( !${expNames.isEmpty()} )
            ,"expressionNames" : $utils.toJson($expNames)
        #end
        #if( !${expValues.isEmpty()} )
            ,"expressionValues" : $utils.toJson($expValues)
        #end
    },

    "condition" : {
        "expression"       : "version = :expectedVersion",
        "expressionValues" : {
            ":expectedVersion" : $util.dynamodb.toDynamoDBJson($context.arguments.expectedVersion)
        }
    }
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-10.png?featherlight=false&width=90pc)
![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-11.png?featherlight=false&width=90pc)

11. Choose **Queries** to call an API call to update a post
- Paste the following mutation to **Queries** pane, then click **Execute query** (the orange play button)
```
mutation updatePost {
  updatePost(
    id:123
    title: "An empty story"
    content: null
    expectedVersion: 2
  ) {
    id
    author
    title
    content
    url
    ups
    downs
    version
  }
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-12.png?featherlight=false&width=90pc)

12. The return result should look similar to the following:
```
{
  "data": {
    "updatePost": {
      "id": "123",
      "author": "A new author",
      "title": "An empty story",
      "content": null,
      "url": "https://aws.amazon.com/appsync/",
      "ups": 1,
      "downs": 0,
      "version": 3
    }
  }
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-13.png?featherlight=false&width=90pc)

{{% notice note %}}
In this request, you asked AWS AppSync and DynamoDB to update the **title** and **content** field only. It left all the other fields alone (other than incrementing the version field)
{{% /notice %}}

13. Click **Execute query** button again, the return result should look similar to the following

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-14.png?featherlight=false&width=90pc)

The request fails because the condition expression evaluates to false:
- The first time you ran the request, the value of the **version** field of the post in DynamoDB was 2, which matched the **expectedVersion** argument. The request succeeded, which meant the version field was incremented in DynamoDB to 3.
- The second time you ran the request, the value of the **version** field of the post in DynamoDB was 3, which did not match the **expectedVersion** argument.

#### Updating the number of votes
Chúng ta sẽ tạo hai mutation mới để cập nhật trường **ups** và **downs** để ghi lại lượt ủng hộ hay phản đối cho bài đăng

1. Chọn tab **Schema**
- Add the below code to **Mutation** type to add 2 new mutations are **upvotePost** and **downvotePost**
```
    upvotePost(id: ID!): Post
    downvotePost(id: ID!): Post
```
- Then click **Save Schema**
- Next, in **Resolvers** pane on the right side, find **upvotePost** field on Mutation type, and then choose **Attach**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-15.png?featherlight=false&width=90pc)

2. Select **PostDynamoDBTable** for **Data source name**
- Paste the following content to **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "UpdateItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
    },
    "update" : {
        "expression" : "ADD ups :plusOne, version :plusOne",
        "expressionValues" : {
            ":plusOne" : { "N" : 1 }
        }
    }
}
```
- Paste the following content to **Configure the response mapping template**
```
$utils.toJson($context.result)
```
- Click **Save Resolver**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-16.png?featherlight=false&width=90pc)

3. Choose **Schema** tab
- In **Resolvers** pane on the right side, find **downvotePost** field on Mutation type, and then choose **Attach**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-17.png?featherlight=false&width=90pc)

4. Select **PostDynamoDBTable** for **Data source name**
- Paste the following content to **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "UpdateItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
    },
    "update" : {
        "expression" : "ADD downs :plusOne, version :plusOne",
        "expressionValues" : {
            ":plusOne" : { "N" : 1 }
        }
    }
}
```
- Paste the following content to **Configure the response mapping template**
```
$utils.toJson($context.result)
```
- Click **Save Resolver**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-18.png?featherlight=false&width=90pc)

5. Choose the **Queries** tab.
- Paste the following mutation to **Queries** pane, then click **Execute query** (the orange play button)
```
mutation votePost {
  upvotePost(id:123) {
    id
    author
    title
    content
    url
    ups
    downs
    version
  }
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-20.png?featherlight=false&width=90pc)

6. The return result should look similar to the following:
```
{
  "data": {
    "upvotePost": {
      "id": "123",
      "author": "A new author",
      "title": "An empty story",
      "content": null,
      "url": "https://aws.amazon.com/appsync/",
      "ups": 2,
      "downs": 0,
      "version": 4
    }
  }
}
```

7. Chane query to **downvotePost**
```
mutation votePost {
  downvotePost(id:123) {
    id
    author
    title
    content
    url
    ups
    downs
    version
  }
}
```
- CLick **Execute query** (the orange play button)

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-21.png?featherlight=false&width=90pc)

8. CLick **Execute query** (the orange play button) again,  you should see the downs and version field incrementing by 1 each time you execute the query.

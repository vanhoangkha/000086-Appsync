---
title : "Deleting data"
date :  "`r Sys.Date()`" 
weight : 5
chapter : false
pre : " <b> 2.5. </b> "
---
Next we will create a new mutation to delete data in DynamoDB

1. Choose the **Schema** tab
- Add the below code to **Mutation** type to add a new **deletePost** mutation
```
    deletePost(id: ID!, expectedVersion: Int): Post
```
- Then click **Save Schema**
- In **Resolvers** pane on the right, find **deletePost** field on the Mutation type and then choose **Attach**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-5-delete-item-1.png?featherlight=false&width=90pc)

2. Select **PostDynamoDBTable** for **Data source name**
- Paste the following content to **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "DeleteItem",
    "key": {
        "id": $util.dynamodb.toDynamoDBJson($context.arguments.id)
    }
    #if( $context.arguments.containsKey("expectedVersion") )
        ,"condition" : {
            "expression"       : "attribute_not_exists(id) OR version = :expectedVersion",
            "expressionValues" : {
                ":expectedVersion" : $util.dynamodb.toDynamoDBJson($context.arguments.expectedVersion)
            }
        }
    #end
}
```
- Paste the following content to **Configure the response mapping template**
```
$utils.toJson($context.result)
```
- Then click **Save Resolver**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-5-delete-item-2.png?featherlight=false&width=90pc)

4. Choose the **Queries** tab
- Paste the following mutation to **Queries** pane, then click **Execute query** (the orange play button)
```
mutation deletePost {
  deletePost(id:123) {
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

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-5-delete-item-3.png?featherlight=false&width=90pc)

5. Click **Execute query** (the orange play button) again, the return value is **NULL** because you deleted it in the previous step

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-5-delete-item-4.png?featherlight=false&width=90pc)

6. aste the following mutation to **Queries** pane, then click **Execute query** (the orange play button) to add new post
```
mutation addPost {
  addPost(
    id:123
    author: "AUTHORNAME"
    title: "Our second post!"
    content: "A new post."
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

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-5-delete-item-5.png?featherlight=false&width=90pc)

7. aste the following mutation to **Queries** pane, then click **Execute query** (the orange play button) to delete the post with **expectedVersion** is 9999

```
mutation deletePost {
  deletePost(
    id:123
    expectedVersion: 9999
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

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-5-delete-item-6.png?featherlight=false&width=90pc)

The request failed because the condition expression evaluates to false: the value for version of the post in DynamoDB does not match the expectedValue specified in the arguments. The current value of the object is returned in the data field in the errors section of the GraphQL response.

8. Change **expectedVersion** value to 1, then click **Execute query** (the orange play button)

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-5-delete-item-7.png?featherlight=false&width=90pc)

The request was successful and the post has been deleted from DynamoDB
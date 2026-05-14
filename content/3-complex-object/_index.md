---
title : "Create complex object"
date: 2024-01-01
weight : 3
chapter : false
pre : " <b> 3. </b> "
---
In the previous sections, we created objects according to the key-value model. In this part we will create more complex objects with AWS AppSyncDynamoDB such as sets, lists, and maps.

1. Choose **Schema** tab
- Add **tags** attribute for Post object
```
  type Post {
  id: ID!
  author: String
  title: String
  content: String
  url: String
  ups: Int!
  downs: Int!
  version: Int!
  tags: [String!]
}
```
- Add below line in **Query** type to add a new query as **allPostsByTag**
```
  allPostsByTag(tag: String!, count: Int, nextToken: String): PaginatedPosts!
```
- Add two new mutations - addTag and removeTag - to type **Mutation**
```
type Mutation {
  addTag(id: ID!, tag: String!): Post
  removeTag(id: ID!, tag: String!): Post
  .....
}
```
- Click **Save Schema**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-1.png?featherlight=false&width=90pc)

2. In **Resolvers** pane on the right, find **allPostsByTag** field on the Query type and then choose **Attach**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-2.png?featherlight=false&width=50pc)

3. Select **PostDynamoDBTable** for **Data source name**
- Paste the following content to **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "Scan",
    "filter": {
      "expression": "contains (tags, :tag)",
        "expressionValues": {
          ":tag": $util.dynamodb.toDynamoDBJson($context.arguments.tag)
        }
    }
    #if( ${context.arguments.count} )
        ,"limit": $util.toJson($context.arguments.count)
    #end
    #if( ${context.arguments.nextToken} )
        ,"nextToken": $util.toJson($context.arguments.nextToken)
    #end
}
```

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-3.png?featherlight=false&width=90pc)

- Paste the following content to **Configure the response mapping template**
```
{
    "posts": $utils.toJson($context.result.items)
    #if( ${context.result.nextToken} )
        ,"nextToken": $util.toJson($context.result.nextToken)
    #end
}
```

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-4.png?featherlight=false&width=90pc)

- Click **Save Resolver**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-5.png?featherlight=false&width=90pc)

4. Choose **Schema** tab
- In **Resolvers** pane on the right, find **addTag** field on the Query type, then choose **Attach**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-6.png?featherlight=false&width=90pc)

5. Select **PostDynamoDBTable** for **Data source name**
- Paste the following content to **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "UpdateItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
    },
    "update" : {
        "expression" : "ADD tags :tags, version :plusOne",
        "expressionValues" : {
            ":tags" : { "SS": [ $util.toJson($context.arguments.tag) ] },
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

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-7.png?featherlight=false&width=90pc)

6. Choose **Schema** tab
- In **Resolvers** pane on the right, find **removeTag** on the Query type, then choose **Attach**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-8.png?featherlight=false&width=90pc)

7. Select **PostDynamoDBTable** for **Data source name**
- Paste the following content to **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "UpdateItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
    },
    "update" : {
        "expression" : "DELETE tags :tags ADD version :plusOne",
        "expressionValues" : {
            ":tags" : { "SS": [ $util.toJson($context.arguments.tag) ] },
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

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-9.png?featherlight=false&width=90pc)

8. Choose **Queries** tab
- Paste the following mutation to **Queries** pane
```
query allPostsByAuthor {
  allPostsByAuthor(
    author: "Nadia"
  ) {
    posts {
      id
      title
    }
    nextToken
  }
}
```
- Then click **Execute query** (the orange play button)

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-10.png?featherlight=false&width=90pc)

9. Paste the following script to add a tag for the above returned object, then press **Execute query** (the orange button)
```
mutation addTag {
  addTag(id:10 tag: "dog") {
    id
    title
    tags
  }
}
```


![CreateGetPostByTag](/images/3-complex-object/3-complex-object-11.png?featherlight=false&width=90pc)

The object returned is a list of **tags** with the value "dog"
10. Change the value of the tag to **puppy** and then click **Execute query** (the orange button)

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-12.png?featherlight=false&width=90pc)

Returns the object that has been added "puppy" to the list **tags**

11. Paste the following script to remove a tag, then press **Execute query** (the orange button)
```
mutation removeTag {
  addTag(id:10 tag: "puppy") {
    id
    title
    tags
  }
}
```

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-13.png?featherlight=false&width=90pc)

12. Paste the following script to get all posts tagged **dog**, then press **Execute query** (orange button)
```
query allPostsByTag {
  allPostsByTag(tag: "dog") {
    posts {
      id
      title
      tags
    }
    nextToken
  }
}
```

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-14.png?featherlight=false&width=90pc)

13. Next, add the ability to add comments to posts. This will be modeled as a list of map objects on the Post object in DynamoDB.
14. Choose **Schema** tab
- Add a new **Comment** type
```
type Comment {
    author: String!
    comment: String!
}
```
- Add new **comments** field for **Post** type
```
type Post {
  id: ID!
  author: String
  title: String
  content: String
  url: String
  ups: Int!
  downs: Int!
  version: Int!
  tags: [String!]
  comments: [Comment!]
}
```
- Add the following line to **Mutation** type to add a new **addComment** mutation
```
  addComment(id: ID!, author: String!, comment: String!): Post
```
- Click **Save Schema**, then click **Attach** in **Revolvers** pane of **addComment** mutation

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-15.png?featherlight=false&width=90pc)

15. Select **PostDynamoDBTable** for **Data source name**
- Paste the following content to **Configure the request mapping template**
```
{
  "version" : "2017-02-28",
  "operation" : "UpdateItem",
  "key" : {
    "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
  },
  "update" : {
    "expression" : "SET comments = list_append(if_not_exists(comments, :emptyList), :newComment) ADD version :plusOne",
    "expressionValues" : {
      ":emptyList": { "L" : [] },
      ":newComment" : { "L" : [
        { "M": {
          "author": $util.dynamodb.toDynamoDBJson($context.arguments.author),
          "comment": $util.dynamodb.toDynamoDBJson($context.arguments.comment)
          }
        }
      ] },
      ":plusOne" : $util.dynamodb.toDynamoDBJson(1)
    }
  }
}
```
- Paste the following content to **Configure the response mapping template**
```
$utils.toJson($context.result)
```
- Then click **Save Resolver**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-16.png?featherlight=false&width=90pc)

16. Choose **Queries** tab
- Paste the following mutation to **Queries** pane, then click **Execute query** (the orange button)

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-17.png?featherlight=false&width=90pc)

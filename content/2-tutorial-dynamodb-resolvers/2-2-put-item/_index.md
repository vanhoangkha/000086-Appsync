---
title : "Writing data"
date: 2024-01-01
weight : 2
chapter : false
pre : " <b> 2.2. </b> "
---
After AWS AppSync is aware of the DynamoDB table, we can link it to individual queries and mutations by defining Resolvers. In this part we will create addPost resolver of Mutation type, allowing us to create posts in the AppSyncTutorial-Post table.

1. Choose **Schema** tab
- In the **Resolvers** pane on the right, find the **addPost** field on the Mutation tyoe, and then choose **Attach**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-2-put-item-1.png?featherlight=false&width=90pc)

2. Select **PostDynamoDBTable** for **Data source name**
- Paste the following content to **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "PutItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
    },
    "attributeValues" : {
        "author" : $util.dynamodb.toDynamoDBJson($context.arguments.author),
        "title" : $util.dynamodb.toDynamoDBJson($context.arguments.title),
        "content" : $util.dynamodb.toDynamoDBJson($context.arguments.content),
        "url" : $util.dynamodb.toDynamoDBJson($context.arguments.url),
        "ups" : { "N" : 1 },
        "downs" : { "N" : 0 },
        "version" : { "N" : 1 }
    }
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-2-put-item-2.png?featherlight=false&width=90pc)

{{% notice note %}}
A **Type** is specified on all the keys and attribute values. For example, you set the author field to { "S" : "${context.arguments.author}" }. The S part indicates to AWS AppSync and DynamoDB that the value will be a string value, and the value is taken from the **author** argument passed by the user.
{{% /notice %}}

3. Scroll down, paste the following content to **Configure the response mapping template**
```
$utils.toJson($context.result)
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-2-put-item-3.png?featherlight=false&width=90pc)

{{% notice note %}}
Because the shape of the data in the AppSyncTutorial-Post table exactly matches the shape of the Post type in GraphQL, the response mapping template just passes the results straight through.
{{% /notice %}}

4. Scroll up, click **Save Resolver**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-2-put-item-4.png?featherlight=false&width=90pc)

5. Choose the **Queries** tab to call API to add new post
- Paste the following mutation to **Queries** pane, then click **Execute query** (the orange play button)
```
mutation addPost {
  addPost(
    id: 123
    author: "AUTHORNAME"
    title: "Our first post!"
    content: "This is our first post."
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

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-2-put-item-5.png?featherlight=false&width=90pc)

6. The return result should look similar to the following
```
{
  "data": {
    "addPost": {
      "id": "123",
      "author": "AUTHORNAME",
      "title": "Our first post!",
      "content": "This is our first post.",
      "url": "https://aws.amazon.com/appsync/",
      "ups": 1,
      "downs": 0,
      "version": 1
    }
  }
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-2-put-item-6.png?featherlight=false&width=90pc)

7. Open [DynamoDB console]()

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-2-put-item-7.png?featherlight=false&width=90pc)

8. Choose **Explore items** tab
- Choose **AppSyncTutorial-Post** table, a new post has been written.

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-2-put-item-8.png?featherlight=false&width=90pc)

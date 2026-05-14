---
title : "Reading data"
date: 2024-01-01
weight : 3
chapter : false
pre : " <b> 2.3. </b> "
---
In this step we will retrieve the data in the DynamoDB table with the key passed in.

1. hoose the Schema tab
- In the **Resolvers** pane on the right, find the **getPost** field on the Mutation tyoe, and then choose **Attach**

![CreateGetResolvers](/images/2-tutorial-dynamodb-resolvers/2-3-get-item-1.png?featherlight=false&width=90pc)

2. Select **PostDynamoDBTable** for **Data source name**
- Paste the following content to **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "GetItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($ctx.args.id)
    }
}
```

![CreateGetResolvers](/images/2-tutorial-dynamodb-resolvers/2-3-get-item-2.png?featherlight=false&width=90pc)

3. Paste the following content to **Configure the response mapping template**, then click **Save Resolver**
```
$utils.toJson($context.result)
```

![CreateGetResolvers](/images/2-tutorial-dynamodb-resolvers/2-3-get-item-3.png?featherlight=false&width=90pc)

4. Choose the Queries tab
- Paste the following mutation to **Queries** pane, then click **Execute query** (the orange play button)
```
query getPost {
  getPost(id:123) {
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

![CreateGetResolvers](/images/2-tutorial-dynamodb-resolvers/2-3-get-item-4.png?featherlight=false&width=90pc)

5. The return result should look similar to the following
```
{
  "data": {
    "getPost": {
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

![CreateGetResolvers](/images/2-tutorial-dynamodb-resolvers/2-3-get-item-5.png?featherlight=false&width=90pc)


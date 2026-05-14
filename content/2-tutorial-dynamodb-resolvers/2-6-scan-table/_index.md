---
title : "Scanning table"
date: 2024-01-01
weight : 6
chapter : false
pre : " <b> 2.6. </b> "
---
In this part we will scan all the data in the DynamoDB table.

1. Choose the **Schema** tab
- Add the below code to **Query** type to add a new **allPost** query
```
    allPost(count: Int, nextToken: String): PaginatedPosts!
```
- Add a new **PaginationPosts** type
```
type PaginatedPosts {
    posts: [Post!]!
    nextToken: String
}
```
- Then click **Save Schema**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-1.png?featherlight=false&width=90pc)

2. Next, in **Resolvers** pane on the right, find **allPost** field on the Query type and then choose **Attach**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-2.png?featherlight=false&width=90pc)

3. Select **PostDynamoDBTable** for **Data source name**
- Paste the following content to **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "Scan"
    #if( ${context.arguments.count} )
        ,"limit": $util.toJson($context.arguments.count)
    #end
    #if( ${context.arguments.nextToken} )
        ,"nextToken": $util.toJson($context.arguments.nextToken)
    #end
}
```
{{% notice note %}}
This resolver has two optional arguments: count, which specifies the maximum number of items to return in a single call, and nextToken, which can be used to retrieve the next set of result
{{% /notice %}}
- Paste the following content to **Configure the response mapping template**
```
{
    "posts": $utils.toJson($context.result.items)
    #if( ${context.result.nextToken} )
        ,"nextToken": $util.toJson($context.result.nextToken)
    #end
}
```
{{% notice note %}}
This response mapping template is different from all the others so far. The result of the allPost query is a PaginatedPosts, which contains a list of posts and a pagination token. The shape of this object is different to what is returned from the AWS AppSync DynamoDB Resolver
{{% /notice %}}
- Sau đó ấn **Save Resolver**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-3.png?featherlight=false&width=90pc)

4. Choose **Queries** tab
- Paste the following mutation to **Queries** pane
```
mutation addPost {
  post1: addPost(id:1 author: "AUTHORNAME" title: "A series of posts, Volume 1" content: "Some content" url: "https://aws.amazon.com/appsync/" ) { title }
  post2: addPost(id:2 author: "AUTHORNAME" title: "A series of posts, Volume 2" content: "Some content" url: "https://aws.amazon.com/appsync/" ) { title }
  post3: addPost(id:3 author: "AUTHORNAME" title: "A series of posts, Volume 3" content: "Some content" url: "https://aws.amazon.com/appsync/" ) { title }
  post4: addPost(id:4 author: "AUTHORNAME" title: "A series of posts, Volume 4" content: "Some content" url: "https://aws.amazon.com/appsync/" ) { title }
  post5: addPost(id:5 author: "AUTHORNAME" title: "A series of posts, Volume 5" content: "Some content" url: "https://aws.amazon.com/appsync/" ) { title }
  post6: addPost(id:6 author: "AUTHORNAME" title: "A series of posts, Volume 6" content: "Some content" url: "https://aws.amazon.com/appsync/" ) { title }
  post7: addPost(id:7 author: "AUTHORNAME" title: "A series of posts, Volume 7" content: "Some content" url: "https://aws.amazon.com/appsync/" ) { title }
  post8: addPost(id:8 author: "AUTHORNAME" title: "A series of posts, Volume 8" content: "Some content" url: "https://aws.amazon.com/appsync/" ) { title }
  post9: addPost(id:9 author: "AUTHORNAME" title: "A series of posts, Volume 9" content: "Some content" url: "https://aws.amazon.com/appsync/" ) { title }
}
```
- Then click **Execute query** (the orange play button), select **allPost**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-5.png?featherlight=false&width=90pc)

5. Copy **nextToken** value after return

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-6.png?featherlight=false&width=90pc)

6. Add **nextToken** field to **allPost** query, hen assign the value you just copied to it

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-7.png?featherlight=false&width=90pc)

7. Click **Execute query** (the orange play button), select **allPost**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-8.png?featherlight=false&width=90pc)

So you have scanned the entire table each time returning the number you want.
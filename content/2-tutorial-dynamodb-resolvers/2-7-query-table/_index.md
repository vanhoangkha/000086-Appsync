---
title : "Querying table"
date: 2024-01-01
weight : 7
chapter : false
pre : " <b> 2.7. </b> "
---
When you want to query some data by certain attribute then this section will help you.

1. Choose the **Schema** tab
- Add the below code to **Query** type to add a new **allPostsByAuthor** mutation
```
    allPostsByAuthor(author: String!, count: Int, nextToken: String): PaginatedPosts!
```
- Then click **Save Schema**

![CreateQueryResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-1.png?featherlight=false&width=90pc)

2. In **Resolvers** pane on the right, find **allPostsByAuthor** field on the Mutation type and then choose **Attach**

![CreateQueryResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-2.png?featherlight=false&width=90pc)

3. Select **PostDynamoDBTable** for **Data source name**
- Paste the following content to **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "Query",
    "index" : "author-index",
    "query" : {
      "expression": "author = :author",
        "expressionValues" : {
          ":author" : $util.dynamodb.toDynamoDBJson($context.arguments.author)
        }
    }
    #if( ${context.arguments.count} )
        ,"limit": $util.toJson($context.arguments.count)
    #end
    #if( ${context.arguments.nextToken} )
        ,"nextToken": "${context.arguments.nextToken}"
    #end
}
```

![CreateQueryResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-3.png?featherlight=false&width=90pc)

- Paste the following content to **Configure the response mapping template**
```
{
    "posts": $utils.toJson($context.result.items)
    #if( ${context.result.nextToken} )
        ,"nextToken": $util.toJson($context.result.nextToken)
    #end
}
```

![CreateQueryResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-4.png?featherlight=false&width=90pc)

- Click **Save resolver**

![CreateQueryResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-5.png?featherlight=false&width=90pc)

4. Choose **Queries** tab
- Paste the following mutation to **Queries** pane
```
mutation addPost {
  post1: addPost(id:10 author: "Nadia" title: "The cutest dog in the world" content: "So cute. So very, very cute." url: "https://aws.amazon.com/appsync/" ) { author, title }
  post2: addPost(id:11 author: "Nadia" title: "Did you know...?" content: "AppSync works offline?" url: "https://aws.amazon.com/appsync/" ) { author, title }
  post3: addPost(id:12 author: "Steve" title: "I like GraphQL" content: "It's great" url: "https://aws.amazon.com/appsync/" ) { author, title }
}
```
- Click **Execute query** (the orange play button)

![CreateQueryResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-6.png?featherlight=false&width=90pc)

5. Paste the following mutation to **Queries** pane
```
query allPostsByAuthor {
  allPostsByAuthor(author: "Nadia") {
    posts {
      id
      title
    }
    nextToken
  }
}
```
- Then click **Execute query** (the orange play button), returned results are posts by author **Nadia**

![CreateQueryResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-7.png?featherlight=false&width=90pc)

6. Change **auhtor** value  **AUTHORNAME** and add **count: 5**
- Click **Execute query** (the orange play button)

![CreateQueryResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-8.png?featherlight=false&width=90pc)

7. Copy **nextToken** in the return result as input to the query
- Click **Execute query** (the orange play button), return the next data

![CreateQueryResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-9.png?featherlight=false&width=90pc)

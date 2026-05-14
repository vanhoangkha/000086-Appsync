---
title : "Truy vấn dữ liệu"
date: 2024-01-01
weight : 7
chapter : false
pre : " <b> 2.7. </b> "
---
Khi bạn muốn truy vấn một vài dữ liệu theo thuộc tính nào đó thì phần này sẽ giúp bạn.

1. Chọn tab **Schema**
- Thêm đoạn code dưới đây vào kiểu **Query** để thêm một truy vấn mới là **allPostsByAuthor**
```
    allPostsByAuthor(author: String!, count: Int, nextToken: String): PaginatedPosts!
```
- Sau đó ấn **Save Schema**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-1.png?featherlight=false&width=90pc)

2. Trong ngăn **Resolvers** ở bên phải, tìm trường **allPostsByAuthor** trên kiểu Mutation, sau đó chọn **Attach**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-2.png?featherlight=false&width=90pc)

3. Chọn **PostDynamoDBTable** cho mục **Data source name**
- Dán nội dung dưới đây vào mục **Configure the request mapping template**
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

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-3.png?featherlight=false&width=90pc)

- Dán nội dung dưới đây vào mục **Configure the response mapping template**
```
{
    "posts": $utils.toJson($context.result.items)
    #if( ${context.result.nextToken} )
        ,"nextToken": $util.toJson($context.result.nextToken)
    #end
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-4.png?featherlight=false&width=90pc)

- Ấn **Save resolver**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-5.png?featherlight=false&width=90pc)

4. Chọn tab **Queries**
- Dán đoạn script dưới đây vào phần **Queries**
```
mutation addPost {
  post1: addPost(id:10 author: "Nadia" title: "The cutest dog in the world" content: "So cute. So very, very cute." url: "https://aws.amazon.com/appsync/" ) { author, title }
  post2: addPost(id:11 author: "Nadia" title: "Did you know...?" content: "AppSync works offline?" url: "https://aws.amazon.com/appsync/" ) { author, title }
  post3: addPost(id:12 author: "Steve" title: "I like GraphQL" content: "It's great" url: "https://aws.amazon.com/appsync/" ) { author, title }
}
```
- Ấn Sau đó ấn **Execute query** (nút màu cam)

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-6.png?featherlight=false&width=90pc)

5. Dán đoạn script dưới đây vào phần **Queries**
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
- Sau đó ấn **Execute query** (nút màu cam), kết quả trả về là các bài đăng của tác giả **Nadia**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-7.png?featherlight=false&width=90pc)

6. Đổi giá trị của trường **auhtor** thành **AUTHORNAME** và thêm trường **count: 5**
- Ấn **Execute query** (nút màu cam)

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-8.png?featherlight=false&width=90pc)

7. Sao chép trường **nextToken** ở kết quả trả về làm đầu vào cho truy vấn
- Ấn **Execute query** (nút màu cam), trả về dữ liệu tiếp theo

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-7-query-table-9.png?featherlight=false&width=90pc)

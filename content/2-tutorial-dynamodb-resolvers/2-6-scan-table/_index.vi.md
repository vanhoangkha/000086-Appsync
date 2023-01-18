---
title : "Quét toàn bảng"
date :  "`r Sys.Date()`" 
weight : 6
chapter : false
pre : " <b> 2.6. </b> "
---
Trong bước này chúng ta sẽ quét toàn bộ dữ liệu trong bảng của DynamoDB.

1. Chọn tab **Schema**
- Thêm dòng dưới đây vào kiểu **Query** để thêm một truy vấn mới là **allPost**
```
    allPost(count: Int, nextToken: String): PaginatedPosts!
```
- Thêm kiểu **PaginationPosts** mới
```
type PaginatedPosts {
    posts: [Post!]!
    nextToken: String
}
```
- Sau đó ấn **Save Schema**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-1.png?featherlight=false&width=90pc)

2. Tiếp theo trong ngăn **Resolvers** ở bên phải, tìm trường **allPost** trên kiểu Query, sau đó chọn **Attach**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-2.png?featherlight=false&width=90pc)

3. Chọn **PostDynamoDBTable** cho mục **Data source name**
- Dán nội dung dưới đây vào mục **Configure the request mapping template**
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
Resolver nầy có 2 đối số đầu vào là count - chỉ định số lượng mục tối đa sẽ trả về trong một lệnh gọi và nextToken - có thể sử dụng để truy xuất kết quả tiếp theo.
{{% /notice %}}
- Dán nội dung dưới đây vào mục **Configure the response mapping template**
```
{
    "posts": $utils.toJson($context.result.items)
    #if( ${context.result.nextToken} )
        ,"nextToken": $util.toJson($context.result.nextToken)
    #end
}
```
{{% notice note %}}
Kết quả truy vấn trả về là một PaginatedPosts, chứa danh sách các bài đăng và pagination token. Shape của đối tượng này khác với những gì được trả về từ AWS AppSync DynamoDB Resolver.
{{% /notice %}}
- Sau đó ấn **Save Resolver**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-3.png?featherlight=false&width=90pc)

4. Chọn tab **Queries**
- Thêm đoạn script dưới đây vào phần **Queries**
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
- Sau đó ấn **Execute query** (nút màu cam), chọn **allPost**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-5.png?featherlight=false&width=90pc)

5. Sao chép giá trị của trường **nextToken** được trả về

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-6.png?featherlight=false&width=90pc)

6. Thêm trường **nextToken** vào câu lệnh truy vấn **allPost**, sau đó gán giá trị mà bạn vừa sao chép cho nó

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-7.png?featherlight=false&width=90pc)

7. Ấn **Execute query** (nút màu cam), chọn **allPost**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-6-scan-table-8.png?featherlight=false&width=90pc)

Vậy là bạn đã quét toàn bộ bảng mỗi lần trả về số lượng mà bạn mong muốn.
---
title : "Xoá dữ liệu"
date :  "`r Sys.Date()`" 
weight : 5
chapter : false
pre : " <b> 2.5. </b> "
---
Tiếp theo chúng ta sẽ tạo một mutation mới để xoá dữ liệu trong DynamoDB

1. Chọn tab **Schema**
- Thêm đoạn code dưới đây vào kiểu **Mutation** để thêm một mutation mới là **deletePost**
```
    deletePost(id: ID!, expectedVersion: Int): Post
```
- Sau đó ấn **Save Schema**
- Tiếp theo trong ngăn **Resolvers** ở bên phải, tìm trường **deletePost** trên kiểu Mutation, sau đó chọn **Attach**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-5-delete-item-1.png?featherlight=false&width=90pc)

2. Chọn **PostDynamoDBTable** cho mục **Data source name**
- Dán nội dung dưới đây vào mục **Configure the request mapping template**
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
- Dán nội dung dưới đây vào mục **Configure the response mapping template**
```
$utils.toJson($context.result)
```
- Sau đó ấn **Save Resolver**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-5-delete-item-2.png?featherlight=false&width=90pc)

4. Chọn tab **Queries**
- Dán đoạn script dưới đây vào phần **Queries**, sau đó ấn **Execute query** (nút màu cam)
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

5. Ấn **Execute query** (nút màu cam) lần nữa, kết quả trả về **NULL** vì bạn đã xoá ở bước trên rồi

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-5-delete-item-4.png?featherlight=false&width=90pc)

6. Dán đoạn script dưới đây vào phần **Queries**, sau đó ấn **Execute query** (nút màu cam) để thêm một bài đăng mới
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

7. Dán đoạn script dưới đây vào phần **Queries**, sau đó ấn **Execute query** (nút màu cam) để xoá bài đăng có **expectedVersion** là 9999

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

Yêu cầu không thành công vì biểu thức điều kiện đánh giá là sai: giá trị cho version của bài đăng trong DynamoDB không khớp với expectedVersion được chỉ định trong các đối số. Giá trị hiện tại của đối tượng được trả về trong trường dữ liệu trong phần lỗi của phản hồi GraphQL.

8. Chỉnh lại giá trị của **expectedVersion** thành 1, sau đó ấn **Execute query** (nút màu cam)

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-5-delete-item-7.png?featherlight=false&width=90pc)

Yêu cầu đã thành công và bài đăng đã được xoá khỏi DynamoDB
---
title : "Ghi dữ liệu"
date: 2024-01-01
weight : 2
chapter : false
pre : " <b> 2.2. </b> "
---
Sau khi AWS AppSync nhận biết được bảng DynamoDB, chúng ta có thể truy vấn hoặc thay đổi nó bằng cách xác định các Resolver. Trong bước này chúng ta sẽ tạo resolver là addPost thuộc kiểu Mutation, cho phép chúng ta tạo bài đăng trong bảng AppSyncTutorial-Post. 

1. Chọn tab **Schema**
- Trong ngăn **Resolvers** ở bên phải, tìm trường **addPost** trên kiểu Mutation, sau đó chọn **Attach**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-2-put-item-1.png?featherlight=false&width=90pc)

2. Chọn **PostDynamoDBTable** cho mục **Data source name**
- Dán nội dung dưới đây vào mục **Configure the request mapping template**
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
Một **Type** được chỉ định trên tất cả các khoá và giá trị của thuộc tính. Ví dụ bạn đặt thuộc tính **author** thành  { "S" : "${context.arguments.author}" }. Trong đó "S" là định nghĩa kiểu giá trị của thuộc tính **author** là một String, và giá trị được lấy từ đối số **author** mà người dùng truyền vào.
{{% /notice %}}

3. Kéo xuống dưới, dán nội dung dưới đây vào mục **Configure the response mapping template**
```
$utils.toJson($context.result)
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-2-put-item-3.png?featherlight=false&width=90pc)

{{% notice note %}}
Vì shape của dữ liệu trong bảng AppSyncTutorial-Post khớp hoàn toàn với shape của Post trong GraphQL, nên mẫu ánh xạ phần hồi chỉ chuyển thẳng kết quả từ bảng qua GraphQL. 
{{% /notice %}}

4. Kéo lên trên, ấn **Save Resolver**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-2-put-item-4.png?featherlight=false&width=90pc)

5. Chọn tab **Queries** để thực hiện gọi API để thêm một bài đăng vào cơ sở dữ liệu
- Dán đoạn script dưới đây vào phần **Queries**, sau đó ấn **Execute query** (nút màu cam)
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

6. Kết quả trả về tương tự như sau:
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

7. Mở bảng điều khiển của [DynamoDB](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home?region=ap-southeast-1#/)

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-2-put-item-7.png?featherlight=false&width=90pc)

8. Chọn tab **Explore items**
- Chọn bảng **AppSyncTutorial-Post**, một bài đăng đã được ghi.

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-2-put-item-8.png?featherlight=false&width=90pc)

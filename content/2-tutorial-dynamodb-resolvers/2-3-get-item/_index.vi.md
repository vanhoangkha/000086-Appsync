---
title : "Đọc dữ liệu"
date: 2024-01-01
weight : 3
chapter : false
pre : " <b> 2.3. </b> "
---
Trong bước này chúng ta sẽ lấy ra dữ liệu trong bảng DynamoDB với key truyền vào.

1. Chọn tab **Schema**
- Trong ngăn **Resolvers** ở bên phải, tìm trường **getPost** trên kiểu **Query**, sau đó chọn **Attach**

![CreateGetResolvers](/images/2-tutorial-dynamodb-resolvers/2-3-get-item-1.png?featherlight=false&width=90pc)

2. Chọn **PostDynamoDBTable** cho mục **Data source name**
- Dán nội dung dưới đây vào mục **Configure the request mapping template**
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

3. Dán nội dung dưới đây vào mục **Configure the response mapping template**, sau đó ấn **Save Resolver**
```
$utils.toJson($context.result)
```

![CreateGetResolvers](/images/2-tutorial-dynamodb-resolvers/2-3-get-item-3.png?featherlight=false&width=90pc)

4. Chọn tab **Queries**
- Dán đoạn script dưới đây vào phần **Queries**, sau đó ấn **Execute query** (nút màu cam)
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

5. Kết quả trả về tương tự như sau:
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


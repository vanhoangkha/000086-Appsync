---
title : "Chuẩn bị "
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 2.1. </b> "
---
1. Tạo bảng trong DynamoDB bằng câu lệnh sau:
```
aws cloudformation create-stack \
    --stack-name AWSAppSyncTutorialForAmazonDynamoDB \
    --template-url https://s3.us-west-2.amazonaws.com/awsappsync/resources/dynamodb/AmazonDynamoDBCFTemplate.yaml \
    --capabilities CAPABILITY_NAMED_IAM
```
Sau khi chạy câu lệnh này một bảng **AppSyncTutorial-Post** được tạo

2. Tạo GraphQL API bằng bảng điều khiển AWS AppSync
- Mở bảng điều khiển của [AWS AppSync](https://ap-southeast-1.console.aws.amazon.com/appsync/home?region=ap-southeast-1#/apis)

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-1.png?featherlight=false&width=90pc)

- Ấn **Create API**

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-2.png?featherlight=false&width=90pc)

- Chọn **Create with wizard**, sau đó ấn **Start**

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-3.png?featherlight=false&width=90pc)

- Nhập `AWSAppSyncTutorial` cho tên của Model
- Ấn **Remove field** để loại bỏ các trường không cần thiết
- Ấn **Create**
![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-4.png?featherlight=false&width=90pc)

- Nhập tên cho API, ví dụ `AWSAppSyncTutorial`
- Ấn **Create**
![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-5.png?featherlight=false&width=90pc)

- Sau khi tạo xong, chọn tab **Schema** ở menu phía bên trái

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-6.png?featherlight=false&width=90pc)

- Thay thế nội dung bằng mã dưới đây, sau đó ấn **Save Schema**
```
schema {
    query: Query
    mutation: Mutation
}

type Query {
    getPost(id: ID): Post
}

type Mutation {
    addPost(
        id: ID!
        author: String!
        title: String!
        content: String!
        url: String!
    ): Post!
}

type Post {
    id: ID!
    author: String
    title: String
    content: String
    url: String
    ups: Int!
    downs: Int!
    version: Int!
}
```
![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-7.png?featherlight=false&width=90pc)

- Tiếp theo để tạo source cho API, ấn tab **Data Sources** 
- Ấn **Create data source**

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-8.png?featherlight=false&width=90pc)

- Nhập tên cho source, ví dụ: `PostDynamoDBTable`
- Chọn kiểu **DynamoDB table** cho kiểu dữ liệu của source
- Chọn vùng chứa dữ liệu mà bạn đã tạo
- Chọn bảng **AppSyncTutorial-Post**
- Chọn **New role** và ấn **Create**

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-9.png?featherlight=false&width=90pc)

- Kết quả sau khi tạo xong:

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-10.png?featherlight=false&width=90pc)

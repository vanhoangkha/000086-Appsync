---
title : "Tạo đối tượng phức tạp"
date :  "`r Sys.Date()`" 
weight : 3
chapter : false
pre : " <b> 3. </b> "
---
Các phần trước chúng ta đã tạo các đới tượng theo mô hình khoá - giá trị. Trong bước này chúng ta sẽ tạo các đối tượng phức tạp hơn với AWS AppSyncDynamoDB như là sets, list và maps.

1. Chọn tab **Schema**
- Thêm thuộc tính **tags** cho bài đăng
```
  type Post {
  id: ID!
  author: String
  title: String
  content: String
  url: String
  ups: Int!
  downs: Int!
  version: Int!
  tags: [String!]
}
```
- Thêm dòng dưới đây vào kiểu **Query** để thêm một truy vấn mới là **allPostsByTag**
```
  allPostsByTag(tag: String!, count: Int, nextToken: String): PaginatedPosts!
```
- Thêm hai mutation mới - addTag và removeTag - vào kiểu **Mutation**
```
type Mutation {
  addTag(id: ID!, tag: String!): Post
  removeTag(id: ID!, tag: String!): Post
  .....
}
```
- Ấn **Save Schema**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-1.png?featherlight=false&width=90pc)

2. Tiếp theo trong ngăn **Resolvers** ở bên phải, tìm trường **allPostsByTag** trên kiểu Query, sau đó chọn **Attach**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-2.png?featherlight=false&width=50pc)

3. Chọn **PostDynamoDBTable** cho mục **Data source name**
- Dán nội dung dưới đây vào mục **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "Scan",
    "filter": {
      "expression": "contains (tags, :tag)",
        "expressionValues": {
          ":tag": $util.dynamodb.toDynamoDBJson($context.arguments.tag)
        }
    }
    #if( ${context.arguments.count} )
        ,"limit": $util.toJson($context.arguments.count)
    #end
    #if( ${context.arguments.nextToken} )
        ,"nextToken": $util.toJson($context.arguments.nextToken)
    #end
}
```

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-3.png?featherlight=false&width=90pc)

- Dán nội dung dưới đây vào mục **Configure the response mapping template**
```
{
    "posts": $utils.toJson($context.result.items)
    #if( ${context.result.nextToken} )
        ,"nextToken": $util.toJson($context.result.nextToken)
    #end
}
```

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-4.png?featherlight=false&width=90pc)

- Ấn **Save Resolver**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-5.png?featherlight=false&width=90pc)

4. Chọn tab **Schema**
- Trong ngăn **Resolvers** ở bên phải, tìm trường **addTag** trên kiểu Query, sau đó chọn **Attach**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-6.png?featherlight=false&width=90pc)

5. Chọn **PostDynamoDBTable** cho mục **Data source name**
- Dán nội dung dưới đây vào mục **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "UpdateItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
    },
    "update" : {
        "expression" : "ADD tags :tags, version :plusOne",
        "expressionValues" : {
            ":tags" : { "SS": [ $util.toJson($context.arguments.tag) ] },
            ":plusOne" : { "N" : 1 }
        }
    }
}
```
- Dán nội dung dưới đây vào mục **Configure the response mapping template**
```
$utils.toJson($context.result)
```

- Ấn **Save Resolver**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-7.png?featherlight=false&width=90pc)

6. Chọn tab **Schema**
- Trong ngăn **Resolvers** ở bên phải, tìm trường **removeTag** trên kiểu Query, sau đó chọn **Attach**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-8.png?featherlight=false&width=90pc)

7. Chọn **PostDynamoDBTable** cho mục **Data source name**
- Dán nội dung dưới đây vào mục **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "UpdateItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
    },
    "update" : {
        "expression" : "DELETE tags :tags ADD version :plusOne",
        "expressionValues" : {
            ":tags" : { "SS": [ $util.toJson($context.arguments.tag) ] },
            ":plusOne" : { "N" : 1 }
        }
    }
}
```
- Dán nội dung dưới đây vào mục **Configure the response mapping template**
```
$utils.toJson($context.result)
```

- Ấn **Save Resolver**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-9.png?featherlight=false&width=90pc)

8. Chọn tab **Queries**
- Dán đoạn script dưới đây vào phần **Queries**
```
query allPostsByAuthor {
  allPostsByAuthor(
    author: "Nadia"
  ) {
    posts {
      id
      title
    }
    nextToken
  }
}
```
- Sau đó ấn **Execute query** (nút màu cam)

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-10.png?featherlight=false&width=90pc)

9. Dán đoạn script sau để thêm tag cho đối tượng vừa trả về bên trên, sau đó ấn **Execute query** (nút màu cam)
```
mutation addTag {
  addTag(id:10 tag: "dog") {
    id
    title
    tags
  }
}
```


![CreateGetPostByTag](/images/3-complex-object/3-complex-object-11.png?featherlight=false&width=90pc)

Kết quả trả về đối tượng đã được một danh sách **tags** với giá trị là "dog"
10. Đổi giá trị của tag thành **puppy** rồi ấn **Execute query** (nút màu cam)

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-12.png?featherlight=false&width=90pc)

Kết quả trả về đối tượng đã được thêm "puppy" vào danh sách **tags**

11. Dán đoạn script sau để xoá một tag, sau đó ấn **Execute query** (nút màu cam)
```
mutation removeTag {
  addTag(id:10 tag: "puppy") {
    id
    title
    tags
  }
}
```

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-13.png?featherlight=false&width=90pc)

12. Dán script sau để lấy ra toàn bộ bài đăng có tag là **dog**, sau đó ấn **Execute query** (nút màu cam)
```
query allPostsByTag {
  allPostsByTag(tag: "dog") {
    posts {
      id
      title
      tags
    }
    nextToken
  }
}
```

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-14.png?featherlight=false&width=90pc)

13. Tiếp theo chúng ta thêm **Comments** vào bài đăng. Điều này sẽ được mô hình hóa dưới dạng danh sách các đối tượng map trên Post trong DynamoDB.
14. Chọn tab **Schema**
- Thêm một kiểu **Comment** mới
```
type Comment {
    author: String!
    comment: String!
}
```
- Thêm thuộc tính **comments** cho kiểu **Post**
```
type Post {
  id: ID!
  author: String
  title: String
  content: String
  url: String
  ups: Int!
  downs: Int!
  version: Int!
  tags: [String!]
  comments: [Comment!]
}
```
- Thêm dòng sau vào kiểu **Mutation** để thêm một mutation mới **addComment**
```
  addComment(id: ID!, author: String!, comment: String!): Post
```
- Ấn **Save Schema**, sau đó ấn **Attach** ở phần **Revolvers** của mutation **addComment**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-15.png?featherlight=false&width=90pc)

15. Chọn **PostDynamoDBTable** cho mục **Data source name**
- Dán nội dung dưới đây vào mục **Configure the request mapping template**
```
{
  "version" : "2017-02-28",
  "operation" : "UpdateItem",
  "key" : {
    "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
  },
  "update" : {
    "expression" : "SET comments = list_append(if_not_exists(comments, :emptyList), :newComment) ADD version :plusOne",
    "expressionValues" : {
      ":emptyList": { "L" : [] },
      ":newComment" : { "L" : [
        { "M": {
          "author": $util.dynamodb.toDynamoDBJson($context.arguments.author),
          "comment": $util.dynamodb.toDynamoDBJson($context.arguments.comment)
          }
        }
      ] },
      ":plusOne" : $util.dynamodb.toDynamoDBJson(1)
    }
  }
}
```
- Dán nội dung dưới đây vào mục **Configure the response mapping template**
```
$utils.toJson($context.result)
```
- Sau đó ấn **Save Resolver**

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-16.png?featherlight=false&width=90pc)

16. Chọn tab **Queries**
- Thêm đoạn script dưới đây vào phần **Queries**, sau đó ấn **Execute query** (nút màu cam)

![CreateGetPostByTag](/images/3-complex-object/3-complex-object-17.png?featherlight=false&width=90pc)

---
title : "Cập nhật dữ liệu"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 2.4. </b> "
---
Trong bước này chúng ta sẽ tạo một Mutation mới cho phép cập nhật đối tượng trong bảng của DynamoDB. Chúng ta sẽ sử dụng UpdateItem của DynamoDB.

#### Cập nhật bài đăng 

1. Chọn tab **Schema**
- Thêm đoạn code dưới đây vào kiểu **Mutation** để thêm một mutation mới là **updatePost**
- Sau đó ấn **Save Schema**
```
    updatePost(
        id: ID!,
        author: String!,
        title: String!,
        content: String!,
        url: String!
    ): Post
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-1.png?featherlight=false&width=90pc)

2. Trong ngăn **Resolvers** ở bên phải, tìm trường **updatePost** trên kiểu Mutation, sau đó chọn **Attach**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-2.png?featherlight=false&width=90pc)

3. Chọn **PostDynamoDBTable** cho mục **Data source name**
- Dán nội dung dưới đây vào mục **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "UpdateItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
    },
    "update" : {
        "expression" : "SET author = :author, title = :title, content = :content, #url = :url ADD version :one",
        "expressionNames": {
            "#url" : "url"
        },
        "expressionValues": {
            ":author" : $util.dynamodb.toDynamoDBJson($context.arguments.author),
            ":title" : $util.dynamodb.toDynamoDBJson($context.arguments.title),
            ":content" : $util.dynamodb.toDynamoDBJson($context.arguments.content),
            ":url" : $util.dynamodb.toDynamoDBJson($context.arguments.url),
            ":one" : { "N": 1 }
        }
    }
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-3.png?featherlight=false&width=90pc)

{{% notice note %}}
Resolver này đang sử dụng DynamoDB UpdateItem, khác biệt đáng kể với hoạt động PutItem. Thay vì viết toàn bộ mục, bạn chỉ yêu cầu DynamoDB cập nhật một số thuộc tính nhất định. Để làm được điều này cần sử dụng DynamoDB Update Expressions. Bạn có thể thấy phần "expression" trong mục "update". Nó cho biết đặt thuộc tính author, title, content và url, sau đó tăng trường version. Expression có các placeholder có tên bắt đầu bằng dấu hai chấm, sau đó được xác định trong trường "expressionValues". Cuối cùng, DynamoDB có các từ dành riêng không thể xuất hiện trong expression. Ví dụ: url là một từ dành riêng, do đó, để cập nhật trường url, bạn có thể sử dụng  placeholder tên và xác định chúng trong trường "expressionNames".
{{% /notice %}}

4. Kéo xuống dưới, dán nội dung dưới đây vào mục **Configure the response mapping template**
```
$utils.toJson($context.result)
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-4.png?featherlight=false&width=90pc)

4. Kéo lên trên, ấn **Save Resolver**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-5.png?featherlight=false&width=90pc)

5. Chọn tab **Queries** để thực hiện gọi API để cập nhật một bài đăng
- Dán đoạn script dưới đây vào phần **Queries**, sau đó ấn **Execute query** (nút màu cam)
```
mutation updatePost {
  updatePost(
    id:"123"
    author: "A new author"
    title: "An updated author!"
    content: "Now with updated content!"
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

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-6.png?featherlight=false&width=90pc)

6. Kết quả trả về tương tự như sau:
```
{
  "data": {
    "updatePost": {
      "id": "123",
      "author": "A new author",
      "title": "An updated author!",
      "content": "Now with updated content!",
      "url": "https://aws.amazon.com/appsync/",
      "ups": 1,
      "downs": 0,
      "version": 2
    }
  }
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-7.png?featherlight=false&width=90pc)

Sau khi thực hiện sau việc cập nhật, thuộc tính **version** được tăng lên một đơn vị vì chúng ta đã yêu cầu  AWS AppSync và DynamoDB thêm một đơn vị cho thuộc tính **version** mỗi khi cập nhật.

Khi bạn sử dụng mutation updatePost như trên sẽ xảy ra 2 vấn đề sau:
- Nếu bạn chỉ muốn cập nhật một trường duy nhất, bạn phải cập nhật tất cả các trường.
- Nếu hai người đang sửa đổi đối tượng, bạn có thể bị mất thông tin.
Thực hiện các bước tiếp theo để khắc phục vấn đề trên

8. Chọn tab **Schema**
- Thay thế code của mutation **updatePost** bằng nội dung dưới đây:
- Sau đó ấn **Save Schema**
```
    updatePost(
        id: ID!,
        author: String,
        title: String,
        content: String,
        url: String,
        expectedVersion: Int!
    ): Post
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-8.png?featherlight=false&width=90pc)

9. Trong ngăn **Resolvers** ở bên phải, tìm trường **updatePost** trên kiểu Mutation, sau đó chọn **Attach**

10. Dán nội dung dưới đây vào mục **Configure the request mapping template**
- Sau đó ấn **Save Resolver**
```
{
    "version" : "2017-02-28",
    "operation" : "UpdateItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
    },

    ## Set up some space to keep track of things you're updating **
    #set( $expNames  = {} )
    #set( $expValues = {} )
    #set( $expSet = {} )
    #set( $expAdd = {} )
    #set( $expRemove = [] )

    ## Increment "version" by 1 **
    $!{expAdd.put("version", ":one")}
    $!{expValues.put(":one", { "N" : 1 })}

    ## Iterate through each argument, skipping "id" and "expectedVersion" **
    #foreach( $entry in $context.arguments.entrySet() )
        #if( $entry.key != "id" && $entry.key != "expectedVersion" )
            #if( (!$entry.value) && ("$!{entry.value}" == "") )
                ## If the argument is set to "null", then remove that attribute from the item in DynamoDB **

                #set( $discard = ${expRemove.add("#${entry.key}")} )
                $!{expNames.put("#${entry.key}", "$entry.key")}
            #else
                ## Otherwise set (or update) the attribute on the item in DynamoDB **

                $!{expSet.put("#${entry.key}", ":${entry.key}")}
                $!{expNames.put("#${entry.key}", "$entry.key")}
                $!{expValues.put(":${entry.key}", { "S" : "${entry.value}" })}
            #end
        #end
    #end

    ## Start building the update expression, starting with attributes you're going to SET **
    #set( $expression = "" )
    #if( !${expSet.isEmpty()} )
        #set( $expression = "SET" )
        #foreach( $entry in $expSet.entrySet() )
            #set( $expression = "${expression} ${entry.key} = ${entry.value}" )
            #if ( $foreach.hasNext )
                #set( $expression = "${expression}," )
            #end
        #end
    #end

    ## Continue building the update expression, adding attributes you're going to ADD **
    #if( !${expAdd.isEmpty()} )
        #set( $expression = "${expression} ADD" )
        #foreach( $entry in $expAdd.entrySet() )
            #set( $expression = "${expression} ${entry.key} ${entry.value}" )
            #if ( $foreach.hasNext )
                #set( $expression = "${expression}," )
            #end
        #end
    #end

    ## Continue building the update expression, adding attributes you're going to REMOVE **
    #if( !${expRemove.isEmpty()} )
        #set( $expression = "${expression} REMOVE" )

        #foreach( $entry in $expRemove )
            #set( $expression = "${expression} ${entry}" )
            #if ( $foreach.hasNext )
                #set( $expression = "${expression}," )
            #end
        #end
    #end

    ## Finally, write the update expression into the document, along with any expressionNames and expressionValues **
    "update" : {
        "expression" : "${expression}"
        #if( !${expNames.isEmpty()} )
            ,"expressionNames" : $utils.toJson($expNames)
        #end
        #if( !${expValues.isEmpty()} )
            ,"expressionValues" : $utils.toJson($expValues)
        #end
    },

    "condition" : {
        "expression"       : "version = :expectedVersion",
        "expressionValues" : {
            ":expectedVersion" : $util.dynamodb.toDynamoDBJson($context.arguments.expectedVersion)
        }
    }
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-10.png?featherlight=false&width=90pc)
![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-11.png?featherlight=false&width=90pc)

11. Chọn tab **Queries** để thực hiện gọi API để cập nhật một bài đăng
- Dán đoạn script dưới đây vào phần **Queries**, sau đó ấn **Execute query** (nút màu cam)
```
mutation updatePost {
  updatePost(
    id:123
    title: "An empty story"
    content: null
    expectedVersion: 2
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

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-12.png?featherlight=false&width=90pc)

12. Kết quả trả về tương tự như sau:
```
{
  "data": {
    "updatePost": {
      "id": "123",
      "author": "A new author",
      "title": "An empty story",
      "content": null,
      "url": "https://aws.amazon.com/appsync/",
      "ups": 1,
      "downs": 0,
      "version": 3
    }
  }
}
```

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-13.png?featherlight=false&width=90pc)

{{% notice note %}}
Trong yêu cầu này, chúng ta yêu cầu AWS AppSync và DynamoDB chỉ cập nhật trường **title** và **content**, giữ nguyên các trường khác trừ việc tăng trường **version**. 
{{% /notice %}}

13. Thử ấn nút **Execute query** lần nữa, kết quả trả về tương tự như sau:

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-14.png?featherlight=false&width=90pc)

Yêu cầu không thành công vì biểu thức điều kiện thực hiện sai:
- Lần đầu tiên chạy, giá trị của trường **version** của bài đăng trong DynamoDB là 2, khớp với **expectedVersion**. Yêu cầu thành công, có nghĩa là trường phiên bản đã được tăng trong DynamoDB lên 3.
- Lần thứ hai chạy, giá trị của trường **version** của bài đăng trong DynamoDB là 3, không khớp với **expectedVersion**.

#### Cập nhật số lượng bình chọn
Chúng ta sẽ tạo hai mutation mới để cập nhật trường **ups** và **downs** để ghi lại lượt ủng hộ hay phản đối cho bài đăng

1. Chọn tab **Schema**
- Thêm đoạn code dưới đây vào kiểu **Mutation** để thêm hai mutation mới là **upvotePost** và **downvotePost**
```
    upvotePost(id: ID!): Post
    downvotePost(id: ID!): Post
```
- Sau đó ấn **Save Schema**
- Tiếp theo trong ngăn **Resolvers** ở bên phải, tìm trường **upvotePost** trên kiểu Mutation, sau đó chọn **Attach**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-15.png?featherlight=false&width=90pc)

2. Chọn **PostDynamoDBTable** cho mục **Data source name**
- Dán nội dung dưới đây vào mục **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "UpdateItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
    },
    "update" : {
        "expression" : "ADD ups :plusOne, version :plusOne",
        "expressionValues" : {
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

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-16.png?featherlight=false&width=90pc)

3. Chọn tab **Schema**
- Trong ngăn **Resolvers** ở bên phải, tìm trường **downvotePost** trên kiểu Mutation, sau đó chọn **Attach**

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-17.png?featherlight=false&width=90pc)

4. Chọn **PostDynamoDBTable** cho mục **Data source name**
- Dán nội dung dưới đây vào mục **Configure the request mapping template**
```
{
    "version" : "2017-02-28",
    "operation" : "UpdateItem",
    "key" : {
        "id" : $util.dynamodb.toDynamoDBJson($context.arguments.id)
    },
    "update" : {
        "expression" : "ADD downs :plusOne, version :plusOne",
        "expressionValues" : {
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

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-18.png?featherlight=false&width=90pc)

5. Chọn tab **Queries**
- Dán đoạn script dưới đây vào phần **Queries**, sau đó ấn **Execute query** (nút màu cam)
```
mutation votePost {
  upvotePost(id:123) {
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

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-20.png?featherlight=false&width=90pc)

6. Kết quả trả về tương tự như sau:
```
{
  "data": {
    "upvotePost": {
      "id": "123",
      "author": "A new author",
      "title": "An empty story",
      "content": null,
      "url": "https://aws.amazon.com/appsync/",
      "ups": 2,
      "downs": 0,
      "version": 4
    }
  }
}
```

7. Đổi lệnh truy vấn thành **downvotePost**
```
mutation votePost {
  downvotePost(id:123) {
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
- Ấn nút **Execute query** (nút màu cam)

![CreateAddPostResolver](/images/2-tutorial-dynamodb-resolvers/2-4-update-item-21.png?featherlight=false&width=90pc)

8. Ấn nút **Execute query** (nút màu cam) lần nữa bạn sẽ thấy trường "downs" tăng lên

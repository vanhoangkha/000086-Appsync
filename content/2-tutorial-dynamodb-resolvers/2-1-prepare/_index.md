---
title : "Preparation"
date: 2024-01-01
weight : 1
chapter : false
pre : " <b> 2.1. </b> "
---
1. Create a table in DynamoDB with the following command:
```
aws cloudformation create-stack \
    --stack-name AWSAppSyncTutorialForAmazonDynamoDB \
    --template-url https://s3.us-west-2.amazonaws.com/awsappsync/resources/dynamodb/AmazonDynamoDBCFTemplate.yaml \
    --capabilities CAPABILITY_NAMED_IAM
```
After running this command a **AppSyncTutorial-Post** table is created

2. Create GraphQL API with AppSync console
- Open [AWS AppSync console]()

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-1.png?featherlight=false&width=90pc)

- Click **Create API**

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-2.png?featherlight=false&width=90pc)

- Choose **Create with wizard**, then click **Start**

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-3.png?featherlight=false&width=90pc)

- Enter Model name - `AWSAppSyncTutorial`
- Click **Remove field** to remove unnecessary fields
- Click **Create**
![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-4.png?featherlight=false&width=90pc)

- Enter API name, such as `AWSAppSyncTutorial`
- Click **Create**
![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-5.png?featherlight=false&width=90pc)

- Once created, select the **Schema** tab in the menu on the left

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-6.png?featherlight=false&width=90pc)

- Replace content with below code, then click **Save Schema**
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

- Next, to create source for API, choose **Data Sources** tab
- Click **Create data source**

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-8.png?featherlight=false&width=90pc)

- Enter source name, such as: `PostDynamoDBTable`
- Select **DynamoDB table** for data source type
- Select the region that created DynamoDB table
- Select **AppSyncTutorial-Post** table
- Select **New role** and click **Create**

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-9.png?featherlight=false&width=90pc)

- Result after creation:

![CreateAPI](/images/2-tutorial-dynamodb-resolvers/2-1-prepare-10.png?featherlight=false&width=90pc)

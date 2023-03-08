---
title : "Cleanup"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 4. </b> "
---
1. Delete DynamoDB table
- Open [AWS CloudFormation console](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home?region=ap-southeast-1#/)
- Choose **AWSAppSyncTutorialForAmazonDynamoDB** stack
- Click **Delete**
- Click **Delete stack**
2. Delete API
- Open [AWS AppSync console](https://ap-southeast-1.console.aws.amazon.com/appsync/home?region=ap-southeast-1#/apis)
- Chọn api **AWSAppSyncTutorial**
- Click **Delete**
- Enter **AWSAppSyncTutorial**
- Click **Delete**
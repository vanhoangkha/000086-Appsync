# Serverless - Introduction to AWS AppSync

> GraphQL APIs with AWS AppSync.

A hands-on workshop from the [AWS First Cloud Journey](https://github.com/vanhoangkha/aws-first-cloud-journey) series, available in English and Vietnamese.

**Read the workshop online:** https://vanhoangkha.github.io/000086-Appsync/

## Overview

In the previous post we learned about CloudWatch and X-Ray, next we will build messaging for web application using AWS Appsync that interacts with GraphQL. To familiarize with AWS Appsync, in this article we will learn how AWS Appsync works by creating, updating, reading, and deleting data in DynamoDB tables.

## Workshop contents

1. Introduction
2. Tutorial: DynamoDB Resolvers
   - Preparation
   - Writing data
   - Reading data
   - Updating data
   - Deleting data
   - Scanning table
   - Querying table
3. Create complex object
4. Cleanup

## Prerequisites

- An AWS account with permissions to create the resources used in the labs
- Basic familiarity with the AWS Management Console
- [Hugo](https://gohugo.io/installation/) (extended edition) if you want to run the site locally

## Run locally

```bash
git clone https://github.com/vanhoangkha/000086-Appsync.git
cd 000086-Appsync
hugo server
```

Then open http://localhost:1313 in your browser.

## Repository structure

| Path | Purpose |
| --- | --- |
| `content/` | Workshop pages (`*.md` English, `*.vi.md` Vietnamese) |
| `static/` | Images and other static assets |
| `layouts/` | Hugo layout overrides |
| `themes/` | Hugo theme |
| `config.toml` | Site configuration |

## Cost and cleanup

The labs create AWS resources that may incur charges. Complete the **Clean up** chapter when you finish to remove them.

## Contributing

Contributions are welcome. See [CONTRIBUTING.md](CONTRIBUTING.md).

To report a security issue, see [SECURITY.md](SECURITY.md).

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE).

---
layout: post
title: Getting Started with AWS Dynamo DB and PHP
description: A Quick Getting Started Guide to AWS Dynamo DB with PHP
---

### Dynamo DB Local tool
For testing Dynamo DB, download local testing tool from:

*<a href="http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html" target="_blank">http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html</a>*

Note that this is not a replacement for DynamoDB and should not be used in a live application.

For running the tool execute following command (note that you need to have JRE installed), after changing the path *'/root/DynamoDBServer/'* accordingly

(java -Djava.library.path=/root/DynamoDBServer/DynamoDBLocal_lib -jar /root/DynamoDBServer/DynamoDBLocal.jar -sharedDb &)

Usually local tool uses the port 8000. Also you can access the web application for managing the DB through following url

![AWS Local tool Shell](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/images/shell-main.png "AWS Local tool Shell")


### Dynamo DB client library

First step is to download the Dynamo DB PHP client library is a part of AWS sdk. It can be downloaded through composer or
from [https://github.com/aws/aws-sdk-php](https://github.com/aws/aws-sdk-php)


### Connecting to Dynamo DB local from PHP




** Not completed yet .. **
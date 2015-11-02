---
layout: post
title: Getting Started with AWS Dynamo DB and PHP
description: A Quick Getting Started Guide to AWS Dynamo DB with Vocanic Dynamo DB Connector
---

Dynamo DB is the well known NoSQL DB solution from Amazon. At Vocanic we've been using a bunch of services provided by AWS.
While looking into way to improve the performance of our data storage mechanisms we though of doing some experiments with
Dynamo DB. Vocanic Dynamo DB is a result of that. It is a wrapper class on top of [DynamoDb Marshaler](http://docs.aws.amazon.com/aws-sdk-php/v2/api/class-Aws.DynamoDb.Marshaler.html) 
and add few function to help migrate our existing applications to Dynamo DB without much of a pain.

This post will provide some essential tips on quickly setting up the test environment and start experimenting with Dynamo DB.

### Dynamo DB Local tool
For testing Dynamo DB locally, you can download local testing tool from:

*<a href="http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html" target="_blank">http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html</a>*

This tool has test interfaces for almost all the functionalities provided by the live DynamoDB. 

For running the tool execute following command (note that you need to have JRE installed), after changing the path *'/root/DynamoDBServer/'* accordingly

(java -Djava.library.path=/root/DynamoDBServer/DynamoDBLocal_lib -jar /root/DynamoDBServer/DynamoDBLocal.jar -sharedDb &)

Usually local tool uses the port 8000. Also you can access the web application for managing the 
DB through *http://localhost:8000/shell/*

![AWS Local tool Shell](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/images/shell-main.png "AWS Local tool Shell")


### Clone Vocanic Dynamo DB Connector - Sample application

Checkout the following github repo

[https://github.com/vocanic/dynamodbconnect-sample](https://github.com/vocanic/dynamodbconnect-sample)

{% highlight bash %}
git clone git://github.com/vocanic/dynamodbconnect-sample.git
{% endhighlight %}

Use composer to download dependencies (If you are new to composer please check [](https://getcomposer.org/doc/00-intro.md))

Change current directory to project root and use following command to download dependencies via composer

{% highlight bash %}
curl -sS https://getcomposer.org/installer | php

composer install
{% endhighlight %}

### Creating your first Dynamo DB table

Change your current directory into <project root>/src

{% highlight bash %}
cd src
{% endhighlight %}

Run the example file for creating a Users table

{% highlight bash %}
php 1-createUsersTable.php
{% endhighlight %}


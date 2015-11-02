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

## Dynamo DB Local tool
For testing Dynamo DB locally, you can download local testing tool from:

*<a href="http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html" target="_blank">http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.DynamoDBLocal.html</a>*

This tool has test interfaces for almost all the functionalities provided by the live DynamoDB. 

For running the tool execute following command (note that you need to have JRE installed), after changing the path *'/root/DynamoDBServer/'* accordingly

(java -Djava.library.path=/root/DynamoDBServer/DynamoDBLocal_lib -jar /root/DynamoDBServer/DynamoDBLocal.jar -sharedDb &)

Usually local tool uses the port 8000. Also you can access the web application for managing the 
DB through *http://localhost:8000/shell/*

![AWS Local tool Shell](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/images/shell-main.png "AWS Local tool Shell")


## Clone Vocanic Dynamo DB Connector - Sample application

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

## Creating your first Dynamo DB table

### Creating Connection

In order to connect to dynamo db you need an access key and secret with permissions for dynamo DB. You can create these keys via
[aws AIM](http://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html). But for accessing a local test DB you can
you some random strings.

{% highlight php %}
if(!defined('AWS_DYNAMO_DB_ACCESS_KEY')){define('AWS_DYNAMO_DB_ACCESS_KEY','__A_VALID_AWS_KEY_REQUIRED__');}
if(!defined('AWS_DYNAMO_DB_ACCESS_SECRET')){define('AWS_DYNAMO_DB_ACCESS_SECRET','__A_VALID_AWS_SECRET_REQUIRED__');}
{% endhighlight %}

Also when using aws client you need to bind it to a particular client version. This can be used to prevent your application from
breaking due to AWS upgrades.

{% highlight php %}
if(!defined('AWS_CLIENT_VERSION')){define('AWS_CLIENT_VERSION',"2012-08-10");}
{% endhighlight %}

* For our sample application these constants are defined in include.inc.php *

Now with these values in hand you can create a connection to DynamoDB local tool

{% highlight php %}
if(defined('USE_LOCAL_DB') && USE_LOCAL_DB === "0"){
    $isTestDB = false;
}
\Vocanic\Common\DynamoDBObject::initializeDynamoDBClient(AWS_DYNAMO_DB_ACCESS_KEY, AWS_DYNAMO_DB_ACCESS_SECRET, NULL, true);

{% endhighlight %}

The fourth parameter passed will force VocanicDynamodbConnector to use local DB

### DynamoDB Hash, range keys and Secondary indices

All DynamoDB table must have a primary key. The primary key can consists of a hash only or a hash/range pair. Usually the hash 
keys should distribute data in the table evenly among each key and range can be used to identify individual records. For an
example you if you are storing users data from several clients in a table the hash of Users table can be client id while,
userId becomes the range key.

Also Dynamo db can have global and local secondary indices (currently we support only global secondary indices). For more
information about refer:

[](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html)
[](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/LSI.html)

### DynamoDB Model Classes with Vocanic DB Connector

Here is a sample ORM model class created for Users table. With client as HASH key, userId as RANGE key and email as
a secondary index.

{% highlight php %}

class User extends \Vocanic\Common\DynamoDBObject{

    public function getTableName(){
        return "Users";
    }

    public function getDataFields(){
        return array(
            "client",
            "userId",
            "email",
            "password",
            "name",
            "location",
            "time"
        );
    }

    public function getTableMeta(){
        return array(
            'TableName'=>$this->getTableName(),
            'AttributeDefinitions'=>array(
                array(
                    'AttributeName' => 'client',
                    'AttributeType' => 'S'
                ),
                array(
                    'AttributeName' => 'userId',
                    'AttributeType' => 'S'
                ),
                array(
                    'AttributeName' => 'email',
                    'AttributeType' => 'S'
                )
            ),
            'KeySchema' => array(
                array(
                    'AttributeName' => 'client',
                    'KeyType'       => 'HASH'
                ),
                array(
                    'AttributeName' => 'userId',
                    'KeyType'       => 'RANGE'
                )
            ),

            'GlobalSecondaryIndexes' => array(
                array(
                    'IndexName' => 'emailIndex',
                    'KeySchema' => array(
                        array('AttributeName' => 'email',    'KeyType' => 'HASH')
                    ),
                    'Projection' => array(
                        'ProjectionType' => 'KEYS_ONLY',
                    ),
                    'ProvisionedThroughput' => array(
                        'ReadCapacityUnits'  => 1,
                        'WriteCapacityUnits' => 1
                    )
                )
            ),


            'ProvisionedThroughput' => array(
                'ReadCapacityUnits'  => 1,
                'WriteCapacityUnits' => 1
            )
        );
    }
}


{% endhighlight %}

Each model class should:

1. Extend \Vocanic\Common\DynamoDBObject
2. Define a set of supported fields in "getDataFields"
3. Define the Dynamo DB table name in "getTableName"
4. Define meta data in "getTableMeta" (this is optional if you are extending VocanicDynamoDBObject)

When defining the meta data section keep in mind that you need to add secondary indices also into "AttributeDefinitions".
Also you need to have a clear understanding of DynamoDB data types which is used to specify "AttributeType" parameter for
each Attribute Definition.

[More about Dynamo DB Data Types](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DataModel.html#DataModel.DataTypes)

Also you can set "ProvisionedThroughput" as required [](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ProvisionedThroughputIntro.html)

* In our sample application, these model classes are defined in src/models.inc.php *


### Creating Users table

Run the example file for creating a Users table

{% highlight bash %}
php src/1-createUsersTable.php
{% endhighlight %}

DynamoDBObject::create method will use getMetaData method of the sub class to create Dynamo DB table.
 

## Inserting a User object

In order to insert a test user into Users table run the following script

{% highlight bash %}
cd src/2-createUser.php
{% endhighlight %}

This will create a new entry in Users table. You can check this using AWS local DB shell which is accessible through
http://localhost:8000/shell/

Inside the script you will notice that we are creating a user model object and calling the Save() method

{% highlight php %}
$user = new User();
$user->client = 'Vocanic';
$user->email = "user@vocanic.com";
$user->password = "Password";
$user->name = "User ".rand();
$user->location = "SL";
$user->time = 100;
$user->Save();
{% endhighlight %}

You might notice that we are not assigning a value to userId. If there was no value assigned to the userId,
VocanicDynamoDB connector assign a random unique value to the range key field. *(We have done this to resolve
some issue with DynamoDB not supporting auto incrementing fields)*







---
layout: post
title: "Getting Started with Cloudformation"
date: 2017-11-20
categories:
  - aws
  - cloudformation
description: "The minimum viable example that I've been looking for."
header-img: "img/post-bg-cogs.jpeg"
---

Few weeks ago, I tried to learn how to use AWS Cloudformation. For those who are not familiar,
Cloudformation is an AWS service that provides an easy way to create and manage AWS Services.
Cloudformation uses templates in a format of JSON or YAML which can be uploaded from AWS console,
or through AWS CLI.

## Requirements
This tutorial requires installing [aws-cli](http://docs.aws.amazon.com/cli/latest/userguide/installing.html).
So, if you don't have it yet, I recommend installing it since it will be easier to run a few
commands than clicking around AWS console.

## The MVE
Let's start with the MVE (minimum viable example) by creating an AWS S3 bucket. The goal here is to
create a stack by writing the least amount of code so that we can understand each component easier.

The only requirement that a Cloudformation template has is a `Resource`. This dictates to AWS which
resource to build. The template also accepts a AwsTemplateFormatVersion (which has not changed since
they made it), Description, Metadata, Parameters, Mappings, Conditions, Transform, Resources,
and Outputs. You can find a detailed explanation on what these optional values does in
[here](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-anatomy.html).

{% highlight yaml %}
# template.yaml
AWSTemplateFormatVersion: "2010-09-09" # this is not required since they only have 1 version.
Description: Optional description of your stack.
Resources:
  yourUniqueBucketName:
    Type: AWS::S3::Bucket
{% endhighlight %}

Each `Resource` requires two components, it should always have a key or identifier. In this example,
I am using S3 bucket. An S3 bucket's identifier is globally namespaced, so the key `yourUniqueBucketName`
should be used to your own unique bucket name. The second requirement for a resource is the `Type`.
This tells cloudformation what kind of resource you are trying to create. The list of resource types can be found
[here](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)

I find that when I create a stack, I create and delete it a few times so I find myself creating a
bash script to save my fingers from typing too much in case I want to edit a few details.

{% highlight bash %}
# create-cfn.sh
# run me with `bash create-fn.sh`
#!/bin/bash

aws cloudformation create-stack \
  --stack-name MyFirstCFNStack \
  --template-body file://${PWD}/template.yml

# delete-cfn.sh
# run me with `bash delete-cfn.sh`
#!/bin/bash

aws cloudformation delete-stack \
  --stack-name MyFirstCFNStack \
{% endhighlight %}

A few things to note here, `--stack-name` and `--template-body` are required. If you have your
Cloudformation template in an s3 file for example, you can pass in `--template-url` instead, but
you cannot use both. When this has run, it should create an S3 bucket. It is pretty simple but the
power of Cloudfomation can be seen when creating a stack that includes more than one AWS service.

## Example 1.0

To see the real power of Cloudformation, we will have to create more than one resource. Let's now
try to create a Lambda that uses a specific Role to execute it.

{% highlight yaml %}
AWSTemplateFormatVersion: "2010-09-09"
Description: Optional description of your stack.
Resources:
  MyAwesomeLambda:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: nodejs4.3
      Handler: index.handler
      Role: !GetAtt LambdaExecutionRole.Arn
      Code:
        ZipFile: !Sub |
          var response = require('cfn-response');
          exports.handler = function(event, context) {
             var responseData = {Value: event.ResourceProperties.List};
             responseData.Value.push(event.ResourceProperties.AppendedItem);
             response.send(event, context, response.SUCCESS, responseData);
          };
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
        - Effect: Allow
          Principal:
            Service:
            - lambda.amazonaws.com
          Action:
          - sts:AssumeRole
{% endhighlight %}

As you can see, we have a lot more going on in this example although this is still the minimum
template required to create a Lambda Function.
A [AWS::Lambda::Function](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-function.html)
requires 4 `Properties`: `Runtime`, `Handler`, `Role` and `Code`. Sadly, the documentation is not
ordered by the required properties. Pro tip: find `Required: Yes` to quickly find required properties.

Since IAM Role is a required property of a Lambda, it is created separately. However, if we are
creating multiple Lambda resource, we only need to create the role once unless the Lambda requires
a different policy.

---

AWS Cloudformation is a really powerful tool. I'm only scratching the surface in here. AWS
documentation tends to be a bit heavy for me but they are very detailed. It does makes sense because
each AWS resource is packed with so much features.

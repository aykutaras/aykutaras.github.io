# Migrating To Serverless Framework

When developing serverless applications, Serverless Framework can ease the pain a lot. If you are starting a new project, you should definitely consider using Serverless Framework.

But what if you have an existing application which already benefits from AWS CloudFormation stacks for orchestration. As of the writing this post, it is not possible to easily migrate existing CloudFormation templates to Serverless Framework compatible ones.

So how do you migrate your existing CloudFormation stack? Don't worry I have some workarounds for you. Before I start, I have to say that this process is really depends on the existing CloudFormation stack and can change from stack to stack. Here is what I have learned so far:

## 1 - Deleting Existing Stack
This one is pretty easy. Just delete existing CloudFormation stack and deploy new Serverless Framework stack. You should consider following things before deciding to use this approach.

First of all when you removed your old CloudFormation stack, everything inside of it will be removed. That means if your stack manages any persisting resources (dynamodb, rds, s3, etc.), these with their corresponding datas will be removed. So for protecting your data you need to get manual backups for each persistence resources and after new CloudFormation stack you need to restore all of them. Secondly, you need to decide a time frame for at least 5 minutes/hours (depending on your application deployment and backup restoration) of downtime. Removing your old stack removes everything and after that you will be creating a totally new CloudFormation stack and restoring all the backups you had which will take time. Lastly, even though you get an export, you will lose all your CloudWatch logs and your backups could not be restored anyhow.

I think there are two possible use cases of this approach. You already using different CloudFormation stacks for Lambda functions and all other AWS resources or your environment is not really getting much traffic or even in test mode. Otherwise please skip this approach completely :)

**Pros**
* Easiest approach
* Clean new stack

**Cons**
* Possible downtime for minutes/hours
* All existing resources with their datas will be removed
* If any [exported output values](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-stack-exports.html) are used by other CloudFormation stacks you need to remove these stacks too


## 2 - Updating Existing Stack

Updating existing CloudFormation stack to serverless framework is a bit tricky but with this way you can have all your resources in your previous stack and build towards it. This way will be changed depending on your CloudFormation stack and I strongly suggest create some sandbox stacks to test your changes.

First you need to create a new CloudFormation template file with following updates;
* An S3 bucket resource named `ServerlessDeploymentBucket` 
* An output value referencing `ServerlessDeploymentBucket` as a value with named `ServerlessDeploymentBucketName`
* Remove existing lambda function completely from stack

If you check serverless CloudFormation templates, you will see two seperate templates. 
```json
{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Description": "The AWS CloudFormation template for this Serverless application",
  "Resources": {
    "ServerlessDeploymentBucket": {
      "Type": "AWS::S3::Bucket"
    }
  },
  "Outputs": {
    "ServerlessDeploymentBucketName": {
      "Value": {
        "Ref": "ServerlessDeploymentBucket"
      }
    }
  }
}

```

Above one is `cloudformation-template-create-stack.json` which is just creating an S3 bucket and referencing name of the bucket. Our aim was to simulate this behavior inside our stack template.

Next, you need to update your serverless.yml file with;
* Change your lambda name same as your old lambda name. You can achieve this adding a `name: NAME` inside your function
* Add a new `AWS::Logs::LogGroup` resource with resource name is `${YOUR_LAMBDA_NAME}LogGroup` and a dummy LogGroupName as a property. This is important because Serverless Framework will try to create a log group same as existing log group of your lambda function. CloudFormation will throw an exception because this group already exists in the system. This is a really bad workaround but I could not find a smarter way for this. We are going to trick CloudFormation that we are creating a new log group but lambda function will continue to send logs to previous log group.
* Add as many LogGroups as your lambda functions inside your serverless and old cloudformation stacks needed

These are must-have operations that should be done. Rest differs between your old stack and new serverless stack. I would strongly suggest that creating sandbox environments and testing your new migration scripts over and over again until you are fully confident with them.

**Pros**
* Maintains existing state
* Preserve all existing resources
* Cloudwatch logs and Exported Output Values are preserved too
* Little or no downtime at all

**Cons**
* Hard to achieve
* Things may get complicated if old stack is too complex

# Migrating To Serverless Framework

Using Serverless Framework for deploying serverless application makes much of the headaches gone. It is easier to use Serverless from scratch but what if we have an existing application which already uses AWS CloudFormation for orchestration. How do we migrate our existing CloudFormation stack?

As of the writing this post, there seems not possible to easily migrate existing cloudformation templates to serverless. But there are ways.
First one is updating existing cloudformation script to serverless framework's needs. With this way you can have all your previous stack and build towards it. This path may become the hardest one dependening on your CloudFormation template and serverless.yml file.

Only required changes to convert an existing CloudFormation template to serverless compatible template are;
* An S3 bucket resource named `ServerlessDeploymentBucket` 
* An output referencing `ServerlessDeploymentBucket` as a value with named `ServerlessDeploymentBucketName`

These two operations are must-have attributes that should be inside your CloudFormation stack. Rest depends differences between your stack and serverless stack and it may differ quite a lot. If you have exported output variables and so on, things will be really complicated.

Second option is just create a new CloudFormation stack and remove the old one completely. At first this one seems really easy but when you think about backing up all the data from databases and possible downtime, it may not be a good option for your production environment.

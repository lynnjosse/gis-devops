---
title: "Scaling horizontally AutoScaling Group"
---

The ability to scale horizontally is very important in building Cloud Native applications.  In this studio, you will be extending your [Airwaze App](https://gitlab.com/LaunchCodeTraining/airwaze-studio) to scale horizontally as traffic on the server increases.

## Setup

For this studio, you should use the VPC that you built in the [Load Balanced Cloud Studio](../../studio/AWS2). 

We will be using the AWS CLI tool for some parts of the studio.  The AWS CLI tool allows you to create and change infrastructure on the cloud via the command line.  

To install the AWS CLI tool run the following commands:
```
$ brew install awscli
$ echo 'complete -C aws_completer aws' >> ~/.bashrc
$ aws --version
```

Next configure the AWS CLI tool with your credentials.  These credentials can be found in IAM > Users > You. Click "Create Access Key".
![Screenshot of IAM credentails](../../materials/week05/day3/iam_get_credentials.png)

<aside class="aside-note">
  It is very important that you keep the `AWS Secret Access Key` private.  Access to that key allows anyone to programmatically spin up infrastructure on the AWS account. 
</aside>

Next configure your the AWS CLI tool.  Use the "Default region name" or `us-east-2`:
```
$ aws configure
AWS Access Key ID [None]: AK-------------------
AWS Secret Access Key [None]: r4------------------
Default region name [None]: us-east-2
Default output format [None]: 
```

For example, you should now be able to list all of the buckets in S3:
```
aws s3 ls
```

### Configure Buckets

Since we are going to be horizontally scaling, we need machines to spin up automatically without any human interaction.  We are going to upload the executable jar file to S3 so that machines can pull down the file when automatically spin up.

First create a new bucket in S3.  Remember **EVERY** bucket in S3 in the whole wide world has to be unique.  Use the pattern below to get a unique name.

```
$ aws s3 mb s3://launchcode-gisdevops-c1-yourname/
```

Run `aws s3 ls` to make sure that the bucket was created properly.

Go ahead and build a new executable jar file using the Gradle `bootRepackage` command.  When it is finished building rename the file to `app.jar` and upload the jar to S3 using the following command:

```
$ aws s3 cp build/libs/app.jar s3://launchcode-gisdevops-c1-yourname/ 
$ aws s3 ls s3://launchcode-gisdevops-c1-yourname/ # check to make sure it uploaded
```

When we run our initialization script later, the script will pull down the `app.jar` file with this command:
```
$ aws s3 sync s3://launchcode-gisdevops-c1-yourname/ /opt/airwaze
```


### Configure your VPC



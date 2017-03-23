---
layout: post
title: AWS IAM roles apply to EC2 instance in the console
---

So while going over some prep material today for the AWS Sysops Associate Exam, I noticed that AWS have finally updated the console to allow
you to attach and remove IAM roles from running EC2 instances.    According to the [A Cloud Guru](http://acloud.guru) training videos this 
was previously available, but could only be done with the CLI tools.   This is quite a useful feature, as previously the only way to attach an IAM
role to an EC2 instance in the console was at creation.

The steps to attach are pretty straight forward.   First, select the EC2 instance, go to Actions -> Instance Settings -> Attach/Replace IAM role
![Menu](/img/aws-iam/aws-iam-console-1.png)

Select the relevant IAM role from the pull down menu:
![Pull Down](/img/aws-iam/aws-iam-console-2.png)

If you're removing the assigned IAM profile (ie selecting No Role) you will get a warning:
![Warning!](/img/aws-iam/aws-iam-console-3.png)

You will get confirmation the new role has been applied:
![Confirmation](/img/aws-iam/aws-iam-console-4.png)

And then selecting the instance in the EC2 Console you can see the applied IAM role in the EC2 Settings
![EC2 Console](/img/aws-iam/aws-iam-console-5.png)


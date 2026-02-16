---
title: 'Solutions Architect Associate - Part 1'
date: '2026-02-15'
tags:
    - aws
    - journal
categories:
    - Certifications
draft: true
# weight: 1
# aliases: ["/first"]
# hidemeta: false
# comments: false
description: "Part two in my ongoing backup journey"
# disableHLJS: true # to disable highlightjs
# disableShare: false
# disableHLJS: false
# hideSummary: true

# cover:
#     image: "<image path/url>" # image path/url
#     alt: "<alt text>" # alt text
#     caption: "<text>" # display caption under cover
#     relative: false # when using page bundles set this to true
#     hidden: true # only hide on current single page
---

I recently became a Certified AWS Cloud Practitioner! The exam was relatively simple, it primarily involved understanding what all of AWS's services do at a very high level. 
However, it doesn't really prepare you to use the services, so I am continuing down the Solutions Architect Path with the Solutions Architect Associate certification!

I found  a highly rated course on Udemy called "Ultimate AWS Certified Solutions Architect Associate 2026" produced by Stéphane Maarek. In an effort to show my competency, retain the information myself, and perhaps even teach you a thing or two, I'll be producing more journal-style blog posts showcasing what I have learned. 

> Side note: anyone in the U.S. Air Force gets access to a bunch of Udemy courses for free through [Digital University](https://digitalu.af.mil/)

# IAM

In our modern world an "account" is typically tied to an "identity". So your Facebook or Google account are tied to your Facebook and Google identies respectively. No other account can access your identity. 
In the world of cloud computing, this coupling falls apart, for an AWS "account" is not an identity, but moreso a bin that can house various resources. 
This bin can then be accessed by various user identities, which themselves are resources housed within the AWS Account bin.

The service which manages these user identities is **IAM**, the first section of Stéphane's course.

**IAM** or **I**dentity and **A**ccess **M**anagement, is the AWS service that manages Users, Groups, permissions, and all the access policies used to access, add, remove, or modify the resorces in your AWS Account.

An IAM User is an idividual identity used to access account resorces. This user can be a human user, or a progomatic user, such as one used in a script or program. 
In my homelab I currently have a progromatic user that backs up my photos to an S3 bucket. The S3 bucket is a resorce, as are an EC2 virtual machine, a Lambda function, or Route 53 DNS entry (we'll cover all these later on, stay tuned).

An IAM user can be part of any number of groups, includng zero. A group can only contain users, not other groups.

Both Groups and Users can have policies assigned to them. A policy is an Allow/Deny statement regarding a users level of access to a resource. Policies are represented as JSON objects and look like this:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AdminAllowAll",
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}
```
The `Version` field will always be `2012-10-17`. I could'nt find references to any other version except `2008-10-17`, so presumably those represent dates, but I am not sure.

The `Statement` section is then a list of objects, each of which Allow or Deny certain access to certain resources. This example is very simple, you could get much more complecated.

Each object includes a `sid` (presumably this states for Security Identifier) which is an optional name for this object, it can be any name you want. Here I have `AdminAlowAll` as an easily understandable name. 
Under the `sid` is the `Effect`, which can be either Allow or Deny. This obviously controls weather the actions listed will be allowed or denied. The `Action` section is a list of API actions that are being referenced. 
You can see here that I use a star (or asterisk) to represent *all* actions. Everything from `access-analyzer:ListAnalyzers` to `xray:UntagResource` and all the others in between (`s3:ListAllBuckets`, `ec2:CopySnapshot` and `iotevents:DescribeAlarmModel` to name but a few).
Lastly is the `Resource` section which specifies which resource these actions should apply to. Again, I have all resources selected here, by using the star.
If you wanted to specifiy a specific resource, you would include its ARN or **A**mazon **R**esource **N**ame. Some examples of ARNs are:
- An S3 bucket: `arn:aws:s3:::backup-photos`
- A IAM Policy: `arn:aws:iam::102381579309:policy/test-policy-name`
- A VPC Subnet: `arn:aws:ec2:us-east-2:122312579368:subnet/subnet-1f7cf9f1d81704b00`
- An SES Identity `arn:aws:ses:us-east-2:881271468296:identity/mckay.one`
Virtually everything you create on AWS will have an ARN associated with it, and each service has many, many granular actions so these policy definitions can grow to be long and convveluted.
Best practice of course would be to break them up into manageable and understandable chunks, but sometimes that is not an option.

Moving away from policies, 


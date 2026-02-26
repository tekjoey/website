---
title: 'AWS Solutions Architect Associate - Pt2 (IAM)'
date: '2026-02-18'
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
description: "Part 2 of my journey to recieving my AWS Certified Solutions Architect Associate certification. Here, I conclude talking about IAM."
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
This is the second post on AWS IAM, and I'll be talking about Roles, IAM Security Tools, and ending with some best practices.

## Roles

We saw in the last post that you can assign policies to user accounts to allow/deny certain actions within your AWS account. But what if you wanted a third party to perform some kind of action on your account? What about an automated system? 
Is there any way to limit their access, or do they just get full *carte blanche* access to your entire account?

Well, of course you can limit their access! The feature is *IAM Roles*. These are essentially the same kind of security policies discussed in the last post, but applied to different identities or agents.

From Amazon's own documentation:
> A role is an IAM identity that you can create in your account that has specific permissions. An IAM role has some similarities to an IAM user. Roles and users are both AWS identities with permissions policies that determine what the identity can and cannot do in AWS. However, instead of being uniquely associated with one person, a role can be assumed by anyone who needs it. A role does not have standard long-term credentials such as a password or access keys associated with it. Instead, when you assume a role, it provides you with temporary security credentials for your role session.

This is huge for security in limiting the potential blast-radius. Roles allow you to give temporary permissions to user accounts, permissions that expire after a set amount of time. So the third-party contractor's user dont just have free-reign to do as he pleases, he has to assume the role first, which is an action logged and auditable by CloudWatch. Your Lamda function's user can assume a role to delete VMs, but that user account normally has no permissions otherwise. Even normal user accounts can use roles to limit their admin privileges to only when they need them, limiting any costly mistakes.

## IAM Credential Report
IAM Credential Report is an account-level security tool that can export the current status of all user accounts and their various credential into a CSV file. You can then edit and manipulate this file as you wish, to determine, for example, which users have MFA enabled, or which users have not logged in in the past 90 days. This report also has information regarding access keys and when they were last rotated, super important for maintaining a strong security posture.

## IAM Access Advisor
IAM Access Advisor (renamed recently to simply "Last Access") is a user-level tool that shows which services were accessed by that user, when they were accessed, and which policy granted them that permission. This is great for maintaining least-privilege, because if a user has permissions to a service they never use, those permissions should be removed.

## Best practices
To close out the portion on Identity andAccess management, here are some tips and best bractices for securing your AWS account.

### Dont use the root account.
Every AWS account comes with a pre-built super-admin account called the "root" account. This account can do absolutely anything, and cannot be restricted. It's tempting to just use it, especially if you are the only one using the AWS account, but the cost is too high. If your root credentials leak, you might as well close your account because anyone with those credentials can do anything they choose. Instead, when you first create your account, use the root account to create another IAM user and give them admin rights and use this user to setup and manage your AWS account. Then enable MFA for the root account, make the password long and complex (like 25+ digits), and just leave it there until you actually need it.

### Use the security tools AWS gives you.
AWS has many MFA options for your user acounts, use them. Set up roles and policies to enable least-privilege. Use Accesss Keys, especially for progromatic user account used in scripts and programs. Create one user per person, or per programatic access, don't share credentials. AWS gives you all these tools for free with no limits so use them. Don't be complacent, because that's where mistakes and real problems arise.

### Apply policies to groups, not users.
If a user needs a permission, they should be added to a group that has that permission. This makes permission management much more scalable and auditable. If they need a specific set of permissions that doesnt exist in a group, create a new group, you dont have a limit to those or policies, so use them!

### Audit IAM frequently.
"Frequently" is a somewhat relative term that depends on the scale and turnover rate of your account. If you have two or three user accounts that don't change permissions much, you could probably do an audit every 6 or 12 months and be safe. Not much is changing in that environment. But if you are creating and removing multiple accounts a month, monthly or even weekly audits may be mor eappropriate, depending on your risk tolerance.

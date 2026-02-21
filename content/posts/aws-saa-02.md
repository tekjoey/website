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

## Roles

We saw in the last post that you can assign policies to user accounts to allow/deny certain actions within your AWS account. But what if you wanted a third party to perform some kind of action on your account? What about an automatied system? 
Is there any way to limit their access, or do they just get full *carte blanche* access to your entire account?

Well, of course you can limit their access! The feature is *IAM Roles*. These are essentially the same kind of security policies discussed in the last post, but applied to different identities or agents.

From Amazon's own documentation:
> A role is an IAM identity that you can create in your account that has specific permissions. An IAM role has some similarities to an IAM user. Roles and users are both AWS identities with permissions policies that determine what the identity can and cannot do in AWS. However, instead of being uniquely associated with one person, a role can be assumed by anyone who needs it. A role does not have standard long-term credentials such as a password or access keys associated with it. Instead, when you assume a role, it provides you with temporary security credentials for your role session.

This is huge for security a limiting the potential blast-radius.

So a role is a set of security permissions that can be added or removed from an entity. So you could write a Lambd function that, for example, disables users that haven't logged in in 90 days. 
That function would take on a role that allows it certain permissions, in this case only the permissions for viewing an account's attributes, and disabling it, nothing else.
Or you have a third party contractor, say a physical security team, that has a VM that houses their door access program. They could assume a role to manage one particular VM, but nothing else.
This can even be used by IAM accounts within your AWS account. A help desk analyst could have a standard low set of permissions, but if they needed to restart a VM, they could assume a particular role, perform the action, then remove that role, so their account no longer has those permissions.



## IAM Credential Report

## IAM Access Advisor

## Best practices
- dont use root account
- - one user per per person
  - policies at groups, not users
  - use mfs
  - use roles
  - use access keys
  - audit access and identities

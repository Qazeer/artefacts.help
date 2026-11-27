---
title: Procedure - Required privileges
summary: 'AWS CLI access requires credentials to be locally configured. While various authentication methods are supported, some tools may only support IAM user long-term credentials.\n\nThe ReadOnlyAccess and SecurityAudit managed policies can be attached to the principal used to conduct the investigations in order to grant the required permissions.'
keywords: Access key ID, Secret access key, aws configure, ReadOnlyAccess, SecurityAudit
tags:
  - aws
  - guides
last_updated: 2024-01-27
sidebar: sidebar
permalink: aws_required_privileges.html
folder: aws
---

### AWS CLI access

The `AWS Command Line Interface (AWS CLI)` can be used to access AWS resources
through a command-line utility. To set up the `AWS CLI` environment, notably the
configuration of credentials, the `aws configure` command may be used.

The `aws configure` command will ask for the following information, which will
be stored (in cleartext) in the `config` and `credentials` files (by default in
a `.aws` folder in the current user's home directory):

  - `Access key ID`.

  - `Secret access key`.

  - AWS default region.

  - Output format.

To create an `Access key ID` and `secret access key`, refer to the
[AWS official documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-authentication-user.html).

### ReadOnlyAccess and SecurityAudit managed policies

The [`ReadOnlyAccess managed policy`](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/ReadOnlyAccess.html)
and [`SecurityAudit managed policy`](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/SecurityAudit.html)
can be attached to the principal used to retrieve the `CloudTrail` logs and
perform the security review in order to grant the required and necessary
permissions.

Specific tools may also require additional permissions. For example,
[`Invictus-AWS`](https://github.com/invictus-ir/Invictus-AWS) notably requires
the permissions to write the exported logs to a specific `S3` bucket.

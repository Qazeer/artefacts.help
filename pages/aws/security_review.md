---
title: Basic security review
summary: 'The AWS account state and configuration (IAM users, users access keys, IAM roles and policies, compute instances, storage objects, etc.) can be reviewed manually using the AWS CLI utility or in a more automated fashion using third-party tools such as Scout Suite.'
keywords: AWS CLI, iam, Scout Suite
tags:
  - aws
last_updated: 2024-01-27
sidebar: sidebar
permalink: aws_security_review.html
folder: aws
---

### Manual enumeration with AWS CLI

The following commands can be used to retrieve basic information about the AWS
account:

```bash
# Retrieves information about the IAM entity usage (number of users, groups, roles, etc.).
aws iam get-credential-report

# Lists the IAM users, with their creation date and password last used timestamp.
aws iam list-users

# Lists the users'access keys, with their status and creation date.
aws iam list-access-keys

# Lists the IAM groups (collection of IAM users that can be associated with specific permissions).
aws iam list-groups

# Retrieves the groups associated with a given user.
aws iam list-groups-for-user --user-name "<USERNAME>"

# Lists the IAM roles (an IAM identity that can be associated with specific permissions. An IAM role can be assumed can users allowed to do so).
aws iam list-roles

# Lists all the IAM policies (an IAM policy grants IAM identities - users, groups, or roles - to resources. Permissions in the policies determine whether an IAM principal (user or role) request is allowed or denied.)
aws iam list-policies

# Lists all the IAM policy names and ARN.
aws iam list-policies --query 'Policies[*].[PolicyName, Arn]' --output text

# Lists the metadata of the specified IAM policy.
aws iam list-policies --query 'Policies[?PolicyName==`<POLICY_NAME>`]'

# Retrieves the IAM policies associated with the specified user / group / role.
# Inline IAM policies embedded in the specified IAM user.
aws iam list-user-policies --user-name "<USERNAME>"
# IAM policies attached to the specified IAM user.
aws iam list-group-policies --group-name "<GROUPNAME>"
aws iam list-role-policies --role-name "<ROLENAME>"

# Lists all the IAM users, groups, and roles that the specified policy is attached to.
# Example policy ARN: arn:aws:iam::aws:policy/service-role/AWSApplicationMigrationReplicationServerPolicy
aws iam list-entities-for-policy --policy-arn "<POLICY_ARN>"

# Lists the version associated with an IAM policy.
aws iam list-policy-versions --policy-arn "<POLICY_ARN>"

# Retrieves the permissions associated with an IAM policy (and policy version).
aws iam get-policy-version --policy-arn "<POLICY_ARN>" --version-id "<v1 | POLICY_VERSION_ID>"

# Lists the S3 buckets in the account.
aws s3 ls

# Retrieves more detailed (compared to s3 ls) information on a bucket (and bucket files).
aws s3api list-objects --bucket <BUCKET_NAME>

# Download / upload files from / to a S3 bucket.
# Source / destination for s3://<BUCKET> or local path.
aws s3 cp [--recursive] <SOURCE> <DEST>
```

### Automated enumeration with ScoutSuite

[`Scout Suite`](https://github.com/nccgroup/ScoutSuite) leverage the API
provided by AWS (as well as other possible Cloud providers) to automatically
enumerate the configuration of the account. It can be used to quickly gather
information on the attack surface of the AWS account across all regions.

```bash
python3 scout.py aws [--access-key-id <ACCESS_KEY_ID>] [--secret-access-key <ACCESS_KEY_SECRET>]
```

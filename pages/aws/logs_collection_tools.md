---
title: Procedure - Logs search and collection tools
summary: 'CloudTrail logs can be collected in a specific region or across all regions using third-party tools, such as Invictus-AWS.'
keywords: awsCloudTrailDownload, Invictus-AWS, awslogs
tags:
  - aws
  - guides
last_updated: 2024-01-27
sidebar: sidebar
permalink: aws_logs_collection_tools.html
folder: aws
---

### awsCloudTrailDownload

The [`awsCloudTrailDownload.py`](https://github.com/dlcowen/sansfor509/blob/main/AWS/awsCloudTrailDownload.py)
Python script can be used to download the `CloudTrail` logs across all regions.

```bash
python awsCloudTrailDownload.py
```

### Invictus-AWS

The [`Invictus-AWS`](https://github.com/invictus-ir/Invictus-AWS) Python script
can be used to retrieve information about the environment (service usage and
configuration) and export logs from a number of sources (`CloudTrail`,
`CloudWatch`, `S3 Access Logs`, ...) to an `S3` bucket. `Invictus-AWS` is
region bound.

In addition to the
[`ReadOnlyAccess managed policy`](https://docs.aws.amazon.com/aws-managed-policy/latest/reference/ReadOnlyAccess.html),
`Invictus-AWS` also requires
[specific permissions, as defined in the provided policy](https://github.com/invictus-ir/Invictus-AWS/blob/main/source/files/policy.json).

As stated in `Invictus-AWS`'s README, the tool is divided into 4 different
steps (which can be run independently):

  - The first step enumerates the activated AWS services and their details.

  - The second step retrieves configuration details about the activated
    services.

  - The third step extracts the available logs for the activated services.

  - The fourth and last step analyzes CloudTrail logs, and only CloudTrail logs,
    by running `Athena` queries against them.

By default, steps 1 to 3 are executed. Step 4 is optional and must be run
separately.

```bash
# Configures the required API access.
aws configure

# Without any parameter, invictus-aws.py starts in an interactive walkthrough mode.
# -r <REGION>: specifies the region (such as "us-east-1") to retrieve the logs from.
# -A <REGION>: retrieves the logs from all regions, starting with the specified (optional) region.
# -w <local | cloud>: defines whether the results should be stored in an S3 bucket only or if the results should also be downloaded to the local storage.
# -s <STEPS>: comma-separated list of the steps to execute (1,2,3 by default, step 4 must be executed alone).
# -start <YYYY-MM-DD> -end <YYYY-MM-DD>: limits the collection of the CloudTrail logs (step 3) to the specified time range (recommended, as the collection can otherwise be very long).
python3 invictus-aws.py [-r <REGION> | -A <REGION>] -w <local | cloud> [-s <STEPS>] [-start <YYYY-MM-DD> -end <YYYY-MM-DD>]

# Downloads the exported/collected elements from invictus-aws.py locally.
aws s3 cp --recursive s3://<INVICTUS_BUCKET> <EXPORT_FOLDER>
```

### awslogs

The [`awslogs`](https://github.com/jorgebastida/awslogs) utility can be used to
access and filter the AWS `CloudWatch` logs. `awslogs` requires the permissions
associated with the [`CloudWatchLogsReadOnlyAccess` policy](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/iam-identity-based-access-control-cwl.html).

```bash
awslogs <get | groups | streams> [--aws-region "<AWS_REGION>"] [--aws-access-key-id "<ACCESS_KEY_ID>"] [--aws-secret-access-key "<ACCESS_KEY_SECRET>"]

# Lists the existing log groups.
awslogs groups

# Lists the streams in the specified log group.
awslogs streams <LOG_GROUP>

# Retrieves the logs from all log groups/streams or from the specified log group/stream.
# The start/end filtering supports multiple filtering options: DD/MM/YYYY HH:mm, <INT><m | h | d | w>.
awslogs get <ALL | LOG_GROUP> <ALL | LOG_GROUP_STREAM> -s <START> -e <END>
```

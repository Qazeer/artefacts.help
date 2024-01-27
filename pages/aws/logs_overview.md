---
title: Logs overview
summary: 'A number of log sources are available in AWS, that can be useful for incident response purposes:\n\n- CloudTrail: logs management operations made in the AWS account through Amazon Management Console actions and API calls. CloudTrail is enabled by default.\n\n- CloudWatch: logs system performance metrics such as CPU usage, filesystem or network inputs/outputs, etc. Some AWS products automatically push metrics to CloudWatch (for free), while other services may require additional configuration to push metrics.\n\n- AWS Config: records the configuration state of a number of resources in the AWS account, either periodically or continuously on configuration item change. AWS Config is not enabled by default.\n\n- S3 Access Logs: logs bucket-level activities, i.e. access, upload, modification, and deletion of data stored in a S3 bucket. S3 Access Logs are not enabled by default and must be enabled on a per bucket basis.\n\n- VPC Flow Logs: logs VPC-level IP network traffic. Different version of VPC Flow Logs, 2 to 5 to date, can be enabled with higher versions recording an increased number of fields per record. VPC Flow Logs are not enabled by default and must be enabled either at the VPC, subnet, or Elastic Network Interfaces level.\n\n- WAF Logs: logs requests processed by the AWS WAF service.'
keywords: CloudTrail, CloudWatch, AWS Config, S3 Access Logs, VPC Flow Logs, WAF Logs
tags:
  - aws
last_updated: 2024-01-27
sidebar: sidebar
permalink: aws_logs_overview.html
folder: aws
---

A number of log sources are available in `AWS`, that can be useful for incident
response purposes:

| Name | Conditions | Description |
|------|------------|-------------|
| `CloudTrail` | Enabled by default. | Logs every operation conducted in the AWS account through `Amazon Management Console` actions and API calls. Essentially logs all management operations API calls made in the account. <br><br> For each action / operation, the following information are logged: <br> - Unique Event ID. <br> - Event name (such as `ListPolicies`, `AssumeRole`, etc.). <br> - The timestamp of the operation. <br> - The region the operation was conducted in. <br> - Information on who realized the action (IAM identity, source IP address, user agent). <br> - The eventual impacted resource. <br> - The eventual request parameters. <br> - ...  |
| `CloudWatch` | Some AWS products automatically push metrics to Amazon CloudWatch (for free), while other services may require additional configuration to push metrics. | Logs system performance metrics such as CPU usage, filesystem or network inputs/outputs, etc. <br><br> An additional `CloudWatch` agent can be installed on EC2 hosts to forward OS-level logs to `CloudWatch`. <br><br> `CloudTrail` logs can be forwarded to `CloudWatch`, for instance to configure automated alerting. |
| `AWS Config` | AWS Config is not enabled by default. | Records the configuration state of a number of resources (`EC2`, `VPC`, security groups, etc.) in the AWS account. <br><br> There are two frequencies at which `AWS Config` can deliver configuration items: periodic and continuous. Periodic recording delivers configuration data once every 24 hours, only if a change has occurred. Continuous recording delivers configuration items whenever a change occurs. <br><br> If enabled, `AWS Config` can be used to detect change in configuration and retrieve historical data on configuration changes (who and when was a given resource created / modified). |
| `S3 Access Logs` | `S3 Access Logs` are not enabled by default and must be enabled on a per bucket basis. | Logs bucket-level activities, i.e. access, upload, modification, and deletion of data stored in a `S3 bucket` (versus operation on the bucket object itself as logged by `CloudTrail`). |
| `CloudWatch` | `VPC Flow Logs` are not enabled by default and must be enabled either at the `VPC`, subnet, or `Elastic Network Interfaces` (`ENI`) level. | Logs `IP` network traffic to `CloudWatch` or an `S3` bucket. <br><br> Different version of `VPC Flow Logs`, 2 to 5 to date, can be enabled. Higher versions record an increased number of fields per record. The `version 2`, chosen by default, records the following fields (in order): <br> - version number. <br> - account id (AWS account ID of the owner of the source network interface for which traffic is recorded). <br> - interface id (ID of the network interface for which the traffic is recorded). <br> - source address. <br> - destination address. <br> - source port. <br> - destination port. <br> - network protocol. <br> - number of packets transferred during the "flow" log. <br> - number of bytes transferred during the "flow" log. <br> - start of the "flow" log. <br> - end of the flow log. <br> - whether the traffic was accepted (`ACCEPT`) or rejected (`REJECT`). <br> - status of the flow log. <br><br> Data is aggregated over one or ten minutes intervals, depending on the flow log definition. <br><br> For more information on `VPC Flow Logs`, refer to the official [AWS documentation](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html). |
| `WAF Logs` | `WAF Logs` require the use of the `AWS WAF` service. | Logs requests processed by the `AWS WAF` service. `WAF Logs` can notably be forwarded to `CloudWatch` or stored in a `S3` bucket. <br><br> Information about the request (source IP, eventual requests headers, eventual parameters, etc.) as well as the rule matched are logged. |

### References

  - [AWS - AWS Service Logs](https://docs.aws.amazon.com/solutions/latest/centralized-logging-with-opensearch/aws-service-logs.html)

  - [Prashant Lakhera - VPC Flow Logs](https://devopslearning.medium.com/vpc-flow-logs-45eca8ae718b)

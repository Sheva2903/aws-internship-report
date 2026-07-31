---
title: "Step-by-step implementation"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Overview

This workshop covers securely migrating the database of DIY Shop - a handmade-goods e-commerce application - from local PostgreSQL to Amazon RDS:

- Placing RDS in a private subnet with no public Internet exposure, restricting connections to only the backend's Security Group
- Granting the application S3 access via an IAM Role instead of static access keys.

#### Content

- [Create VPC and Networking](5.3.1-create-vpc-and-networking/)
- [Security Group Configuration](5.3.2-security-group-configuration/)
- [EC2, RDS creation and secure connection](5.3.3-ec2-rds-creation-secure-connection/)
- [S3 and IAM Role least-privilege](5.3.4-s3-iam-least-privilege/)
- [CloudWatch and SNS](5.3.5-cloudwatch/)

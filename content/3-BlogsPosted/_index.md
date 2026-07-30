---
title: "Blogs Posted"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

This section introduces three blog posts about cloud security, focusing on responsibility boundaries, shared infrastructure, and secure data deletion in AWS environments.

### [Blog 1 - SHARED RESPONSIBILITY MODEL ON AWS](3.1-Blog1/)

This blog explains how security responsibilities are divided between AWS and the customer. It also shows how the responsibility boundary changes across different cloud services and discusses practical controls such as IAM, Amazon S3 public access settings, logging, and monitoring.

### [Blog 2 - MULTI-TENANCY AND VIRTUALIZATION IN AWS](3.2-Blog2/)

This blog examines how AWS supports multiple tenants on shared cloud infrastructure through virtualization and isolation technologies. It also discusses customer security controls such as IAM, VPC, Security Groups, KMS, CloudTrail, and CloudWatch, together with AWS technologies such as the Nitro System and Firecracker microVMs.

### [Blog 3 - CRYPTOGRAPHIC DELETION FOR CLOUD STORAGE](3.3-Blog3/)

This blog introduces FADE and the concept of cryptographic deletion for cloud storage. Instead of relying only on physical deletion, the approach separates encrypted data from its cryptographic keys and makes the data permanently unreadable by destroying the policy-bound decryption key.
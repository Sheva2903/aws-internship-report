---
title: "Introduction"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

This workshop covers securely migrating the database of DIY Shop - a handmade-goods e-commerce application - from local PostgreSQL to Amazon RDS:

- Placing RDS in a private subnet with no public Internet exposure, restricting connections to only the backend's Security Group
- Granting the application S3 access via an IAM Role instead of static access keys.

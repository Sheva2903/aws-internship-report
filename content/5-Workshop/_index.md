---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploying DIY Shop on AWS

#### Overview

This workshop presents the process of deploying the DIY Shop application on AWS.

DIY Shop is a bilingual online store for handmade products. The system includes a Spring Boot backend, a PostgreSQL database, product image storage, and monitoring.

The workshop covers the main AWS components used in the project:

- **Amazon VPC** for network isolation
- **Security Groups** for traffic control
- **Amazon EC2** for running the backend
- **Amazon RDS for PostgreSQL** for storing application data
- **Amazon S3** for storing product images
- **AWS IAM** for managing service permissions
- **Amazon CloudWatch** for logs and monitoring

The implementation follows a simple and secure architecture. The application server is placed in a public subnet, while the database is placed in private subnets. Access between services is limited through Security Groups and IAM roles.

#### Workshop Objectives

After completing this workshop, you will be able to:

- Create a VPC and configure the required networking components
- Configure Security Groups for EC2 and RDS
- Launch an EC2 instance and deploy the Spring Boot backend
- Create an Amazon RDS PostgreSQL database
- Connect EC2 to RDS securely
- Store product images in Amazon S3
- Configure an IAM role with least-privilege permissions
- Monitor the application with Amazon CloudWatch
- Clean up the AWS resources after completing the workshop

#### Content

1. [Workshop Introduction](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequisite/)
3. [Step-by-step Implementation](5.3-Implementation-Testing/)
4. [Clean Up](5.4-Cleanup/)
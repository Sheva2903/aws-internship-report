---
title: "Proposal"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

{{% notice warning %}}
⚠️ **Note:** Some words will be kept in English version so as to be ensure the meaning for the context.
{{% /notice %}}

In this section, you need to summarize the contents of the workshop that you **plan** to conduct.

# DIY SHOP

## E-commerce management leverages AWS services

### 1. Executive Summary

DIY Shop is a website built for a handmade-goods and hand-painted artwork business, supporting the shop owner (single-seller) in managing and operating the store online. The application lets customers browse the product catalog and place orders through the website using two payment methods: (1)Cash on Delivery (COD) and (2) Bank Transfer (VietQR). After successfully placing an order, the system lets customers track their order status using the order code received at checkout. On the operational side, a dashboard supports the shop owner in managing the product catalog, controlling inventory, and processing order statuses, with payment status tracked independently from delivery status. The website leverages AWS services to realize automated operations and management.

### 2. Problem Statement

### What’s the Problem?

Sellers of handmade goods and paintings currently lack a dedicated online sales channel to showcase products, take orders, and track inventory. This makes the process error-prone as order volume grows, since there is no shared source of data between the seller and the customer. As a single, non-technical seller - operating without a dedicated technical or operations team - the owner would have to handle tasks beyond their expertise, such as securing customers' personal data (name, phone number, and address collected at checkout) and handling server incidents. This can lead to time-consuming recovery efforts, and even the risk of exposing customer information.

### The Solution

The system is deployed on **Amazon EC2**, running as a managed `systemd` service, connected to an **Amazon RDS PostgreSQL instance** placed in a private subnet (with least-privilege-restricted connectivity); product images are stored on **Amazon S3** via an **IAM role**. **Amazon CloudWatch** collects application and access logs, deriving error metrics from them through metric filters, then triggers email alerts when the error rate exceeds a threshold. **GitHub Actions** automates build, test, and deployment on every code change.

### 3. Solution Architecture

DIY shop is deployed inside a dedicated VPC spanning 2 Avaibility Zones (AZs) to ensure high availability. User requests pass through three layers before reaching the application from Route 32 to CloudFront and WAF then Application Load Balancer. The application runs on EC2 instances managed by an Auto Scaling Group, distributed across 2 AZs in a public subnets. The data consists of RDS PostgreSQL in Multi-AZ mode and S3 for product image storage. The entire operational lifecycle is supported by CloudWatch (monitoring), SNS (alerting), Secrets Manager (credential management), IAM (access control), and GitHub Actions (CI/CD automation).

![DIY Shop Architecture](final_arch.png)

### AWS Services Used

- **Amazon VPC**: Provides an isolated network with 2 public and 2 private subnets across 2 AZ
- **Amazon EC2**: Hosts the Spring Boot backend, running as a managed `systemd` service in a public subnet
- **Amazon RDS**: Managed relational database and all core transactional data
- **Amazon S3**: Stores product images; the database keeps only a reference
- **Route 53**: resolves the domain to CloudFront
- **CloudFront**: CDN, caches static assets at edge locations
- **AWS IAM**: Enforces least privilege
- **Amazon CloudWatch**: Centralizes application and access logs into log group, derives error-count metrics from those logs via metric filters, and evaluates alarms against them
- **Amazon SNS**: Delivers an email notification whenever a CloudWatch Alarm enters the `ALARM` state
- **Secrets Manager**: stores and automatically rotates DB and seller credentials

### Component Design

- **Edge and Traffic Distribution**: Route 53 resolves the domain to an alias record pointing at CloudFront (static asset caching, WAF attachment). Every request is inspected by WAF before being forwarded to ALB (load balancing, TLS termination).
- **Application Tier**: Auto Scaling Group manages EC2 instances (2 AZs), each running a single Spring Boot `jar` with the React frontend bundled in the same origin, so no CORS needed
- **Data Tier**: RDS PostgreSQL Multi-AZ, use S3 to store product images via IAM Role and presigned URLs
- **Security and Credential Management**: Secret Manager stores DB crendentials; IAM enforces least-privilege access for EC2
- **Observability**: CloudWatch Agent collects logs and metrics. Alarm is responsible for check conditions based on
- **CI/CD**: GitHub Actions automatically builds, tests, and deploys on every push

### 4. Technical Implementation

**Implementation Phases**
The project was delivered in 2 consecutive phases: (1) Core Foundation, (2) Security and Reliability.

About core foundation (2 weeks):

- Provisioned the VPC, Security Groups, RDS PostgreSQL, and EC2; migrated the schema via Flyway onto the real RDS instance
- Set up an S3 bucket with a least-privilege IAM Role for product image storage, validated with a real upload test
- Stabilized the application via a `systemd` service, wired up CloudWatch Agent , Metric Filters, Alarms and SNS. Then, built a GitHub Actions CI/CD pipeline for automated build and deploy, and attached an Elastic IP

About security and reliability (1 weeks):

- **Secrets Manager**: moved all credentials from plaintext files on EC2 to secrets with an audit trail and automatic rotation
- **Multi-AZ RDS**: enabled Multi-AZ on the existing instance, eliminating the single point of failure at the data tier
- **Application Load Balancer**: added a load-balancing and TLS-termination layer, forming the foundation for the Auto Scaling Group
- **AWS WAF**: attached a Web ACL to filter application-layer attacks and rate-limit the login endpoint

**Technical Requirements**

- **Backend**: Java 17, Spring Boot 4.0.6, Spring Data JPA, Flyway (schema versioning), Spring Security (form login + CSRF for the seller portal)
- **Frontend**: React + Vite + TypeScript + Tailwind CSS, bundled directly into Spring Boot's static/ (same-origin, no CORS configuration needed)
- **Database**: PostgreSQL 18 (RDS), schema fully managed through Flyway migrations versioned alongside the source code
- **Infrastructure**: All AWS resources were provisioned via the Console (no IaC, due to time constraints), with each configuration step documented for reuse in the Workshop lab
- **CI/CD**: GitHub Actions, build and test using a PostgreSQL service container that mirrors the real RDS environment in CI

### 5. Timeline & Milestones

**Project Timeline**

- Internship (Months 1-2): 2 months.
  - Month 1: Study AWS and doing labs
  - Month 2: Design architecture, implement, test and launch

### 6. Budget Estimation

You can find mybudget estimation on the [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=aa60066de7ae4feb705c3dd690f39a0977a659b9).

### Infrastructure Costs

- Amazon Route 53: $0.50/month (Hosted Zones, additional records in hosted zones)
- Amazon WAF: $7.00/month (Web ACLs utilized 1 per month, 2 added rules)
- Amazon Cloudfront $0.00/month (Free plan)
- Amazon S3: $0.20/month (Standard storage - 8Gb/month)
- Amazon EC2: $4.16/month (Linux, t3.micro, enable monitoring)
- Amazon RDS: $100.36/month (100Gb, db.t4g.micro, on-demand only)

Total: $121.22/month and $1454.64/12 months

### 7. Risk Assessment

#### Risk Matrix

- Single Point of Failure (EC2/RDS): High impact, medium probability.
- Credential Leakage / Data Breach: High impact, low probability.
- Cost Overruns (RDS Multi-AZ, always-on EC2): Medium impact, medium probability.
- Faulty Deployment: Medium impact, low probability.

#### Mitigation Strategies

- Availability: Auto Scaling Group across 2 AZs behind an ALB, RDS in Multi-AZ mode.
- Security: Secrets Manager for credential rotation, IAM least privilege, WAF Web ACL.
- Cost: CloudWatch billing alarms, right-sized instances (t3.micro/db.t4g.micro).
- Deployment: GitHub Actions CI/CD runs build and tests before every deploy.

#### Contingency Plans

- Roll back to the previous stable build via GitHub Actions if a deployment fails.
- Restore RDS from an automated snapshot in case of data corruption.

### 8. Expected Outcomes

- Ensures baseline security through IAM and WAF
- Ensures reliable Flyway schema migrations to RDS
- Logs and metrics are visualized through CloudWatch dashboards, with email notifications sent whenever an alarm is triggered

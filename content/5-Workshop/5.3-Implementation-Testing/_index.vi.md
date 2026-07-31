---
title: "Truy cập S3 từ VPC"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Sử dụng Gateway endpoint

Workshop này hướng dẫn di chuyển an toàn cơ sở dữ liệu của DIY Shop - một ứng dụng thương mại điện tử bán đồ thủ công - từ PostgreSQL cục bộ sang Amazon RDS:

- Đặt RDS trong private subnet không lộ ra Internet công cộng, giới hạn kết nối chỉ cho Security Group của backend
- Cấp quyền truy cập S3 cho ứng dụng thông qua IAM Role thay vì access key tĩnh.

#### Nội dung

- [Create VPC and Networking](5.3.1-create-vpc-and-networking/)
- [Security Group Configuration](5.3.2-security-group-configuration/)
- [EC2, RDS creation and secure connection](5.3.3-ec2-rds-creation-secure-connection/)
- [S3 and IAM Role least-privilege](5.3.4-s3-iam-least-privilege/)
- [CloudWatch and SNS](5.3.5-cloudwatch/)

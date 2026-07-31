---
title: "Triển khai từng bước"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Tổng quan

Trong phần này, bạn sẽ triển khai từng bước hạ tầng AWS bảo mật cho ứng dụng DIY Shop.

Quá trình triển khai bắt đầu từ việc tạo VPC và cấu hình networking, bao gồm public subnets, private subnets, route tables, Internet Gateway và S3 Gateway VPC Endpoint. Sau đó, bạn sẽ cấu hình Security Groups để kiểm soát kết nối giữa EC2 và RDS. Tiếp theo, bạn sẽ tạo EC2 instance, deploy backend Spring Boot, kết nối backend tới Amazon RDS một cách bảo mật, cấu hình quyền truy cập S3 thông qua IAM Role theo nguyên tắc least privilege, và cuối cùng thiết lập CloudWatch để thu thập logs, tạo metrics, alarms và gửi cảnh báo qua email.

Mục tiêu của phần này là đưa ứng dụng từ môi trường local lên AWS, đồng thời giữ database trong private subnet, tránh sử dụng AWS access key tĩnh trong ứng dụng, và bổ sung khả năng giám sát cơ bản để vận hành và xử lý lỗi.

#### Nội dung

- [Tạo VPC và cấu hình mạng](5.3.1-create-vpc-and-networking/)
- [Cấu hình Security Group](5.3.2-security-group-configuration/)
- [Tạo EC2, RDS và kết nối bảo mật](5.3.3-ec2-rds-creation-secure-connection/)
- [S3 và IAM Role theo nguyên tắc least privilege](5.3.4-s3-iam-least-privilege/)
- [Giám sát với CloudWatch](5.3.5-cloudwatch/)
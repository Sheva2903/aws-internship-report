---
title: "Giới thiệu"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

Workshop này hướng dẫn cách di chuyển cơ sở dữ liệu của DIY Shop - một ứng dụng thương mại điện tử bán đồ thủ công - từ PostgreSQL chạy local sang Amazon RDS một cách bảo mật:

- Đặt RDS trong private subnet, không public ra Internet, và chỉ cho phép kết nối từ Security Group của backend
- Cấp quyền truy cập S3 cho ứng dụng thông qua IAM Role thay vì sử dụng access key tĩnh.
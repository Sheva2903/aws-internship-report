---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Triển khai DIY Shop trên AWS

#### Tổng quan

Workshop này trình bày quá trình triển khai ứng dụng DIY Shop trên AWS.

DIY Shop là một website bán đồ handmade hỗ trợ tiếng Việt và tiếng Anh. Hệ thống gồm backend Spring Boot, cơ sở dữ liệu PostgreSQL, nơi lưu trữ hình ảnh sản phẩm và chức năng giám sát ứng dụng.

Workshop sử dụng các thành phần AWS chính sau:

- **Amazon VPC** để tạo mạng riêng cho hệ thống
- **Security Groups** để kiểm soát lưu lượng
- **Amazon EC2** để chạy backend
- **Amazon RDS for PostgreSQL** để lưu dữ liệu ứng dụng
- **Amazon S3** để lưu trữ hình ảnh sản phẩm
- **AWS IAM** để quản lý quyền truy cập giữa các dịch vụ
- **Amazon CloudWatch** để theo dõi log và hoạt động của ứng dụng

Kiến trúc được xây dựng theo hướng đơn giản và an toàn. Application server được đặt trong public subnet, trong khi database được đặt trong các private subnet. Quyền truy cập giữa các dịch vụ được giới hạn bằng Security Group và IAM Role.

#### Mục tiêu workshop

Sau khi hoàn thành workshop, người thực hiện có thể:

- Tạo VPC và cấu hình các thành phần mạng cần thiết
- Cấu hình Security Group cho EC2 và RDS
- Khởi tạo EC2 và triển khai backend Spring Boot
- Tạo database PostgreSQL trên Amazon RDS
- Kết nối an toàn giữa EC2 và RDS
- Lưu trữ hình ảnh sản phẩm trên Amazon S3
- Cấu hình IAM Role theo nguyên tắc đặc quyền tối thiểu
- Theo dõi ứng dụng bằng Amazon CloudWatch
- Dọn dẹp các tài nguyên AWS sau khi hoàn thành workshop

#### Nội dung

1. [Giới thiệu workshop](5.1-Workshop-overview/)
2. [Các bước chuẩn bị](5.2-Prerequiste/)
3. [Triển khai từng bước](5.3-Implementation-Testing/)
4. [Dọn dẹp tài nguyên](5.4-Cleanup/)
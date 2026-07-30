---
title: "Các bài blog đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Phần này giới thiệu ba bài blog về bảo mật cloud, tập trung vào ranh giới trách nhiệm, hạ tầng dùng chung và cơ chế xóa dữ liệu an toàn trong môi trường AWS.

### [Bài 1 - SHARED RESPONSIBILITY MODEL TRÊN AWS](3.1-Blog1/)

Bài viết giải thích cách AWS và khách hàng phân chia trách nhiệm bảo mật. Nội dung cũng làm rõ cách ranh giới trách nhiệm thay đổi theo từng loại dịch vụ cloud, đồng thời đề cập đến các biện pháp thực tế như IAM, kiểm soát public access của Amazon S3, logging và monitoring.

### [Bài 2 - MULTI-TENANCY VÀ VIRTUALIZATION TRONG AWS](3.2-Blog2/)

Bài viết phân tích cách AWS hỗ trợ nhiều tenant trên cùng hạ tầng cloud thông qua virtualization và các lớp isolation. Nội dung cũng trình bày các biện pháp bảo mật phía khách hàng như IAM, VPC, Security Groups, KMS, CloudTrail và CloudWatch, cùng với các công nghệ của AWS như Nitro System và Firecracker microVMs.

### [Bài 3 - CRYPTOGRAPHIC DELETION CHO CLOUD STORAGE](3.3-Blog3/)

Bài viết giới thiệu FADE và khái niệm cryptographic deletion cho cloud storage. Thay vì chỉ phụ thuộc vào việc xóa vật lý, cơ chế này tách dữ liệu đã mã hóa khỏi khóa mật mã và làm cho dữ liệu không thể đọc lại bằng cách phá hủy khóa giải mã được ràng buộc với policy.
---
title: "Các bước chuẩn bị"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

- Một tài khoản AWS có quyền tạo tài nguyên trong các dịch vụ: VPC, EC2, RDS, S3, IAM. Free Tier là đủ - mọi bước trong lab này đều nằm trong giới hạn Free Tier nếu sử dụng đúng loại instance và dung lượng lưu trữ được khuyến nghị.
- Tất cả tài nguyên như VPC, EC2, RDS và S3 được tạo trong cùng một Region, ví dụ: `ap-southeast-2 (Singapore)`.
- Công cụ cần dùng: chỉ cần một trình duyệt web để truy cập AWS Console. Tất cả các bước đều được thực hiện thủ công qua giao diện web để tập trung vào việc hiểu từng cấu hình.
- Một SSH client để kết nối tới EC2 sau khi tạo xong, ví dụ: MobaXterm, hoặc lệnh `ssh` có sẵn trên Mac/Linux/Git Bash.
- Kiến thức nền hữu ích: hiểu cơ bản về VPC, Subnet, Security Group, và từng chạy PostgreSQL hoặc ứng dụng Spring Boot ở local trước đó. Không bắt buộc, nhưng sẽ giúp bạn theo dõi lab dễ hơn.
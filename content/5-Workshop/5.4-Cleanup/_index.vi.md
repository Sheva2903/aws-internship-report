---
title: "Dọn dẹp tài nguyên"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

### Dọn dẹp tài nguyên

Sau khi hoàn thành workshop, cần xóa các tài nguyên AWS đã tạo để tránh phát sinh chi phí không cần thiết.

Nên xóa tài nguyên theo thứ tự ngược với lúc tạo: xóa các tài nguyên phụ thuộc trước, sau đó mới xóa các tài nguyên nền tảng.

### 1. Xóa tài nguyên CloudWatch và SNS

1. Mở **CloudWatch Console** → **Alarms**.
2. Chọn ba alarm:
   - `common-metric`
   - `http-4xx-count`
   - `http-5xx-count`
3. Chọn **Actions → Delete**.
4. Vào **Log groups** → chọn `/diyshop/app`.
5. Mở tab **Metric filters** và xóa filter `common-metric`.
6. Thực hiện tương tự với `/diyshop/access`, sau đó xóa:
   - `http-4xx-count`
   - `http-5xx-count`
7. Xóa hai Log Group:
   - `/diyshop/app`
   - `/diyshop/access`
8. Mở **SNS Console** → **Topics** → chọn `diyshop-alerts` → **Delete**.

Khi topic bị xóa, email subscription liên kết với topic cũng sẽ được xóa theo.

### 2. Xóa EC2 và giải phóng Elastic IP

1. Mở **EC2 Console** → **Instances**.
2. Chọn instance `diyshop-app-server`.
3. Chọn **Instance state → Terminate instance**.
4. Sau khi instance bị xóa, vào **Elastic IPs**.
5. Chọn Elastic IP không còn liên kết với instance.
6. Chọn **Actions → Release Elastic IP address**.

> Lưu ý: Việc terminate EC2 không tự động giải phóng Elastic IP. Elastic IP không được sử dụng vẫn có thể phát sinh chi phí.

### 3. Xóa S3 Bucket

1. Mở **Amazon S3 Console**.
2. Chọn bucket `diyshop-s3-bucket-...`.
3. Chọn **Empty** và nhập tên bucket để xác nhận.
4. Sau khi bucket không còn object, chọn **Delete**.
5. Nhập lại tên bucket để xác nhận việc xóa.

S3 bucket phải được làm trống trước khi có thể xóa.

### 4. Xóa IAM Role và IAM Policy

1. Mở **IAM Console** → **Roles**.
2. Chọn role `diyshop-ec2-role`.
3. Trong tab **Permissions**, detach policy `diyshop-s3-product-images-policy`.
4. Xóa role `diyshop-ec2-role`.
5. Vào **Policies**.
6. Chọn policy `diyshop-s3-product-images-policy`.
7. Chọn **Delete**.

### 5. Xóa Amazon RDS

1. Mở **RDS Console** → **Databases**.
2. Chọn database `diyshop-db-instance`.
3. Nếu đang bật **Deletion protection**, chọn **Modify** và tắt tùy chọn này trước.
4. Chọn **Actions → Delete**.
5. Với môi trường lab, có thể không cần tạo final snapshot.
6. Nhập `delete me` để xác nhận.
7. Sau khi database được xóa, vào **Subnet groups**.
8. Xóa DB Subnet Group `rds-subnet-group`.

Trong môi trường production thực tế, nên tạo final snapshot trước khi xóa database.

### 6. Xóa Security Group

1. Mở **VPC Console** → **Security Groups**.
2. Xóa các Security Group đã tạo:
   - `EC2-WebApp-SG`
   - `SG-RDSPostgreSQL`

Không xóa Security Group mặc định của VPC.

### 7. Xóa VPC Endpoint

1. Mở **VPC Console** → **Endpoints**.
2. Chọn endpoint `diy-vpce-s3`.
3. Chọn **Delete VPC endpoints** và xác nhận.

Nếu endpoint được tạo tự động bằng VPC wizard, tên có thể khác với tên trên.

### 8. Xóa VPC

Thực hiện bước này cuối cùng, sau khi các tài nguyên phụ thuộc đã được xóa.

1. Mở **VPC Console** → **Your VPCs**.
2. Chọn `diy-vpc`.
3. Chọn **Actions → Delete VPC**.
4. Kiểm tra danh sách tài nguyên liên quan.
5. Xác nhận xóa VPC.

AWS Console có thể xóa cùng các thành phần còn lại như subnet, route table và Internet Gateway nếu không còn tài nguyên phụ thuộc.

### Kiểm tra sau khi dọn dẹp

Sau khi hoàn tất, kiểm tra lại:

- Trong **VPC Console**, xác nhận `diy-vpc` không còn tồn tại.
- Trong **EC2 Console**, xác nhận instance và Elastic IP đã được xóa.
- Trong **RDS Console**, xác nhận database đã được xóa.
- Trong **S3 Console**, xác nhận bucket không còn tồn tại.
- Trong **Billing Console**, kiểm tra không còn tài nguyên `diyshop-*` tiếp tục phát sinh chi phí.
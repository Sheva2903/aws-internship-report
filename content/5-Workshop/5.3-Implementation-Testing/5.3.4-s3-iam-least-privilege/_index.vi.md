---
title: "Tạo S3 Bucket và cấu hình IAM Role"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.3.4. </b> "
---

1. **Tạo S3 Bucket**

Truy cập S3 Console, chọn `Buckets`, chọn `General purpose bucket`, sau đó chọn `Create bucket`. Trong phần tạo bucket này, tôi giữ gần như toàn bộ cấu hình mặc định. Với phần tên bucket, tôi chọn `Account Regional namespace` và nhập tên bucket của mình.

Gần như giữ mặc định có nghĩa là:

- Giữ ACLs ở trạng thái `disabled`
- Tick chọn `Block all public access`
- Chọn `SSE-S3` cho encryption

2. **Tạo IAM Policy theo nguyên tắc least privilege**

Truy cập IAM console, chọn `Policies` và chọn `Create policy`. Chuyển sang tab `JSON` và thêm nội dung sau:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3policy",
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::<your-bucket-name>/<folder-name-storage>/*"
    }
  ]
}
```

Tiếp theo, nhập tên policy và nhấn `Create policy`.

![IAM Policy](iam_policy.png)

3. **Tạo IAM Role và gán Policy cho EC2**

Truy cập IAM console, chọn `Roles` và chọn `Create role`. Chọn như sau:

```txt
Trusted entity type: AWS Service
Use case: EC2
Permission policy: <your-created-policy>
Role name: <enter-role-name>
```

![IAM Role](role.png)

Kiểm tra bằng cách upload file ảnh. Trong project của tôi, có một endpoint cho `seller` để upload ảnh sản phẩm và endpoint này cũng yêu cầu `csrf` thông qua login. Vì vậy, tôi sẽ kiểm tra như sau:

```bash
[ec2-user@ip-10-0-13-67 ~]$ curl -c test/cookies.txt http://localhost:8081/api/seller/auth/csrf
{"headerName":"X-XSRF-TOKEN","parameterName":"_csrf","token":"hsXi0Inf0zGh_pPmYjDNTxMg8G85DsEabCZIzv22IFVbw7XStKPQ4L_utVSMm_fVBh35eHZD3VdYbfM3XhBx-Z-BRDdr-oLh"}[ec2-user@ip-10-0-13-67 ~]$ curl -b test/cookies.txt -c test/cookies.txt -X POST http://localhost:8081/api/seller/auth/login -H "X-XSRF-TOKEN: hsXi0Inf0zGh_pPmYjDNTxMg8G85DsEabCZIzv22IFVbw7XStKPQ4L_utVSMm_fVBh35eHZD3VdYbfM3XhBx-Z-BRDdr-oLh" -d "username=<seller-username>&password=<seller-password>"
[ec2-user@ip-10-0-13-67 ~]$ curl -b test/cookies.txt -c test/cookies.txt http://localhost:8081/api/seller/auth/csrf
{"headerName":"X-XSRF-TOKEN","parameterName":"_csrf","token":"vEaSN0YqB2ZPgx2Ev02wG4PMGsx6TW3-rD-kCSZUGZY2VVIki3GmBCMcPgdi5XiwhmCEKLOqN64Yfw7Tzw2RPh8yKKcCbWYV"}[ec2-user@ip-10-0-13-67 ~]$ curl -b test/cookies.txt -c test/cookies.txt -X POST http://localhost:8081/api/seller/products/3/images -H "X-XSRF-TOKEN: vEaSN0YqB2ZPgx2Ev02wG4PMGsx6TW3-rD-kCSZUGZY2VVIki3GmBCMcPgdi5XiwhmCEKLOqN64Yfw7Tzw2RPh8yKKcCbWYV" -F "image=@test/aws.png;type=image/png"
{"id":21,"imageUrl":"https://diyshop-s3-bucket-210039345083-ap-southeast-2-an.s3.ap-southeast-2.amazonaws.com/products/af937a81-4a72-493c-964e-b1df61294aa4.png?X-Amz-Security-Token=IQoJb3JpZ2luX2VjENL%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaDmFwLXNvdXRoZWFzdC0yIkgwRgIhAOmvFfKHOoulUtG4yzG3AZt9p2%2Bo2fYvqESvYoJQgtErAiEAw0mymWeFavy9XdCWTAMX0eatCbPuy%2BjcgtbIXr6buuUq0QUIm%2F%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FARAAGgwyMTAwMzkzNDUwODMiDLmumXrwY2rplPTOqyqlBczwsX9Qo8DMwRdFOpqOmEEieDzepmIQn3chGl799SEygDz8uhhToLpnkX4cbSumr0dJBEZ0gn9vcFRrDZQiMKo1t0x6OJicWYGrzlnyCbk81JNu1ossHSxyk5IneHtN7FSxKmjy0hCAtIWSihpgi6wW48IKNLJnVUQFb9nGMUSA4O0YMQLoH1r7FAvOQhrrzu%2BZYRm2Y1IgqnnbeK5gTs1zsL8KSISkmMupQY0hlazhxZeq5NIMjzFiPOH1p45GRjlGVK1RhARlcS7wMBy5BwH%2FGNlExJp9n4nB4MmAmXlZjXRY1Ezj0Yeu7T4GAIgwHNRk0k2erQQorPw8v487JsNeS%2ByZJirORbV3eXAhGaE7w4qx7UVYMe4vUNLandJ8FpNTBjEESK8Mkt4or8tswnEgK4csxTPaeN%2Fdj6eNGSD4k0tPoqR%2FeuAlavXY5JXIC1xsC4mywAl002xYTr6JJzULOwm9JsnsToZW6JscWP5%2BWMIxmWoVE7zvW5jyHj5VW2CVD2aNkqwMev1p7bn6i6wKHuJkhQd51T95Qwpn3AmUErtF%2BVLtNK1CKnRMwOvGe1z4dUHOe2aIEZqX5NRrILRyYm4LopPnyWUWiZce0ftxvtTrLWi2czqStWMyAKbfF%2BAwRDekeLroGNgmQgDPLACmk6I9raI5lj%2FjAvCgCQbk6ym%2FPnjSNh8s31GC7janrzhXrm9Z8wN3zzzZH8CgVHphg1g6H4ldB691qX0CRhSgOALG0gNbqc0zHo0u%2Bas1Z4mibHwgeDh8zZiPQ5P6es%2FRsrJBDcu1q3qDS0MS6NaRa8n0%2B4dWNObeSqKgCGOyDXWqSNMMA6Z83EvDdKI0CroZwWX5Vwwpo6s9EGDw2RyQkYC6Z6ZMur1sREL0auEso1LMUEYrMJSartMGOrAB63SIibslqovXrrSEJ07BZOnTXTDRZRCNlD5bO6VsTTjSs4bolNsyxIsdfK3jVNw332IWKjlj%2B1YJ0Ca1vGeJ1iyZPuPqt7HDWzVoJDJljjPGPo5cf5XO53d0ht605T%2BdEkNtpzmMQUhxNe1khjSoI2qN1rgRTz7A63bcvSS8KH7cylheRuFJ%2FHyzMPRAfZQ2rwbwO4Ieaw4wDy6stx%2BGSAUhdUcbt3SNTkhWm2IN4vk%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20260730T182724Z&X-Amz-SignedHeaders=host&X-Amz-Credential=ASIATBZ2SB654AQCVUCA%2F20260730%2Fap-southeast-2%2Fs3%2Faws4_request&X-Amz-Expires=3600&X-Amz-Signature=ccc6181fca19e3ad8b0542a2f4819d38143446f48b12162e91266bd7ac2157ef","primaryImage":false,"sortOrder":3}
```

Sau đó, tôi đã đẩy toàn bộ ảnh sản phẩm lên S3.

![S3 Upload](upload.png)
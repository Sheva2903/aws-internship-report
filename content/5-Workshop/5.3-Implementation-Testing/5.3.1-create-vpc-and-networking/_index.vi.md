---
title: "Tạo VPC và cấu hình mạng"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

1. **Tạo VPC**

![Create VPC](vpc1.jpg)
![Create VPC](vpc2.png)

{{% notice note %}}
Hãy kiểm tra tính sẵn sàng cao bằng cách đảm bảo có **ít nhất 2 AZ**. Điều này sẽ tạo ra **2 public subnets** và **2 private subnets**.
{{% /notice %}}

Sau khi tạo VPC thành công với chế độ `VPC and more`, bạn đã có sẵn 2 private subnet và 2 public subnet như sau:

![Preview VPC](previewvpc.png)

2. **Đi tới `Internet Gateways` ở thanh bên trái để tạo gateway cho VPC**

![Create internet gateway](ige.png)

3. **Sau đó nhấn `Actions`, chọn `Attach to VPC` và chọn VPC bạn vừa tạo. Nếu thành công, bạn sẽ thấy kết quả tương tự như sau:**

![Succes attachment](attachment.jpg)

4. **Cấu hình route table cho public subnets**

![Create route table](create_rtb.png)

Sau đó nhấn `Edit Route` trong `Actions` để thêm route:

```json
Destination: 0.0.0.0/0
Target: Internet Gateway - <choose your IGW>
```

![Edit Route](edit_route.jpg)

Tiếp theo, nhấn `Edit subnet association` trong `Actions` để thêm subnet. Ở bước này, hãy chọn 2 public subnet của bạn.

![Choose subnet association](edit_subnet.png)

5. **Cấu hình route table cho private subnets**

Bước này tương tự như cấu hình public route table. Thay vì chọn public subnets, bây giờ hãy chọn private subnets.

6. **Tạo S3 Gateway VPC Endpoint**

Đi tới mục `Endpoints` ở thanh bên trái để tạo S3 endpoint.

```txt
- Name tag: <enter-your-name>
- Type: AWS Service
```

Trong phần `Services`, tìm `S3` và chọn service name có type là `Gateway`.

![Search for S3 service](service_s3endpoint.jpg)

Trong phần `Network Settings` và `Route Table`, chọn VPC bạn đã tạo và chọn **private subnets**. Giữ nguyên phần `Policy` ở mặc định. Để kiểm tra kết quả, quay lại private route table. Bạn sẽ thấy một route mới có dạng `pl-xxxxx`. Đó chính là endpoint vừa được tạo.

![Verify step](verify_endpoint.png)

{{% notice note %}}
Chúng ta không tạo **NAT Gateway** vì **EC2 instance** sẽ được đặt trong public subnet, còn **RDS** sẽ giao tiếp thông qua **S3 Gateway Endpoint** từ private subnet.
{{% /notice %}}
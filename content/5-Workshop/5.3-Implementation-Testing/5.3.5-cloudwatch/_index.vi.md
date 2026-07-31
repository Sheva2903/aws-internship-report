---
title: "Giám sát với CloudWatch"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5.3.5. </b> "
---

1. **Cài đặt CloudWatch trên EC2**

SSH vào EC2 của bạn và cài CloudWatch Agent bằng lệnh: `sudo dnf install -y amazon-cloudwatch-agent`

Tiếp theo, thêm quyền cho CloudWatch Agent trong EC2 role để agent có thể gửi log hoặc metric.

![Add permission](image.png)

2. **Tạo cấu hình cho CloudWatch Agent trên EC2**

```bash
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/home/ec2-user/app.log",
            "log_group_name": "/diyshop/app",
            "log_stream_name": "{instance_id}"
          },
          {
            "file_path": "/var/log/tomcat/access.log",
            "log_group_name": "/diyshop/access",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}

```

Sau đó, chạy cấu hình vừa tạo, ví dụ:

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/etc/config.json
```

Tiếp theo, kiểm tra trên CloudWatch console:

![CloudWach Log](image-1.png)

3. **Tạo metric filters**

Truy cập CloudWatch console, chọn `Log Management` và chọn log group cần cấu hình, sau đó chuyển sang tab `Metric filters`. Khi tạo metric filter, chúng ta cần điền filter pattern, ví dụ `?ERROR ?WARN ?FAILED`. Điều này có nghĩa là tôi muốn lọc các response có chứa từ khóa error, warn hoặc failed.

Ở bước thứ hai, cấu hình như sau:

![Second Step](image-2.png)

Lưu ý với `namespace`: các metric filter khác cần được đặt trong cùng namespace.

Sau đó, tôi làm tương tự cho các metric filter `http-4xx-count` và `http-5xx-count`, dùng để lọc các response có status code theo pattern 4xx và 5xx.

4. **Tạo Alarm và SNS**

Đầu tiên, tôi tạo SNS topic giống như các bước trước đó, bằng cách truy cập SNS console, chọn Topics và tạo topic.

Thứ hai, tôi tạo subscription. Ở phần `Protocol`, chọn `Email` và nhập email sẽ nhận cảnh báo.

![Subcription](image-3.png)

Thứ ba, tôi tạo alarm cho từng metric với statistic là `SUM` để dễ theo dõi hiệu quả. Threshold tôi đặt là `Greater than 0` trong `1 minute`. Ở phần `Configure Actions`, cấu hình như sau để nhận thông báo khi alarm được kích hoạt:

![Alarms Notification](image-4.png)

Bây giờ, nếu tôi truy cập bất kỳ endpoint nào hoặc cố tình trigger alarm, hệ thống sẽ tạo logs, gửi alarm về email của tôi và hiển thị graph thống kê:

![alt text](image-5.png)

![alt text](image-6.png)

![alt text](image-7.png)
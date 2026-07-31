---
title: "Tạo EC2 và triển khai backend"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

1. **Tạo EC2 instance**

Truy cập EC2 console, chọn `Instances` và chọn `Launch instance`. Nhập tên instance của bạn vào phần `Name`.

Tiếp theo, chọn hệ điều hành cho instance. Trong lab này, tôi sẽ chọn:

```txt
AMI: Amazon Linux 2023
Instance type: t3.micro
```

![OS Model](os_instance.png)

Tiếp theo, ở phần Key Pair, tạo key với định dạng `.pem` và lưu lại trên máy để sử dụng sau.

![Key Creation](key.png)

Tiếp theo, chuyển tới phần Network Settings. Chọn VPC của bạn, public subnet, chọn `enable` cho auto-assign public IP. IP này sẽ thay đổi khi instance reboot. Ở phần security group, chọn security group bạn đã tạo cho EC2.

![Network Settings](network_ec2.png)

Tiếp theo, chuyển tới phần Storage. Trong advanced setting của phần này, chỉ cần bật `Encrypted` cho EBS Encryption.

![Storage](storage.png)

2. **Deploy backend lên EC2 và kết nối RDS**

Ở bước này, cách làm sẽ phụ thuộc vào công nghệ backend của project. Trong trường hợp của tôi, backend sử dụng Spring Boot, nên tôi cần cài JDK trên EC2 và bắt đầu kết nối tới RDS.

Đầu tiên, tôi sẽ SSH vào EC2 bằng lệnh: `ssh -i <your-pem-file-location> <user>@<public-ip>`

Nếu thành công, bạn sẽ thấy kết quả tương tự như sau:

```bash
┌──(mq㉿mqngyn)-[~]
└─$ ssh -i ~/diy-app.pem ec2-user@52.65.133.54
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html

A newer release of "Amazon Linux" is available.
  Version 2023.12.20260724:
  Version 2023.12.20260727:
Run "/usr/bin/dnf check-release-update" for full release and version update info
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'
Last login: Thu Jul 30 09:57:45 2026 from 116.100.247.223
[ec2-user@ip-10-0-13-67 ~]$
```

Bây giờ, tôi sẽ cài JDK vào EC2 instance bằng lệnh `sudo dnf install -y java-17-amazon-corretto`. Sau đó, tôi build file `.jar` và copy lên EC2:

```bash
./mvnw clean package -DskipTests
scp -i <your-pem-key> <jar-location-after-build> ec2-user@<public-ip>:/home/ec2-user/app.jar
```

Trong project này, tôi cấu hình một `systemd` service cho ứng dụng Spring Boot trên EC2 để ứng dụng chạy như một background process ổn định. Service này sẽ tự động restart khi lỗi `(Restart=on-failure)` và tự chạy lại sau khi EC2 reboot, thay vì phụ thuộc vào một phiên `java -jar app.jar` thủ công vốn sẽ dừng ngay khi SSH connection bị đóng.

```bash
[ec2-user@ip-10-0-13-67 ~]$ cat /etc/systemd/system/diyshop.service
[Unit]
Description=DIY Shop Spring Boot App
After=network.target

[Service]
Type=simple
User=ec2-user
EnvironmentFile=/etc/diyshop/env
ExecStart=/usr/bin/java -jar /home/ec2-user/app.jar
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Lưu ý rằng trong file `env`, tôi đã định nghĩa tất cả các biến cần thiết cho ứng dụng và kết nối tới RDS, ví dụ như `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`.

Sau đó, bạn có thể kiểm tra bằng lệnh `sudo systemctl status <your-app-name>`. Nếu status là active, bạn đã thành công:

```bash
[ec2-user@ip-10-0-13-67 ~]$ sudo systemctl status diyshop
● diyshop.service - DIY Shop Spring Boot App
     Loaded: loaded (/etc/systemd/system/diyshop.service; enabled; preset: disabled)
     Active: active (running) since Thu 2026-07-30 17:20:19 UTC; 8s ago
   Main PID: 16654 (java)
      Tasks: 18 (limit: 1059)
     Memory: 170.1M
        CPU: 15.473s
     CGroup: /system.slice/diyshop.service
             └─16654 /usr/bin/java -jar /home/ec2-user/app.jar
```

Tiếp theo, kiểm tra kết nối tới RDS. Đầu tiên, truy cập PostgreSQL RDS bằng lệnh: `psql -h diyshop-db-instance.cdcmqck2qzuj.ap-southeast-2.rds.amazonaws.com -p 5432 -U <master_username> -d diyshop`. Sau đó chạy `SELECT version, description, success FROM flyway_schema_history;` để kiểm tra Flyway:

```bash
diyshop=> SELECT version, description, success FROM flyway_schema_history;
 version |          description          | success
---------+-------------------------------+---------
 1       | create categories             | t
 2       | create products               | t
 3       | create product images         | t
 5       | create orders                 | t
 6       | add product image storage     | t
 4       | seed initial catalog          | t
 7       | seed additional mock data     | t
 8       | add order cancellation reason | t
 9       | create shop settings          | t
(9 rows)
```

Nếu `success = t`, migration của bạn đã chạy thành công.
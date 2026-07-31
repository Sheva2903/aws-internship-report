---
title: "Cấu hình Security Group"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

1. **Tạo security group cho EC2**

Trong VPC console, nhấn `Security Group` để tạo security group.

```txt
Security group name: <enter-your-group-name>
Description: <enter-your-description>
VPC: <choose-your-vpc>
```

Tiếp theo, cấu hình inbound rules và giữ outbound rules ở mặc định:

```txt
Type: Custom TCP
Port: <enter-your-port>
Source: 0.0.0.0/0 # trong lab này không có ALB nên tạm thời để open

Type: SSH
Port: 22
Source: 0.0.0.0/0
```

![Inbound Rules](inbound.jpg)

![Outbound Rules](outbound.jpg)

2. **Tạo security group cho RDS**

Bước này tương tự như hướng dẫn tạo security group cho EC2. Trong inbound rules, ở tùy chọn `Source`, chọn `Custom` và chọn security group của EC2.

![Inbound rules for RDS](inbound_rds.png)

Trong outbound rules, bạn sẽ cấu hình như sau:

```txt
Type: HTTPS
Port: 443
Destination: Custom - <search-for-prefix-list>
```

![Outbound rules for RDS](outbound_rds.png)
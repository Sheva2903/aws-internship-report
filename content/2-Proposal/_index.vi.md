---
title: "Đề xuất dự án"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

{{% notice warning %}}
⚠️ **Lưu ý:** Một số từ ngữ sẽ được giữ nguyên tiếng Anh trong bản dịch để đảm bảo đúng ngữ nghĩa theo ngữ cảnh.
{{% /notice %}}

# DIY SHOP

## E-commerce management leverages AWS services

### 1. Tóm tắt điều hành

DIY Shop là một website được xây dựng cho hoạt động kinh doanh đồ thủ công và tranh vẽ tay, hỗ trợ chủ shop (người bán duy nhất) quản lý và vận hành cửa hàng trực tuyến. Ứng dụng cho phép khách hàng duyệt danh mục sản phẩm và đặt hàng qua website với hai phương thức thanh toán: (1) Thanh toán khi nhận hàng (COD) và (2) Chuyển khoản ngân hàng (VietQR). Sau khi đặt hàng thành công, hệ thống cho phép khách hàng theo dõi trạng thái đơn hàng bằng mã đơn nhận được khi thanh toán. Về phía vận hành, một dashboard hỗ trợ chủ shop quản lý danh mục sản phẩm, kiểm soát tồn kho và xử lý trạng thái đơn hàng, với trạng thái thanh toán được theo dõi độc lập với trạng thái giao hàng. Website tận dụng các dịch vụ AWS để hiện thực hóa việc tự động hóa vận hành và quản lý.

### 2. Tuyên bố vấn đề

### Vấn đề là gì?

Người bán đồ thủ công và tranh vẽ hiện chưa có một kênh bán hàng trực tuyến chuyên biệt để trưng bày sản phẩm, nhận đơn hàng và theo dõi tồn kho. Điều này khiến quy trình dễ phát sinh sai sót khi số lượng đơn hàng tăng lên, do không có nguồn dữ liệu chung giữa người bán và khách hàng. Là một người bán đơn lẻ, không có chuyên môn kỹ thuật - hoạt động mà không có đội ngũ kỹ thuật hay vận hành riêng - chủ shop sẽ phải xử lý những công việc vượt quá chuyên môn của mình, chẳng hạn như bảo mật dữ liệu cá nhân của khách hàng (họ tên, số điện thoại và địa chỉ thu thập khi thanh toán) và xử lý sự cố máy chủ. Điều này có thể dẫn đến việc khắc phục tốn nhiều thời gian, thậm chí có nguy cơ lộ thông tin khách hàng.

### Giải pháp

Hệ thống được triển khai trên **Amazon EC2**, chạy dưới dạng dịch vụ `systemd` được quản lý, kết nối tới một **instance Amazon RDS PostgreSQL** đặt trong private subnet (với kết nối bị giới hạn theo nguyên tắc least-privilege); hình ảnh sản phẩm được lưu trên **Amazon S3** thông qua một **IAM role**. **Amazon CloudWatch** thu thập log ứng dụng và log truy cập, từ đó suy ra các metric lỗi thông qua metric filter, rồi kích hoạt cảnh báo qua email khi tỷ lệ lỗi vượt ngưỡng. **GitHub Actions** tự động hóa việc build, test và deploy mỗi khi có thay đổi code.

### 3. Kiến trúc giải pháp

DIY Shop được triển khai trong một VPC riêng, trải rộng trên 2 Availability Zone (AZ) để đảm bảo tính sẵn sàng cao. Yêu cầu của người dùng đi qua ba lớp trước khi đến được ứng dụng, từ Route 53 tới CloudFront và WAF, rồi đến Application Load Balancer. Ứng dụng chạy trên các EC2 instance được quản lý bởi Auto Scaling Group, phân bổ trên 2 AZ trong các public subnet. Phần dữ liệu bao gồm RDS PostgreSQL ở chế độ Multi-AZ và S3 để lưu trữ hình ảnh sản phẩm. Toàn bộ vòng đời vận hành được hỗ trợ bởi CloudWatch (giám sát), SNS (cảnh báo), Secrets Manager (quản lý thông tin xác thực), IAM (kiểm soát truy cập) và GitHub Actions (tự động hóa CI/CD).

![Kiến trúc DIY Shop](final_arch.png)

### Các dịch vụ AWS sử dụng

- **Amazon VPC**: Cung cấp mạng biệt lập với 2 public subnet và 2 private subnet trên 2 AZ
- **Amazon EC2**: Lưu trữ backend Spring Boot, chạy dưới dạng dịch vụ `systemd` được quản lý trong public subnet
- **Amazon RDS**: Cơ sở dữ liệu quan hệ được quản lý, lưu toàn bộ dữ liệu giao dịch cốt lõi
- **Amazon S3**: Lưu trữ hình ảnh sản phẩm; cơ sở dữ liệu chỉ lưu tham chiếu
- **Route 53**: phân giải domain trỏ về CloudFront
- **CloudFront**: CDN, cache tài nguyên tĩnh tại các edge location
- **AWS IAM**: Áp dụng nguyên tắc least privilege
- **Amazon CloudWatch**: Tập trung log ứng dụng và log truy cập vào log group, suy ra metric đếm lỗi từ các log đó thông qua metric filter, và đánh giá alarm dựa trên các metric này
- **Amazon SNS**: Gửi thông báo email mỗi khi CloudWatch Alarm chuyển sang trạng thái `ALARM`
- **Secrets Manager**: lưu trữ và tự động xoay vòng thông tin xác thực của DB và người bán

### Thiết kế thành phần

- **Edge và phân phối lưu lượng**: Route 53 phân giải domain tới một alias record trỏ về CloudFront (cache tài nguyên tĩnh, gắn WAF). Mọi request đều được WAF kiểm tra trước khi chuyển tới ALB (cân bằng tải, chấm dứt TLS).
- **Tầng ứng dụng**: Auto Scaling Group quản lý các EC2 instance (2 AZ), mỗi instance chạy một file `jar` Spring Boot duy nhất với frontend React được đóng gói cùng origin, nên không cần cấu hình CORS
- **Tầng dữ liệu**: RDS PostgreSQL Multi-AZ, sử dụng S3 để lưu hình ảnh sản phẩm thông qua IAM Role và presigned URL
- **Bảo mật và quản lý thông tin xác thực**: Secrets Manager lưu trữ thông tin xác thực DB; IAM áp dụng quyền truy cập least-privilege cho EC2
- **Khả năng quan sát (Observability)**: CloudWatch Agent thu thập log và metric. Alarm chịu trách nhiệm kiểm tra điều kiện dựa trên các metric đó
- **CI/CD**: GitHub Actions tự động build, test và deploy mỗi khi có push

### 4. Triển khai kỹ thuật

**Các giai đoạn triển khai**
Dự án được thực hiện qua 2 giai đoạn liên tiếp: (1) Nền tảng cốt lõi (Core Foundation), (2) Bảo mật và độ tin cậy (Security and Reliability).

Về nền tảng cốt lõi (2 tuần):

- Khởi tạo VPC, Security Group, RDS PostgreSQL và EC2; migrate schema thông qua Flyway lên instance RDS thật
- Thiết lập một S3 bucket với IAM Role least-privilege để lưu hình ảnh sản phẩm, xác thực bằng một lần upload thử thực tế
- Ổn định ứng dụng thông qua dịch vụ `systemd`, kết nối CloudWatch Agent, Metric Filter, Alarm và SNS. Sau đó, xây dựng pipeline CI/CD GitHub Actions để build và deploy tự động, và gắn một Elastic IP

Về bảo mật và độ tin cậy (1 tuần):

- **Secrets Manager**: chuyển toàn bộ thông tin xác thực từ file văn bản thuần trên EC2 sang secrets có audit trail và tự động xoay vòng
- **Multi-AZ RDS**: bật Multi-AZ cho instance hiện có, loại bỏ điểm lỗi đơn (single point of failure) ở tầng dữ liệu
- **Application Load Balancer**: thêm một lớp cân bằng tải và chấm dứt TLS, làm nền tảng cho Auto Scaling Group
- **AWS WAF**: gắn một Web ACL để lọc các cuộc tấn công ở tầng ứng dụng và giới hạn tốc độ (rate-limit) cho endpoint đăng nhập

**Yêu cầu kỹ thuật**

- **Backend**: Java 17, Spring Boot 4.0.6, Spring Data JPA, Flyway (quản lý phiên bản schema), Spring Security (đăng nhập bằng form + CSRF cho cổng quản trị người bán)
- **Frontend**: React + Vite + TypeScript + Tailwind CSS, được đóng gói trực tiếp vào thư mục static/ của Spring Boot (cùng origin, không cần cấu hình CORS)
- **Cơ sở dữ liệu**: PostgreSQL 18 (RDS), schema được quản lý hoàn toàn qua các migration Flyway, đi kèm phiên bản cùng với source code
- **Hạ tầng**: Toàn bộ tài nguyên AWS được khởi tạo qua Console (không dùng IaC, do giới hạn thời gian), mỗi bước cấu hình được ghi lại để tái sử dụng trong lab Workshop
- **CI/CD**: GitHub Actions, build và test bằng một PostgreSQL service container mô phỏng môi trường RDS thật trong CI

### 5. Lộ trình & Mốc triển khai

**Lộ trình dự án**

- Thực tập (Tháng 1-2): 2 tháng.
  - Tháng 1: Học AWS và làm lab
  - Tháng 2: Thiết kế kiến trúc, triển khai, kiểm thử và ra mắt

### 6. Ước tính ngân sách

Bạn có thể xem ước tính ngân sách trên [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=aa60066de7ae4feb705c3dd690f39a0977a659b9).

### Chi phí hạ tầng

- Amazon Route 53: 0,50 USD/tháng (Hosted Zone, các record bổ sung trong hosted zone)
- Amazon WAF: 7,00 USD/tháng (1 Web ACL sử dụng mỗi tháng, 2 rule bổ sung)
- Amazon CloudFront: 0,00 USD/tháng (gói miễn phí)
- Amazon S3: 0,20 USD/tháng (lưu trữ Standard - 8GB/tháng)
- Amazon EC2: 4,16 USD/tháng (Linux, t3.micro, bật monitoring)
- Amazon RDS: 100,36 USD/tháng (100GB, db.t4g.micro, chỉ on-demand)

Tổng: 121,22 USD/tháng và 1454,64 USD/12 tháng

### 7. Đánh giá rủi ro

#### Ma trận rủi ro

- Điểm lỗi đơn (Single Point of Failure) ở EC2/RDS: Ảnh hưởng cao, xác suất trung bình.
- Rò rỉ thông tin xác thực / lộ dữ liệu: Ảnh hưởng cao, xác suất thấp.
- Vượt ngân sách (RDS Multi-AZ, EC2 chạy liên tục): Ảnh hưởng trung bình, xác suất trung bình.
- Triển khai lỗi: Ảnh hưởng trung bình, xác suất thấp.

#### Chiến lược giảm thiểu

- Tính sẵn sàng: Auto Scaling Group trên 2 AZ phía sau ALB, RDS ở chế độ Multi-AZ.
- Bảo mật: Secrets Manager để xoay vòng thông tin xác thực, IAM least-privilege, WAF Web ACL.
- Chi phí: CloudWatch billing alarm, chọn kích thước instance phù hợp (t3.micro/db.t4g.micro).
- Triển khai: CI/CD GitHub Actions chạy build và test trước mỗi lần deploy.

#### Kế hoạch dự phòng

- Rollback về bản build ổn định trước đó thông qua GitHub Actions nếu deploy thất bại.
- Khôi phục RDS từ snapshot tự động nếu xảy ra lỗi dữ liệu.

### 8. Kết quả kỳ vọng

- Đảm bảo bảo mật cơ bản thông qua IAM và WAF
- Đảm bảo migration schema qua Flyway lên RDS diễn ra ổn định
- Log và metric được hiển thị qua dashboard CloudWatch, kèm thông báo email được gửi mỗi khi có alarm được kích hoạt

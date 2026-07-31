---
title: "Đề xuất dự án"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

Trong phần này, tôi tóm tắt đề xuất workshop/project về việc triển khai và bảo mật ứng dụng DIY Shop trên AWS.

# Nền tảng thương mại điện tử DIY Shop

## Giải pháp AWS bảo mật cho website bán đồ thủ công trực tuyến

### 1. Tóm tắt điều hành

DIY Shop là một website được xây dựng cho mô hình kinh doanh đồ thủ công và tranh vẽ thủ công, hỗ trợ chủ shop đơn lẻ trong việc quản lý và vận hành cửa hàng online. Ứng dụng cho phép khách hàng xem danh mục sản phẩm và đặt hàng trực tiếp trên website thông qua hai phương thức thanh toán: (1) Thanh toán khi nhận hàng (COD) và (2) Chuyển khoản ngân hàng bằng VietQR. Sau khi đặt hàng thành công, hệ thống cho phép khách hàng theo dõi trạng thái đơn hàng bằng mã đơn hàng nhận được sau khi checkout.

Ở phía vận hành, dashboard hỗ trợ chủ shop quản lý danh mục sản phẩm, kiểm soát tồn kho và xử lý trạng thái đơn hàng. Trạng thái thanh toán được theo dõi độc lập với trạng thái giao hàng. Website tận dụng các dịch vụ AWS để hỗ trợ tự động hóa quá trình triển khai, vận hành, giám sát và quản lý hệ thống.

### 2. Tuyên bố vấn đề

### Vấn đề hiện tại

Người bán đồ thủ công và tranh vẽ hiện chưa có một kênh bán hàng online riêng để trưng bày sản phẩm, nhận đơn hàng và theo dõi tồn kho. Khi số lượng đơn hàng tăng lên, quy trình xử lý thủ công dễ phát sinh lỗi vì không có một nguồn dữ liệu thống nhất giữa người bán và khách hàng.

Với mô hình một người bán không chuyên về kỹ thuật và không có đội ngũ kỹ thuật hoặc vận hành riêng, chủ shop sẽ phải tự xử lý nhiều vấn đề nằm ngoài chuyên môn, chẳng hạn như bảo vệ thông tin cá nhân của khách hàng gồm tên, số điện thoại, địa chỉ nhận hàng, hoặc xử lý sự cố máy chủ. Điều này có thể dẫn đến việc mất nhiều thời gian khôi phục hệ thống, đồng thời làm tăng rủi ro lộ thông tin khách hàng.

### Giải pháp

Hệ thống được triển khai trên **Amazon EC2**, chạy dưới dạng một `systemd` service để đảm bảo ứng dụng hoạt động ổn định như một background process. Ứng dụng kết nối tới **Amazon RDS PostgreSQL instance** được đặt trong private subnet, với kết nối được giới hạn theo nguyên tắc least privilege. Ảnh sản phẩm được lưu trữ trên **Amazon S3** thông qua **IAM Role**, thay vì sử dụng access key tĩnh trong ứng dụng.

**Amazon CloudWatch** được sử dụng để thu thập application logs và access logs. Từ các logs này, hệ thống tạo error metrics thông qua metric filters, sau đó kích hoạt email alerts khi số lượng lỗi vượt quá ngưỡng cấu hình. **GitHub Actions** được sử dụng để tự động hóa quá trình build, test và deploy mỗi khi có thay đổi trong source code.

### 3. Kiến trúc giải pháp

DIY Shop được triển khai bên trong một VPC riêng, trải rộng trên 2 Availability Zones (AZs) để tăng tính sẵn sàng cao. User requests đi qua các lớp edge và phân phối traffic trước khi tới application layer, bao gồm Route 53, CloudFront, WAF và Application Load Balancer. Ứng dụng chạy trên các EC2 instances được quản lý bởi Auto Scaling Group, phân tán trên 2 AZs.

Tầng dữ liệu bao gồm RDS PostgreSQL ở chế độ Multi-AZ để lưu trữ dữ liệu giao dịch chính, và S3 để lưu ảnh sản phẩm. Toàn bộ vòng đời vận hành được hỗ trợ bởi CloudWatch cho monitoring, SNS cho alerting, Secrets Manager cho credential management, IAM cho access control, và GitHub Actions cho CI/CD automation.

![DIY Shop Architecture](architecture.jpg)

### AWS Services được sử dụng

- **Amazon VPC**: Cung cấp môi trường mạng tách biệt với 2 public subnets và 2 private subnets trải rộng trên 2 AZs.
- **Amazon EC2**: Chạy Spring Boot backend dưới dạng một `systemd` service trong public subnet.
- **Amazon RDS**: Cung cấp managed relational database để lưu trữ dữ liệu giao dịch chính của hệ thống.
- **Amazon S3**: Lưu trữ ảnh sản phẩm; database chỉ lưu reference tới ảnh.
- **Route 53**: Phân giải domain tới CloudFront.
- **CloudFront**: CDN dùng để cache static assets tại edge locations.
- **AWS IAM**: Áp dụng nguyên tắc least privilege cho quyền truy cập giữa các dịch vụ.
- **Amazon CloudWatch**: Tập trung application logs và access logs vào log groups, tạo error-count metrics thông qua metric filters và đánh giá alarms dựa trên các metrics đó.
- **Amazon SNS**: Gửi email notification khi CloudWatch Alarm chuyển sang trạng thái `ALARM`.
- **Secrets Manager**: Lưu trữ và hỗ trợ rotation cho database credentials và seller credentials.

### Thiết kế thành phần

- **Edge và Traffic Distribution**: Route 53 phân giải domain tới alias record trỏ về CloudFront. CloudFront cache static assets và có thể gắn WAF để kiểm tra request trước khi chuyển tiếp tới ALB. ALB chịu trách nhiệm load balancing và TLS termination.
- **Application Tier**: Auto Scaling Group quản lý các EC2 instances trên 2 AZs. Mỗi instance chạy một Spring Boot `jar`, trong đó React frontend được bundle chung vào cùng origin, nên không cần cấu hình CORS riêng.
- **Data Tier**: RDS PostgreSQL Multi-AZ lưu trữ dữ liệu giao dịch chính. S3 được dùng để lưu ảnh sản phẩm thông qua IAM Role và presigned URLs.
- **Security và Credential Management**: Secrets Manager lưu database credentials; IAM áp dụng least-privilege access cho EC2.
- **Observability**: CloudWatch Agent thu thập logs và metrics. CloudWatch Alarm kiểm tra điều kiện dựa trên metric filters và kích hoạt SNS notification khi có lỗi vượt ngưỡng.
- **CI/CD**: GitHub Actions tự động build, test và deploy ứng dụng mỗi khi có thay đổi được push lên repository.

### 4. Triển khai kỹ thuật

**Các giai đoạn triển khai**

Dự án được triển khai qua 2 giai đoạn liên tiếp: (1) Core Foundation, (2) Security and Reliability.

Về giai đoạn Core Foundation trong 2 tuần:

- Provision VPC, Security Groups, RDS PostgreSQL và EC2; migrate database schema lên RDS thật thông qua Flyway.
- Thiết lập S3 bucket với IAM Role theo nguyên tắc least privilege để lưu trữ ảnh sản phẩm, sau đó kiểm tra bằng một upload test thực tế.
- Ổn định ứng dụng bằng `systemd` service, cấu hình CloudWatch Agent, Metric Filters, Alarms và SNS. Sau đó xây dựng GitHub Actions CI/CD pipeline để tự động build và deploy, đồng thời gắn Elastic IP cho EC2.

Về giai đoạn Security and Reliability trong 1 tuần:

- **Secrets Manager**: Di chuyển toàn bộ credentials từ plaintext files trên EC2 sang secrets có audit trail và hỗ trợ automatic rotation.
- **Multi-AZ RDS**: Bật Multi-AZ cho database instance hiện có để loại bỏ single point of failure ở data tier.
- **Application Load Balancer**: Bổ sung lớp load balancing và TLS termination, tạo nền tảng để mở rộng bằng Auto Scaling Group.
- **AWS WAF**: Gắn Web ACL để lọc các application-layer attacks và rate-limit login endpoint.

**Yêu cầu kỹ thuật**

- **Backend**: Java 17, Spring Boot 4.0.6, Spring Data JPA, Flyway để quản lý database schema versioning, Spring Security với form login và CSRF cho seller portal.
- **Frontend**: React + Vite + TypeScript + Tailwind CSS, được bundle trực tiếp vào thư mục `static/` của Spring Boot để chạy cùng origin và không cần cấu hình CORS.
- **Database**: PostgreSQL 18 trên Amazon RDS, schema được quản lý hoàn toàn bằng Flyway migrations và versioned cùng source code.
- **Infrastructure**: Tất cả AWS resources được provision thông qua AWS Console. Do giới hạn thời gian, project không sử dụng Infrastructure as Code, nhưng mỗi bước cấu hình đều được ghi lại để tái sử dụng trong workshop lab.
- **CI/CD**: GitHub Actions thực hiện build và test, sử dụng PostgreSQL service container để mô phỏng môi trường RDS thật trong CI.

### 5. Lộ trình & Mốc triển khai

**Lộ trình dự án**

- Thời gian thực tập: 2 tháng.
  - Tháng 1: Học AWS và thực hiện các labs.
  - Tháng 2: Thiết kế kiến trúc, triển khai, kiểm thử và đưa hệ thống vào hoạt động.

### 6. Ước tính ngân sách

Có thể xem chi phí ước tính trên [AWS Pricing Calculator](https://calculator.aws/#/estimate?id=621f38b12a1ef026842ba2ddfe46ff936ed4ab01).  
Hoặc tải [tệp ước tính ngân sách](../attachments/budget_estimation.pdf).

### Chi phí hạ tầng

- AWS Services:
  - Amazon EC2: Chi phí cho instance chạy backend Spring Boot.
  - Amazon RDS PostgreSQL: Chi phí cho managed database instance và storage.
  - Amazon S3: Chi phí lưu trữ ảnh sản phẩm và requests.
  - Amazon CloudWatch: Chi phí log ingestion, log storage, metrics và alarms.
  - Amazon SNS: Chi phí gửi email notification cho alarms.
  - AWS Secrets Manager: Chi phí lưu trữ và quản lý secrets.
  - Elastic IP / Data Transfer: Chi phí liên quan đến IP tĩnh và traffic ra Internet nếu vượt free-tier hoặc quota miễn phí.

Tổng chi phí phụ thuộc vào loại instance, dung lượng database, lượng log, số lượng ảnh sản phẩm và traffic thực tế của website.

### 7. Đánh giá rủi ro

#### Ma trận rủi ro

- Server downtime: Ảnh hưởng cao, xác suất trung bình.
- Database unavailable: Ảnh hưởng cao, xác suất thấp đến trung bình.
- Lộ credentials hoặc cấu hình sai quyền truy cập: Ảnh hưởng cao, xác suất trung bình.
- S3 object bị public ngoài ý muốn: Ảnh hưởng cao, xác suất thấp.
- Vượt ngân sách do logs, storage hoặc data transfer tăng nhanh: Ảnh hưởng trung bình, xác suất trung bình.

#### Chiến lược giảm thiểu

- Server: Chạy ứng dụng bằng `systemd` service với `Restart=on-failure`, sử dụng CloudWatch để phát hiện lỗi sớm.
- Database: Đặt RDS trong private subnet, chỉ cho phép kết nối từ Security Group của backend, và bật Multi-AZ trong giai đoạn nâng cao.
- Credentials: Di chuyển credentials sang Secrets Manager, hạn chế lưu plaintext credentials trên EC2.
- S3: Bật Block Public Access, sử dụng IAM Role theo nguyên tắc least privilege và chỉ cấp quyền trên folder cần thiết.
- Chi phí: Tạo CloudWatch Alarms và AWS Budget alerts để theo dõi log volume, storage và các dịch vụ phát sinh chi phí.

#### Kế hoạch dự phòng

- Nếu deployment mới lỗi, rollback về phiên bản `.jar` ổn định trước đó.
- Nếu database migration lỗi, kiểm tra Flyway history và rollback bằng backup hoặc snapshot.
- Nếu EC2 gặp sự cố, tạo instance mới từ cấu hình đã được document và deploy lại ứng dụng.
- Nếu S3 upload lỗi, kiểm tra IAM Role, bucket policy, prefix permission và CloudWatch logs của ứng dụng.
- Nếu AWS service gặp sự cố trong một AZ, sử dụng Multi-AZ RDS và Auto Scaling Group để giảm ảnh hưởng tới hệ thống.

### 8. Kết quả kỳ vọng

#### Cải tiến kỹ thuật

Ứng dụng DIY Shop được triển khai trên AWS thay vì chỉ chạy local, có database managed bằng RDS, lưu trữ ảnh sản phẩm trên S3, backend chạy ổn định trên EC2 và có khả năng giám sát thông qua CloudWatch. Quy trình vận hành được cải thiện nhờ CI/CD, logs tập trung, alarms và email notifications.

#### Giá trị dài hạn

Hệ thống tạo nền tảng để chủ shop vận hành kênh bán hàng online riêng, quản lý sản phẩm, tồn kho và đơn hàng tập trung. Kiến trúc có thể tiếp tục mở rộng với Multi-AZ, Load Balancer, Auto Scaling, WAF, Secrets Manager và CI/CD automation để phù hợp với nhu cầu production trong tương lai.
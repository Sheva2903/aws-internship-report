---
title: "Bài viết 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# SHARED RESPONSIBILITY MODEL TRÊN AWS: BẢO MẬT CLOUD KHÔNG DỪNG Ở NHÀ CUNG CẤP

Cloud computing giúp giảm gánh nặng vận hành hạ tầng vật lý. Thay vì tự quản lý data center, máy chủ, thiết bị mạng và hệ thống lưu trữ, người dùng có thể triển khai tài nguyên trực tiếp trên các nền tảng như AWS.

Tuy nhiên, đưa hệ thống lên AWS không có nghĩa là toàn bộ trách nhiệm bảo mật được chuyển hết cho AWS. Cloud chỉ làm thay đổi ranh giới trách nhiệm. Nó không loại bỏ trách nhiệm của người dùng.

Các nghiên cứu về cloud security nhấn mạnh rằng cloud không chỉ là một nền tảng kỹ thuật. Nó còn bao gồm kiến trúc, quy trình, con người, chính sách, dữ liệu, ứng dụng, nhà cung cấp, bên kiểm toán và cả chuỗi cung ứng mở rộng. Vì vậy, bảo mật trong cloud cần được nhìn như một mô hình trách nhiệm chia sẻ, không phải một trách nhiệm được giao khoán hoàn toàn cho nhà cung cấp.

## 1. Shared Responsibility Model là gì?

AWS mô tả mô hình này bằng hai phần: **security of the cloud** và **security in the cloud**.

AWS chịu trách nhiệm bảo vệ hạ tầng vận hành các dịch vụ AWS. Phần này bao gồm data center, máy chủ vật lý, hệ thống lưu trữ, hạ tầng mạng và các lớp nền tảng của dịch vụ managed.

Khách hàng chịu trách nhiệm với cách mình sử dụng AWS. Phần này bao gồm dữ liệu, cấu hình identity, access policy, network rule, mã nguồn ứng dụng, guest operating system trong một số dịch vụ, lựa chọn mã hóa, logging và quy trình bảo mật nội bộ.

Có thể hiểu ngắn gọn như sau:

> AWS bảo vệ nền tảng cloud. Người dùng bảo vệ những gì mình xây dựng và cấu hình trên nền tảng đó.

{{< figure src="/aws-internship-report/images/3-BlogsPosted/3.1-Blog1/aws-shared-responsibility-model.png" title="Shared Responsibility Model on AWS" >}}

## 2. Trách nhiệm thay đổi theo từng dịch vụ

Ranh giới trách nhiệm không giống nhau ở mọi dịch vụ AWS.

Với **Amazon EC2**, người dùng có nhiều quyền kiểm soát hơn, nhưng cũng phải chịu nhiều trách nhiệm hơn. Người dùng cần quản lý guest operating system, cập nhật phần mềm, Security Group, SSH key, bảo mật ứng dụng và dữ liệu lưu trên instance.

Với **Amazon RDS**, AWS quản lý nhiều phần hạ tầng database bên dưới. Tuy vậy, người dùng vẫn phải quản lý database user, network access, encryption setting, parameter group, backup configuration và kết nối từ ứng dụng.

Với **Amazon S3**, AWS vận hành hạ tầng lưu trữ. Người dùng phải cấu hình bucket policy, Block Public Access, encryption, lifecycle rule, versioning và access logging.

Với **AWS Lambda**, AWS quản lý nền tảng chạy function, scaling và phần lớn hạ tầng thực thi. Người dùng vẫn chịu trách nhiệm với function code, dependency, IAM role, secret, environment variable và logic kiểm tra dữ liệu đầu vào.

Điều này tương ứng với cách cloud security thường phân chia theo IaaS, PaaS và SaaS. Dịch vụ càng managed thì provider càng chịu nhiều trách nhiệm hạ tầng hơn. Tuy nhiên, dữ liệu, quyền truy cập, cấu hình và hành vi ứng dụng vẫn luôn là phần quan trọng thuộc trách nhiệm của người dùng.

## 3. Nhiều sự cố cloud bắt đầu từ cấu hình sai

Nhiều sự cố bảo mật trên cloud không bắt đầu từ việc hạ tầng vật lý của cloud provider bị lỗi. Chúng thường bắt đầu từ sai sót cấu hình ở phía người dùng.

Một số ví dụ phổ biến:

* IAM policy cấp nhiều quyền hơn mức cần thiết.
* S3 bucket bị public khi không có lý do rõ ràng.
* Security Group mở port nhạy cảm ra Internet.
* Access key bị commit lên Git repository.
* Secret được lưu trực tiếp trong source code hoặc environment variable dạng plaintext.
* Logging bị tắt, thiếu hoặc không được theo dõi.
* Dữ liệu nhạy cảm không được mã hóa.
* Không có kế hoạch backup và recovery rõ ràng.

Các đặc điểm như resource pooling, virtualization, multi-tenancy, broad network access và on-demand self-service giúp cloud linh hoạt. Nhưng chính các đặc điểm này cũng làm bảo mật phụ thuộc nhiều hơn vào cấu hình rõ ràng, kiểm soát truy cập chặt, giám sát và quản trị.

## 4. IAM nên là lớp được kiểm soát đầu tiên

Trong AWS, IAM quyết định ai được thực hiện hành động nào, trên tài nguyên nào và trong điều kiện nào. Một policy cấp quyền quá rộng có thể biến một lỗi nhỏ thành sự cố nghiêm trọng.

Nguyên tắc thực tế là **least privilege**. Mỗi user, role, service và workload chỉ nên có đúng quyền cần thiết cho nhiệm vụ của nó.

Ví dụ, một Lambda function chỉ cần đọc file trong một S3 prefix cụ thể thì không nên có toàn quyền với toàn bộ S3 trong tài khoản. Policy nên được giới hạn theo đúng bucket, prefix và action cần dùng.

Một thiết kế IAM an toàn cũng cần tính đến MFA, role-based access, short-lived credentials, permission boundary và review quyền định kỳ.

## 5. S3 cần được kiểm soát public access cẩn thận

Amazon S3 được dùng rất rộng rãi để lưu log, backup, static file, deployment artifact, dữ liệu phân tích và nội dung khách hàng. Vì vậy, sai cấu hình S3 có thể gây hậu quả nghiêm trọng.

Một bucket private nên được giữ private theo mặc định. S3 Block Public Access nên được bật nếu không có lý do rõ ràng để public object.

Với mỗi bucket, nên trả lời được các câu hỏi cơ bản:

```text
Bucket này có cần public không?
Ai được đọc object?
Ai được ghi object?
Có bật encryption không?
Có cần versioning không?
Có cần access logging không?
Các phiên bản cũ có cần lifecycle rule không?
```

S3 là dịch vụ lưu trữ có độ bền cao và được AWS quản lý tốt, nhưng AWS không thể tự quyết định object nào của khách hàng là nhạy cảm. Quyết định đó thuộc về người dùng.

## 6. Không có log thì rất khó điều tra

Bảo mật không chỉ là ngăn chặn. Hệ thống cũng phải hỗ trợ điều tra khi có sự cố.

Khi có thay đổi trong tài khoản AWS, đội ngũ vận hành cần trả lời được:

```text
Ai đã thực hiện thay đổi?
Thay đổi xảy ra lúc nào?
Hành động đến từ đâu?
Tài nguyên nào bị ảnh hưởng?
Hành động đó có nằm trong dự kiến không?
```

AWS CloudTrail ghi lại hoạt động tài khoản và các API call trên nhiều dịch vụ AWS. CloudWatch Logs, VPC Flow Logs, S3 access logs, AWS Config, GuardDuty và Security Hub có thể bổ sung thêm khả năng quan sát tùy theo workload.

Logging cần được bật trước khi sự cố xảy ra. Nếu chỉ bắt đầu bật log sau sự cố, nhiều bằng chứng quan trọng có thể đã không tồn tại.

## 7. Checklist thực tế

Một checklist AWS security cơ bản nên gồm:

```text
IAM:
- Bật MFA cho user có quyền cao.
- Tránh dùng root account cho công việc hằng ngày.
- Ưu tiên role thay vì access key dài hạn.
- Review các policy dùng "*" quá rộng.

S3:
- Giữ bucket private nếu không cần public.
- Bật Block Public Access cho bucket private.
- Mã hóa dữ liệu nhạy cảm.
- Dùng versioning hoặc backup cho dữ liệu quan trọng.

Network:
- Không public database trực tiếp ra Internet.
- Đặt internal service trong private subnet.
- Giới hạn inbound rule trong Security Group.
- Review outbound access khi cần.

Logging:
- Bật CloudTrail.
- Lưu log ở nơi được bảo vệ.
- Theo dõi thay đổi IAM, S3 policy và Security Group.
- Tạo cảnh báo cho hành động quản trị rủi ro.

Application:
- Không lưu secret trong source code.
- Kiểm tra input.
- Cập nhật dependency.
- Dùng managed secret storage khi phù hợp.
```

## 8. Kết luận

AWS giúp loại bỏ nhiều gánh nặng vận hành hạ tầng vật lý, nhưng không loại bỏ trách nhiệm bảo mật của khách hàng. Người dùng vẫn kiểm soát dữ liệu, quyền truy cập, cấu hình dịch vụ, mã nguồn ứng dụng, logging và phương án khôi phục.

Shared Responsibility Model là nền tảng để thiết kế hệ thống an toàn trên AWS. Khi hiểu rõ ranh giới này, đội ngũ kỹ thuật sẽ cẩn thận hơn với IAM, S3, network, logging, encryption, secret management và backup.

Có thể xem cloud security như một quan hệ phối hợp:

> AWS bảo vệ nền tảng. Người dùng bảo vệ cách mình sử dụng nền tảng đó.

## Tài liệu tham khảo

- AWS Shared Responsibility Model: https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/shared-responsibility.html
- AWS IAM best practices: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
- Amazon S3 Block Public Access: https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html
- AWS CloudTrail Documentation: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
- Gururaj Ramachandra, Mohsin Iftikhar, Farrukh Aslam Khan, "A Comprehensive Survey on Security in Cloud Computing", Procedia Computer Science, 2017.
- M. Ali, S. U. Khan, A. V. Vasilakos, "Security in cloud computing: Opportunities and challenges", Information Sciences, 2015.

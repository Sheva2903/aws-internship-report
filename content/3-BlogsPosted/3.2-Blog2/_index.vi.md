---
title: "Bài viết 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# MULTI-TENANCY VÀ VIRTUALIZATION TRONG AWS: NỀN TẢNG MỞ RỘNG CLOUD VÀ NƠI BẢO MẬT PHẢI BẮT ĐẦU

Cloud computing hoạt động dựa trên một ý tưởng quan trọng: nhiều khách hàng có thể cùng sử dụng một hạ tầng vật lý lớn, nhưng workload của từng khách hàng vẫn phải được tách biệt. Cách làm này giúp cloud provider phân bổ tài nguyên compute, storage và network hiệu quả hơn, từ đó giúp hệ thống mở rộng nhanh và giảm chi phí vận hành.

Cơ chế này được gọi là **multi-tenancy**. Trong thực tế, nhiều khách hàng, ứng dụng hoặc workload có thể cùng dùng chung một resource pool bên dưới. Để mô hình này an toàn, cloud platform phải đảm bảo các tenant được cô lập bằng virtualization, kiểm soát truy cập, phân đoạn mạng, mã hóa, giám sát và quy trình vận hành phù hợp.

Trong các tài liệu khảo sát về cloud security, multi-tenancy và virtualization thường được xem là hai chủ đề trung tâm. Chúng tạo ra lợi ích lớn cho cloud, nhưng cũng là nguồn gốc của nhiều rủi ro bảo mật nếu lớp isolation và cấu hình phía người dùng không được xử lý đúng.

## 1. Multi-tenancy là gì?

Multi-tenancy nghĩa là nhiều tenant cùng sử dụng một hạ tầng dùng chung. Tenant ở đây có thể là một khách hàng, một tài khoản, một ứng dụng, một workload hoặc một đơn vị nghiệp vụ.

Trong môi trường truyền thống, một tổ chức có thể tự mua và vận hành máy chủ vật lý riêng. Trong cloud, tổ chức thường không cần biết chính xác máy chủ vật lý nào đang chạy workload của mình. Cloud provider sẽ cấp phát tài nguyên từ một resource pool lớn.

Trên AWS, Amazon EC2 instances mặc định chạy trên shared tenancy hardware. Nếu có yêu cầu chặt hơn về compliance, licensing hoặc placement, người dùng có thể chọn Dedicated Instances hoặc Dedicated Hosts.

Điểm quan trọng là: dùng chung hạ tầng không được phép dẫn đến dùng chung quyền truy cập. Workload A không được đọc memory, storage, network traffic hoặc metadata của Workload B.

## 2. Virtualization: Lớp cô lập chính

Virtualization cho phép một máy chủ vật lý chạy nhiều môi trường thực thi độc lập. Mỗi workload nhìn thấy một môi trường riêng, trong khi CPU, memory, storage và network bên dưới vẫn đến từ hạ tầng vật lý dùng chung.

Trên Amazon EC2 hiện đại, AWS Nitro System là một phần quan trọng của mô hình này. AWS mô tả Nitro là tập hợp các thành phần phần cứng và phần mềm giúp EC2 đạt hiệu năng, tính sẵn sàng và bảo mật cao. Nitro cũng giảm bề mặt tấn công bằng cách chuyển nhiều chức năng network, storage và management sang các thành phần chuyên dụng.

Tài liệu AWS nêu rằng Nitro Hypervisor được thiết kế tối giản. Nó không có networking stack, không có general-purpose file system và không hỗ trợ peripheral device driver. Hypervisor càng nhỏ thì càng có ít thành phần có thể bị tấn công.

Với workload serverless và container, AWS cũng sử dụng các công nghệ như Firecracker microVMs. Firecracker được xây dựng để cung cấp lightweight virtualization cho các dịch vụ như AWS Lambda và AWS Fargate.

{{< figure src="images/3-BlogsPosted/3.2-Blog2/aws-multitenancy-virtualization.png" title="Multi-tenancy and Virtualization in AWS" >}}

## 3. Các rủi ro bảo mật chính

Multi-tenancy và virtualization tạo ra nhiều rủi ro cần được kiểm soát cẩn thận.

**VM isolation** là vấn đề đầu tiên. Nhiều máy ảo hoặc môi trường thực thi có thể chạy trên cùng phần cứng. Chúng cần được tách biệt ở các ranh giới CPU, memory, storage, network và metadata.

**VM image và AMI** là một rủi ro thực tế hơn với người dùng AWS. Amazon Machine Image là template dùng để tạo EC2 instance. Nếu một AMI chứa malware, SSH key cũ, secret bị lộ, package có lỗ hổng hoặc user không cần thiết, mọi instance tạo từ AMI đó đều có thể kế thừa cùng điểm yếu.

**Sai cấu hình virtual network** là lỗi phổ biến ở phía người dùng. Trong AWS, phần này liên quan đến VPC, subnet, route table, Security Group, Network ACL và VPC endpoint. Một database đặt trong public subnet hoặc Security Group mở inbound quá rộng vẫn có thể làm lộ tài nguyên, dù hạ tầng bên dưới của AWS được bảo vệ tốt.

**Rủi ro data storage** cũng thay đổi trong cloud. Dữ liệu có thể nằm trên hạ tầng dùng chung, vì vậy logical isolation, encryption, access control, snapshot, backup policy và logging trở nên quan trọng. Người dùng cần biết ai có quyền đọc dữ liệu, ai có thể share snapshot, khóa KMS nào đang bảo vệ dữ liệu và backup được giữ trong bao lâu.

**Rủi ro API và access control** cũng rất đáng chú ý. Hầu hết thao tác trên AWS đều đi qua API. IAM policy yếu, access key tồn tại quá lâu, thiếu MFA hoặc role có quyền quá rộng đều có thể tạo ra rủi ro nghiêm trọng.

## 4. Các kiểm soát thực tế trên AWS

Một môi trường AWS an toàn không phụ thuộc vào một công cụ duy nhất. Nó cần nhiều lớp kiểm soát hoạt động cùng nhau.

Sử dụng **IAM least privilege**. Mỗi role chỉ nên có quyền cần thiết cho tác vụ của nó. Một Lambda function chỉ đọc một S3 prefix không nên có toàn quyền với toàn bộ S3 trong tài khoản.

Thiết kế network theo hướng **private mặc định**. Database, cache, internal API và service quản trị không nên public nếu không có lý do rõ ràng. Public subnet chỉ nên dành cho tài nguyên thật sự cần Internet exposure, ví dụ load balancer.

Sử dụng **image và pipeline đáng tin cậy**. AMI, container image, Lambda layer và dependency nên đến từ nguồn đáng tin cậy. Chúng cần được cập nhật, scan và rebuild định kỳ.

Sử dụng **encryption và kiểm soát khóa**. Dữ liệu nhạy cảm nên được mã hóa khi lưu trữ và khi truyền tải. Với AWS KMS, cần kiểm tra cả IAM policy lẫn key policy vì cả hai đều ảnh hưởng đến quyền sử dụng khóa.

Bật **logging và monitoring**. CloudTrail, CloudWatch Logs, VPC Flow Logs, S3 access logs, GuardDuty, Security Hub và AWS Config giúp phát hiện và điều tra các thay đổi rủi ro hoặc hoạt động đáng ngờ.

Có kế hoạch **backup và recovery**. Snapshot, versioning, lifecycle policy, cross-region replication và kiểm thử restore đều là một phần của bảo mật, vì availability và recoverability cũng là yêu cầu bảo mật.

## 5. Kết luận

Multi-tenancy và virtualization giúp cloud mở rộng tốt và tiết kiệm chi phí. Chúng cho phép nhiều workload cùng sử dụng hạ tầng dùng chung nhưng vẫn được tách biệt ở mức logic.

Tuy nhiên, chính mô hình dùng chung hạ tầng cũng làm cloud security phức tạp hơn. AWS phải bảo vệ hạ tầng bên dưới và các lớp isolation. Người dùng phải bảo vệ workload, identity configuration, network boundary, dữ liệu, mã nguồn ứng dụng và quy trình giám sát của mình.

Có thể nhìn nhận thực tế như sau:

> Cloud security phụ thuộc vào isolation mạnh từ provider và cấu hình đúng từ phía customer.

Khi cả hai phía được xử lý tốt, multi-tenancy không còn là điểm yếu, mà trở thành nền tảng cho hệ thống cloud có khả năng mở rộng và vận hành an toàn.

## Tài liệu tham khảo

- AWS Shared Responsibility Model: https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/shared-responsibility.html
- Amazon EC2 Dedicated Instances: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-instance.html
- Amazon EC2 Dedicated Hosts: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/dedicated-hosts-overview.html
- The Security Design of the AWS Nitro System: https://docs.aws.amazon.com/whitepapers/latest/security-design-of-aws-nitro-system/the-components-of-the-nitro-system.html
- Firecracker: https://firecracker-microvm.github.io/
- M. Ali, S. U. Khan, A. V. Vasilakos, "Security in cloud computing: Opportunities and challenges", Information Sciences, 2015.

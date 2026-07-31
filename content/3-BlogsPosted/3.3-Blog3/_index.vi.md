---
title: "Bài viết 3"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# CRYPTOGRAPHIC DELETION TRONG CLOUD STORAGE: KHI XÓA DỮ LIỆU KHÔNG CHỈ LÀ XÓA OBJECT

{{% notice warning %}}
Xóa một object khỏi cloud storage và làm cho dữ liệu không thể khôi phục không phải lúc nào cũng giống nhau. Trong hệ thống lưu trữ phân tán, dữ liệu có thể tồn tại dưới dạng replica, backup, snapshot hoặc version cũ. FADE tiếp cận vấn đề này bằng cách xóa khóa mật mã thay vì chỉ dựa vào xóa vật lý.
{{% /notice %}}

{{< figure src="images/3-BlogsPosted/3.3-Blog3/fade-cryptographic-deletion-diagram.png" title="FADE: Cryptographic deletion for cloud storage" >}}

## 1. Vấn đề của việc xóa dữ liệu trên cloud

Trong hệ thống file thông thường, thao tác xóa file không phải lúc nào cũng ghi đè ngay nội dung thật trên ổ đĩa. Nhiều trường hợp hệ thống chỉ xóa metadata hoặc con trỏ đến file. Dữ liệu gốc có thể vẫn còn cho đến khi vùng lưu trữ đó bị ghi đè.

Trên cloud storage, vấn đề khó kiểm soát hơn. Một object có thể được nhân bản để tăng độ bền, được backup để phục hồi sau sự cố, được di chuyển giữa nhiều hệ thống lưu trữ, hoặc được giữ lại trong version cũ. Người dùng thường không kiểm soát trực tiếp mọi bản sao vật lý của dữ liệu.

Câu hỏi bảo mật đặt ra là:

```text
Khi cloud báo rằng dữ liệu đã bị xóa, làm sao biết dữ liệu đó thật sự không còn khả năng khôi phục?
```

NIST SP 800-88 mô tả media sanitization là quá trình làm cho việc truy cập dữ liệu mục tiêu trở nên không khả thi ở một mức nỗ lực nhất định. Một hướng xử lý là crypto erase: làm dữ liệu mã hóa trở nên không thể truy cập bằng cách phá hủy hoặc vô hiệu hóa khóa mã hóa.

## 2. FADE trong một câu

FADE là viết tắt của **File Assured Deletion**. Đây là một thiết kế secure overlay cloud storage, bảo vệ file đã outsource bằng mã hóa và làm file trở nên không thể khôi phục khi access policy tương ứng bị thu hồi.

Ý tưởng chính rất đơn giản:

```text
Delete the key, not every cloud replica.
```

Thay vì cố chứng minh rằng mọi bản sao vật lý đã biến mất, FADE làm cho các bản sao mã hóa còn lại trở nên vô dụng bằng cách phá hủy key material cần thiết để giải mã.

## 3. Kiến trúc

FADE tách data storage khỏi key management.

**FADE Client** mã hóa file trước khi upload và giải mã file sau khi download. Các khóa tạm thời được sinh ở phía client và bị xóa sau quá trình upload.

**Cloud Storage**, ví dụ Amazon S3 trong ngữ cảnh cloud object storage, lưu encrypted file và encrypted metadata. Nó không giữ đầy đủ đường dẫn khóa cần thiết để khôi phục plaintext.

**Key Managers** quản lý các khóa gắn với policy. Chúng chỉ cấp key material khi user còn thỏa access policy. Khi policy hết hạn hoặc bị thu hồi, key material liên quan sẽ bị phá hủy.

Vì vậy FADE được gọi là overlay system: nó có thể nằm phía trên cloud storage hiện có mà không yêu cầu storage provider phải thiết kế lại hạ tầng lưu trữ bên dưới.

## 4. Chuỗi khóa

FADE không mã hóa file rồi lưu raw key ngay bên cạnh file. Nó tạo một chuỗi khóa phụ thuộc lẫn nhau.

```text
File F
  encrypted by Data Key K
      ↓
Encrypted File

Data Key K
  encrypted by S
      ↓
Encrypted K

S
  protected by a policy key
      ↓
Encrypted S
```

Đường giải mã sẽ là:

```text
Policy key → S → Data Key K → File F
```

Nếu policy key bị phá hủy, `S` không thể khôi phục. Nếu `S` không thể khôi phục, `K` cũng không thể khôi phục. Nếu `K` không thể khôi phục, file sẽ không đọc được.

## 5. Luồng upload

Một luồng upload đơn giản có thể hiểu như sau:

```text
1. User chọn file F.
2. Client gắn access policy P cho F.
3. Client sinh Data Key K.
4. Client mã hóa F bằng K.
5. Client sinh khóa S.
6. Client mã hóa K bằng S.
7. Client bảo vệ S bằng policy key của P.
8. Encrypted file và metadata được upload lên cloud storage.
9. Các khóa tạm thời ở local bị xóa.
```

Sau khi upload, cloud chỉ lưu ciphertext và encrypted metadata. Cloud không có đầy đủ chuỗi khóa cần thiết để khôi phục file gốc.

## 6. Luồng download

Khi download, client lấy encrypted file và metadata từ cloud storage. Sau đó client liên hệ Key Managers.

Nếu access policy vẫn hợp lệ, Key Managers hỗ trợ khôi phục key material cần thiết để lấy `S`. Client dùng `S` để khôi phục `K`, rồi dùng `K` để giải mã file.

```text
User satisfies policy
    ↓
Key Managers release/decrypt S
    ↓
Client recovers K
    ↓
Client decrypts the file
```

Nếu policy không còn hợp lệ, key material không được cấp. Object mã hóa có thể vẫn tồn tại, nhưng không thể đọc được.

## 7. Assured deletion

Assured deletion xảy ra khi access policy hết hạn hoặc bị thu hồi.

```text
Policy revoked
    ↓
Policy key deleted
    ↓
S unrecoverable
    ↓
K unrecoverable
    ↓
File permanently inaccessible
```

FADE không chứng minh rằng mọi replica vật lý đã bị xóa sạch. Nó cung cấp bảo đảm ở tầng mật mã: mọi bản sao mã hóa còn lại không thể giải mã nếu thiếu key đã bị phá hủy.

Cần phân biệt rõ:

```text
Physical deletion       = xóa hoặc ghi đè dữ liệu lưu trữ
Cryptographic deletion  = làm dữ liệu mã hóa không thể đọc bằng cách xóa khóa
```

Với cloud storage, cryptographic deletion hữu ích vì user thường không thể kiểm tra trực tiếp mọi replica, backup và thiết bị lưu trữ.

## 8. Liên hệ với Amazon S3 và AWS KMS

Amazon S3 là một ví dụ phù hợp khi thảo luận về cloud object storage. AWS cũng hỗ trợ client-side encryption cho Amazon S3, trong đó dữ liệu được mã hóa trước khi gửi lên S3.

Tuy nhiên, FADE không phải một AWS managed service. FADE là một research protocol và security pattern cho assured deletion. S3 client-side encryption là mô hình mã hóa được AWS hỗ trợ; FADE bổ sung thiết kế policy-based deletion quanh key management độc lập.

AWS KMS cũng liên quan đến chủ đề này vì nó quản lý cryptographic keys và hỗ trợ lên lịch xóa customer managed keys. AWS xem việc xóa KMS key là thao tác có tính phá hủy, vì dữ liệu được mã hóa bằng key đó có thể không khôi phục được sau khi key bị xóa.

Bài học thực tế không phải là AWS KMS chính là FADE. Bài học là key lifecycle management là một phần của data lifecycle management.

## 9. Không nhầm FADE với S3 Object Lock

S3 Object Lock bảo vệ object khỏi việc bị xóa hoặc ghi đè trong một khoảng retention period, hoặc khi có legal hold. Đây là cơ chế retention và compliance.

FADE giải quyết mục tiêu khác: làm encrypted data không thể đọc được sau khi key liên quan bị phá hủy.

```text
S3 Object Lock = ngăn xóa hoặc ghi đè
FADE           = làm dữ liệu không thể đọc sau khi xóa key
```

Hai cơ chế này giải quyết hai bài toán khác nhau.

## 10. Điểm mạnh

FADE có một số điểm mạnh quan trọng.

Thứ nhất, nó có thể hoạt động như một overlay phía trên cloud storage hiện có. Điều này làm thiết kế tương thích với các storage system vốn không được xây dựng riêng cho assured deletion.

Thứ hai, nó giảm phụ thuộc vào physical deletion. Trong hệ thống phân tán, kiểm chứng mọi replica và backup đã bị xóa là việc khó. FADE chuyển trọng tâm sang key destruction.

Thứ ba, nó hỗ trợ policy-based access control. Quyền truy cập phụ thuộc vào việc user còn thỏa policy được gắn với key hay không.

Thứ tư, nó tách trust boundary. Cloud storage giữ ciphertext, còn Key Managers kiểm soát key material cần thiết để giải mã.

## 11. Giới hạn

FADE không giải quyết mọi vấn đề của data deletion.

Giới hạn lớn nhất là niềm tin vào Key Managers. FADE giảm niềm tin đặt vào cloud storage provider, nhưng lại giả định Key Managers trung thực, an toàn và phá hủy key material đúng lúc.

Giới hạn khác là public verifiability. Nếu Key Manager nói rằng key đã bị phá hủy, user vẫn cần một cách để tin hoặc audit tuyên bố đó. Vì vậy các nghiên cứu sau này quan tâm đến verifiable deletion, Merkle tree, hash chain, trusted execution environment và blockchain-based evidence.

Ngoài ra còn có rủi ro vận hành. Nếu key bị xóa nhầm, dữ liệu có thể mất khả năng khôi phục thật sự. Trong production, key deletion cần có approval, logging, monitoring và recovery planning.

## 12. Kết luận

FADE hữu ích vì nó đặt lại cách nhìn về cloud deletion. Thay vì chỉ hỏi mọi bản sao vật lý đã biến mất chưa, FADE hỏi rằng những bản sao còn lại có còn giải mã được hay không.

Pattern cốt lõi là:

```text
Encrypt data before upload.
Store ciphertext in cloud storage.
Manage keys separately from the data.
Bind keys to access policies.
Destroy policy keys when access is revoked.
Without the key, the data becomes unreadable.
```

Với các hệ thống dùng Amazon S3 hoặc cloud object storage nói chung, bài học chính là: data deletion không chỉ là storage operation. Nó cũng là bài toán quản lý khóa mật mã.


## References

- FADE project page: https://ansrlab.cse.cuhk.edu.hk/software/fade/
- FADE: Secure Overlay Cloud Storage with File Assured Deletion: https://research.cuhk.edu.hk/en/publications/fade-secure-overlay-cloud-storage-with-file-assured-deletion-2/
- Secure Overlay Cloud Storage with Access Control and Assured Deletion: https://research.cuhk.edu.hk/en/publications/secure-overlay-cloud-storage-with-access-control-and-assured-dele-3/
- NIST SP 800-88 Rev. 1, Guidelines for Media Sanitization: https://csrc.nist.gov/pubs/sp/800/88/r1/final
- Amazon S3 client-side encryption: https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingClientSideEncryption.html
- AWS KMS key deletion: https://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys.html
- Amazon S3 Object Lock: https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html


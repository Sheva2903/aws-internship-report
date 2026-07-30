---
title: "Blog 3"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# CRYPTOGRAPHIC DELETION IN CLOUD STORAGE: WHEN DELETING DATA IS NOT JUST DELETING AN OBJECT

{{% notice warning %}}
Deleting an object from cloud storage and making data permanently unrecoverable are not always the same thing. In distributed storage, data may exist as replicas, backups, snapshots, or older versions. FADE approaches this problem by deleting the cryptographic key instead of relying only on physical deletion.
{{% /notice %}}

{{< figure src="/images/3-BlogsPosted/3.3-Blog3/fade-cryptographic-deletion-diagram.png" title="FADE: Cryptographic deletion for cloud storage" >}}

## 1. The cloud deletion problem

In a traditional file system, deleting a file may only remove the metadata or pointer to that file. The actual bytes may remain on disk until they are overwritten.

In cloud storage, the situation is harder to reason about. A stored object can be replicated for durability, backed up for recovery, moved across storage systems, or retained as an older version. The user normally does not control every physical copy of the data.

This creates a security question:

```text
When the cloud reports that data has been deleted, how can the user know that the data is no longer recoverable?
```

NIST SP 800-88 discusses media sanitization as a process that makes access to target data infeasible. One approach is crypto erase, where encrypted data is made inaccessible by destroying or invalidating the encryption key.

## 2. FADE in one sentence

FADE stands for **File Assured Deletion**. It is a secure overlay cloud storage design that protects outsourced files with encryption and makes them unrecoverable when the corresponding access policy is revoked.

The main idea is simple:

```text
Delete the key, not every cloud replica.
```

Instead of trying to prove that every physical copy has disappeared, FADE makes remaining encrypted copies useless by destroying the key material required to decrypt them.

## 3. Architecture

FADE separates data storage from key management.

**FADE Client** encrypts files before upload and decrypts files after download. Temporary keys are generated locally and removed after the upload process.

**Cloud Storage**, such as Amazon S3 in a cloud-storage discussion, stores encrypted files and encrypted metadata. It does not hold the full decryption path needed to recover plaintext.

**Key Managers** manage policy-bound keys. They release key material only when the user satisfies the required access policy. When the policy expires or is revoked, the relevant key material is destroyed.

This is why FADE is called an overlay system: it can be placed above existing cloud storage without requiring the storage provider to redesign the underlying storage infrastructure.

## 4. The key chain

FADE does not encrypt a file and store the raw key beside it. It creates a chain of dependent keys.

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

The decryption path is therefore:

```text
Policy key → S → Data Key K → File F
```

If the policy key is destroyed, `S` cannot be recovered. If `S` cannot be recovered, `K` cannot be recovered. If `K` cannot be recovered, the file remains unreadable.

## 5. Upload flow

A simplified upload flow looks like this:

```text
1. The user selects file F.
2. The client assigns an access policy P to F.
3. The client generates Data Key K.
4. The client encrypts F with K.
5. The client generates key S.
6. The client encrypts K with S.
7. The client protects S using the policy key for P.
8. The encrypted file and metadata are uploaded to cloud storage.
9. Temporary local keys are deleted.
```

After upload, the cloud stores ciphertext and encrypted metadata. It does not have the complete key chain needed to recover the plaintext file.

## 6. Download flow

During download, the client retrieves the encrypted file and metadata from cloud storage. It then contacts the Key Managers.

If the access policy is still valid, the Key Managers help recover the key material needed to obtain `S`. The client then uses `S` to recover `K`, and uses `K` to decrypt the file.

```text
User satisfies policy
    ↓
Key Managers release/decrypt S
    ↓
Client recovers K
    ↓
Client decrypts the file
```

If the policy is not valid, the key material is not released. The encrypted object may still exist, but it remains unreadable.

## 7. Assured deletion

Assured deletion happens when the access policy expires or is revoked.

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

FADE does not prove that every physical replica has been wiped. It provides a cryptographic guarantee: any remaining encrypted copy cannot be decrypted without the destroyed key.

This distinction matters:

```text
Physical deletion       = remove or overwrite stored data
Cryptographic deletion  = make encrypted data unreadable by destroying the key
```

For cloud storage, cryptographic deletion is useful because the user usually cannot inspect every replica, backup, and storage device directly.

## 8. Relation to Amazon S3 and AWS KMS

Amazon S3 is a useful example of cloud object storage when discussing this pattern. AWS also supports client-side encryption for Amazon S3, where data is encrypted before it is sent to S3.

However, FADE is not an AWS managed service. It is a research protocol and security pattern for assured deletion. S3 client-side encryption is an AWS-supported encryption model; FADE adds a specific policy-based deletion design around independent key management.

AWS KMS is also related to this discussion because it manages cryptographic keys and supports scheduled deletion of customer managed keys. AWS treats KMS key deletion as a destructive operation because data encrypted under that key may become unrecoverable after the key is deleted.

The practical lesson is not that AWS KMS equals FADE. The lesson is that key lifecycle management is part of data lifecycle management.

## 9. Do not confuse FADE with S3 Object Lock

S3 Object Lock protects objects from deletion or overwriting for a fixed retention period or under a legal hold. It is a retention and compliance mechanism.

FADE addresses a different goal: making encrypted data unreadable after the relevant key is destroyed.

```text
S3 Object Lock = prevent deletion or overwrite
FADE           = make data unreadable after key deletion
```

These mechanisms solve different problems.

## 10. Strengths

FADE has several important strengths.

First, it can work as an overlay above existing cloud storage. This makes the design compatible with storage systems that were not originally built for assured deletion.

Second, it reduces reliance on physical deletion. In a distributed system, verifying every replica and backup is difficult. FADE shifts the security control to key destruction.

Third, it supports policy-based access control. Access depends on whether the user still satisfies the policy bound to the key.

Fourth, it separates trust boundaries. Cloud storage keeps ciphertext, while Key Managers control the key material required for decryption.

## 11. Limitations

FADE is not a complete answer to every deletion problem.

The main limitation is trust in the Key Managers. FADE reduces trust in the cloud storage provider, but it assumes the Key Managers are honest, secure, and correctly destroy key material when required.

Another limitation is public verifiability. If a Key Manager claims that a key was destroyed, the user still needs a way to trust or audit that claim. This is why later research explores verifiable deletion, Merkle trees, hash chains, trusted execution environments, and blockchain-based evidence.

There is also an operational risk. If a key is deleted by mistake, the data can become unrecoverable. In production systems, key deletion should require approval, logging, monitoring, and recovery planning.

## 12. Conclusion

FADE is useful because it reframes cloud deletion. Instead of asking only whether every stored copy has physically disappeared, it asks whether any remaining copy is still decryptable.

The core pattern is:

```text
Encrypt data before upload.
Store ciphertext in cloud storage.
Manage keys separately from the data.
Bind keys to access policies.
Destroy policy keys when access is revoked.
Without the key, the data becomes unreadable.
```

For systems using Amazon S3 or cloud object storage in general, the key lesson is clear: data deletion is not only a storage operation. It is also a cryptographic key management problem.


## References

- FADE project page: https://ansrlab.cse.cuhk.edu.hk/software/fade/
- FADE: Secure Overlay Cloud Storage with File Assured Deletion: https://research.cuhk.edu.hk/en/publications/fade-secure-overlay-cloud-storage-with-file-assured-deletion-2/
- Secure Overlay Cloud Storage with Access Control and Assured Deletion: https://research.cuhk.edu.hk/en/publications/secure-overlay-cloud-storage-with-access-control-and-assured-dele-3/
- NIST SP 800-88 Rev. 1, Guidelines for Media Sanitization: https://csrc.nist.gov/pubs/sp/800/88/r1/final
- Amazon S3 client-side encryption: https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingClientSideEncryption.html
- AWS KMS key deletion: https://docs.aws.amazon.com/kms/latest/developerguide/deleting-keys.html
- Amazon S3 Object Lock: https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html


---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# SHARED RESPONSIBILITY MODEL ON AWS: CLOUD SECURITY DOES NOT END WITH THE PROVIDER

Cloud computing reduces the operational burden of managing physical infrastructure. Instead of maintaining data centers, servers, network devices, and storage systems directly, users can provision resources through platforms such as AWS.

However, moving to AWS does not mean that all security responsibilities move to AWS. Cloud changes the boundary of responsibility. It does not remove the user's responsibility.

Cloud security studies emphasize that cloud computing is not only a technical platform. It also involves architecture, processes, people, policies, data, applications, providers, auditors, and extended supply chains. For that reason, security in cloud environments must be treated as a shared responsibility model rather than a fully outsourced responsibility.

## 1. What is the Shared Responsibility Model?

AWS describes the model through two areas: **security of the cloud** and **security in the cloud**.

AWS is responsible for protecting the infrastructure that runs AWS services. This includes data centers, physical servers, storage hardware, networking infrastructure, and the foundational layers of managed services.

Customers are responsible for how they use AWS. This includes their data, identity configuration, access policies, network rules, application code, guest operating systems in some services, encryption choices, logging, and internal security processes.

A simple way to understand the model is this:

> AWS secures the cloud foundation. Customers secure what they build and configure on top of it.

{{< figure src="images/3-BlogsPosted/3.1-Blog1/aws-shared-responsibility-model.png" title="Shared Responsibility Model on AWS" >}}

## 2. Responsibility Changes by Service

The responsibility boundary is not the same for every AWS service.

With **Amazon EC2**, customers receive more control, but also more responsibility. They must manage the guest operating system, software updates, Security Groups, SSH keys, application security, and the data stored on the instance.

With **Amazon RDS**, AWS manages more of the database infrastructure. Customers still manage database users, network access, encryption settings, parameter groups, backup configuration, and application connections.

With **Amazon S3**, AWS operates the storage infrastructure. Customers must configure bucket policies, Block Public Access, encryption, lifecycle rules, versioning, and access logging.

With **AWS Lambda**, AWS manages the execution platform, scaling, and much of the runtime infrastructure. Customers still own the function code, dependencies, IAM role, secrets, environment variables, and input validation logic.

This matches the general cloud security view of IaaS, PaaS, and SaaS. The more managed a service is, the more infrastructure responsibility moves to the provider. However, data, access control, configuration, and application behavior always remain important customer responsibilities.

## 3. Many Cloud Incidents Start from Misconfiguration

Many cloud security incidents do not begin with a failure of the cloud provider's physical infrastructure. They often begin with customer-side configuration mistakes.

Common examples include:

* IAM policies that grant more permissions than required.
* S3 buckets exposed to the public without a valid reason.
* Security Groups that open sensitive ports to the Internet.
* Access keys committed to a Git repository.
* Secrets stored directly in source code or plaintext environment variables.
* Logging disabled, incomplete, or never reviewed.
* Sensitive data stored without encryption.
* No clear backup or recovery plan.

Cloud features such as resource pooling, virtualization, multi-tenancy, broad network access, and on-demand self-service make cloud platforms flexible. The same features also make security more dependent on clear configuration, strict access control, monitoring, and governance.

## 4. IAM Should Be Controlled First

On AWS, IAM controls who can perform which action, on which resource, and under which conditions. A single over-permissioned policy can turn a small mistake into a serious incident.

The practical rule is **least privilege**. Each user, role, service, and workload should receive only the permissions needed for its task.

For example, a Lambda function that only reads files from one S3 prefix should not receive full access to all S3 resources in the account. Its policy should be limited to the required bucket, prefix, and read actions.

A secure IAM design should also consider MFA, role-based access, short-lived credentials, permission boundaries, and regular access review.

## 5. S3 Requires Careful Public Access Control

Amazon S3 is widely used for logs, backups, static files, deployment artifacts, analytics data, and customer content. Because of that, S3 misconfiguration can have serious consequences.

A private bucket should stay private by default. S3 Block Public Access should be used unless there is a clear reason to expose objects publicly.

For each bucket, basic questions should be answered:

```text
Does this bucket need to be public?
Who can read objects?
Who can write objects?
Is encryption enabled?
Is versioning required?
Is access logging required?
Should old versions be managed with lifecycle rules?
```

S3 is durable and highly managed, but AWS cannot decide which of the customer's objects are sensitive. That decision belongs to the customer.

## 6. Without Logs, Incidents Become Guesswork

Security is not only prevention. A system must also support investigation.

When something changes in an AWS account, the team should be able to answer:

```text
Who made the change?
When did it happen?
From where was the action performed?
Which resource was affected?
Was the action expected?
```

AWS CloudTrail records account activity and API calls across AWS services. CloudWatch Logs, VPC Flow Logs, S3 access logs, AWS Config, GuardDuty, and Security Hub can provide additional visibility depending on the workload.

Logging should be enabled before incidents happen. If logging is added only after an incident, important evidence may already be missing.

## 7. Practical Checklist

A basic AWS security checklist should include:

```text
IAM:
- Enable MFA for privileged users.
- Avoid daily use of the root account.
- Replace long-lived access keys with roles where possible.
- Review policies that use "*" broadly.

S3:
- Keep buckets private unless public access is required.
- Enable Block Public Access for private buckets.
- Encrypt sensitive data.
- Use versioning or backup for important data.

Network:
- Avoid exposing databases directly to the Internet.
- Place internal services in private subnets.
- Limit inbound rules in Security Groups.
- Review outbound access where needed.

Logging:
- Enable CloudTrail.
- Store logs in a protected location.
- Monitor changes to IAM, S3 policies, and Security Groups.
- Create alerts for risky administrative actions.

Application:
- Do not store secrets in source code.
- Validate input.
- Patch dependencies.
- Use managed secret storage where appropriate.
```

## 8. Conclusion

AWS removes much of the burden of operating physical infrastructure, but it does not remove the customer's security responsibilities. The user still controls data, permissions, service configuration, application code, logging, and recovery design.

The Shared Responsibility Model is the foundation for designing secure systems on AWS. When the boundary is understood clearly, teams are more likely to configure IAM carefully, keep S3 private by default, design networks with limited exposure, enable logging, protect secrets, and prepare for recovery.

Cloud security should therefore be understood as a partnership:

> AWS protects the platform. Customers protect their use of the platform.

## References

- AWS Shared Responsibility Model: https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/shared-responsibility.html
- AWS IAM best practices: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
- Amazon S3 Block Public Access: https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-control-block-public-access.html
- AWS CloudTrail Documentation: https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html
- Gururaj Ramachandra, Mohsin Iftikhar, Farrukh Aslam Khan, "A Comprehensive Survey on Security in Cloud Computing", Procedia Computer Science, 2017.
- M. Ali, S. U. Khan, A. V. Vasilakos, "Security in cloud computing: Opportunities and challenges", Information Sciences, 2015.

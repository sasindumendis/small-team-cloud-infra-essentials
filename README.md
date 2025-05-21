# **Cloud Infrastructure Best Practice Essentials for Small Teams**

# **TL;DR**

* Create network boundaries by isolating your infrastructure assets for different workloads and stages through a multi-account or multi-VPC setup.

* Provision individual identities with minimum required privileges for everyone accessing the infrastructure (avoid relying on shared credentials/ keys and assigning excessive privileges). Enforce 2FA and a strong password policy.

* Put the servers in private subnets by default. Only leave required ports open and allow traffic only from/ to intended sources and destinations.

* Harden your app instances/ containers, run them in a stateless manner whenever possible and keep them up to date.

* Make sure stored data is properly secured.

* Enable logging on all relevant resources, and configure monitoring and alerting for anomalies.

* Configure and backup frequently and systematically. This can be critical for business continuity in case of denial of services attacks and ransomware attacks.

* Design your infrastructure in a disposable manner and automate provisioning as much as possible.

# **Cloud Computing Account Management**

* Isolate different workloads (public facing vs management, development vs production...) by setting up multiple accounts, or at least separate VPCs.

* For multiple separate projects with multiple teams under the same organization:

  * In AWS: Set up separate accounts for each of them and configure AWS Organizations and ControlTower for centralized management.

  * In GCP: Set up separate Projects.

* Establish a resource tagging strategy and enforce it.

* Periodically remove no longer used resources after taking necessary backups.

## AWS

* Ensure auditing of AWS accounts is possible by configuring AWS CloudTrail and AWS Config.

* Enable Amazon GuardDuty for continuous threat detection and Security Hub for overall security posture monitoring.

* Block account level public access to S3 buckets.

# **Identity & Access Management**

## AWS

* Do not use AWS root user for operations other than billing or support contract modifications. For all other operations, access provisioned with Identity Center (for multi-account setups) and/ or IAM should be used.

* Enable MFA on the root user and all IAM users with a console password.

* Configure IAM password policy to ensure strong password requirements.

* Configure sensible password and key rotation requirements.

* Avoid associating specific policies for individual users, instead inherit them through groups having roles with minimum required privileges.

* Use roles also for inter-service access management.

# **Network Security**

## AWS

* Treat VPCs as the base layer of networking and security. Create separate VPCs for each of different workloads and stages to isolate them and control access. Avoid using the default VPC as it is.

  * [Everything You Need To Know About Networking On AWS](https://grahamlyons.com/article/everything-you-need-to-know-about-networking-on-aws)

  * [How to deploy a production-grade VPC on AWS](https://gruntwork.io/guides/networking/how-to-deploy-production-grade-vpc-aws/#what-is-a-vpc)

* Place your application servers and databases in private Subnets.

* If SSHing application servers, databases etc. across the internet is needed, set up a bastion host in a public subnet to jump through or make use of services such as AWS Systems Manager.

* Create and attach Security Groups with fine-grained inbound/ outbound rules for all relevant resources. Avoid leaving the default Security Group to be used.

* Make sure all internet facing resources such as Load Balancers, API Gateways, CDNs, bastion hosts etc. only accept SSL/TLS connections. AWS Certificate Manager or services like LetsEncrypt can be used to manage certificates.

* Configure AWS WAF to protect your Load Balancers, API Gateways etc. Or alternatively opting to use services offered by Cloudflare is also recommended.

# **Server Setup & Maintenance**

* Recommendations for Linux instances:

  * Always set the timezone on servers to UTC. If you are not using Amazon Linux (which comes preconfigured), you should also confirm your servers configure NTP correctly,

  * Install and configure Fail2Ban.

  * Set up automatic periodic security patching with Unattended Upgrades.

  * Disable password based SSH access.

  * Configure log rotation to avoid filling up disk space over time with Logrotate.

  * Refer [Ubuntu hardening guide](https://blog.codelitt.com/my-first-10-minutes-on-a-server-primer-for-securing-ubuntu/) for essentials.

* If sensible in your case, use [CIS hardened Images](https://www.cisecurity.org/cis-hardened-images/).

* Join the security advisory mailing lists for any third party software you use and monitor those lists for announcements of critical security vulnerabilities.

## AWS

* Depending on the infrastructure set up, script above instance-initializing steps for repeated use, or create an AMI to base new instances off.

* Do NOT share EC2 KeyPairs with your team. Instead, authorize individuals to use their own keys.

* You may also use AWS Systems Manager for managing patches for the OSs and applications.

# **Deployments**

* Do not commit any sensitive data into your code repository.

* Do not hardcode secrets in your pipeline scripts. Use environment variable management  mechanisms provided by the CI/CD solution instead.

* Store your application secrets required in runtime like database passwords using AWS Secrets Manager or similar tools which handles encryption, rotation, and access control. Avoid leaving them in plain text.

* Make sure your docker containers use up-to-date base images.

  * [Docker security best practices](https://snyk.io/blog/10-docker-image-security-best-practices/)

# **Logging, Monitoring & Alerting**

* Assess, decide on and configure solutions to collect and aggregate application, system and infrastructure logs separately for easy centralized auditability, metrics monitoring and alert management (e.g.: AWS CloudWatch, GCP StackDriver,  NewRelic, DataDog...).

## AWS

* If there are no dedicated solutions configured for log aggregation, important metrics monitoring and alarm management, forward all logs to CloudWatch groups and setup monitoring and alarms as required.

  * Configure alerts on key metrics: e.g., high CPU usage on EC2 instances, too many 4xx or 5xx errors on load balancers, low disk space on your database.

* Make sure important service level logs are collected such as Amazon VPC Flow Logs, Amazon S3, CloudTrail, and Elastic Load Balancer access logging.

# **Backups, Recovery & Automation**

* Anything that cannot be easily reproduced if lost should be backed up on a frequency sensible to the specific business context (this typically includes not only databases and user uploaded data but things like the DNS configurations etc. as well.). Ideally this should also be automated.

* Periodically test if the backups are restorable without issues.

* All code deployments to long running environments should be properly tagged in Git (or the DVCS in use) for easy redeployability.

* Based on the availability needs of your project, design your infrastructure so that each layer of it can be replaced with minimal disruption (for example DNS failover, auto scaling app instances, database failover etc).

* Decide on an off-site backup strategy for the project and keep your code, data and configuration/ provisioning steps ready in case of a rare event of data center or availability zone failure, cloud account compromise or suspension etc.

* It is also highly recommended to automate provisioning your infrastructure using AWS CloudFormation or Terraform etc. so that in the event of a disaster an identical setup can be easily recreated.

## AWS

* Configure redundancy level, versioning support and archival strategy (object lifecycle settings) for S3 buckets depending on your project needs.

* If you run your applications in EC2 in a stateful manner, EBS volumes or the data directories should also be backed up on a sensible frequency.

* If high availability of your database is critical, configure RDS multi-AZ setup.

* For RDS and other database/ data storage services automatic backups should be configured. Alternatively you may configure them centrally from AWS Backup.

* AWS provided automatic backups for RDS etc do not expose individual databases in a manner you could restore them elsewhere. Therefore you might also need to have a mechanism to backup individual databases for off-site keeping or populating non-production environments etc.

# **Data Protection**

* Make sure user data, application assets, backups, logs etc. are encrypted at rest and stored private by default. Only allow access to them through intended services.

## AWS

* Use AWS Key Management Service (KMS) to protect data at rest across a wide range of AWS services and your applications. Enable default encryption for Amazon EBS volumes, and Amazon S3 buckets, RDS and ElastiCache instances.

* Most AWS services used for keeping different kinds of data also offer deletion protection. Configure it where necessary.

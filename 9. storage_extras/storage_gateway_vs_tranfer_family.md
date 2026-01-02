Here’s a breakdown of their differences, use cases, and how to choose.

### At a Glance

| Feature | AWS Storage Gateway | AWS Transfer Family |
| :--- | :--- | :--- |
| **Primary Purpose** | **Extend on-premises storage** to AWS. Makes cloud storage appear as a local, on-premises resource. | **Managed file transfer service** for sending files to/from AWS storage **using standard protocols**. |
| **Core Analogy** | A "bridge" or "cache" that seamlessly connects local apps to the cloud. | A "managed FTP/SFTP/FTPS server" in the cloud. |
| **Key Protocols** | **iSCSI** (block), **NFS** (file), **SMB** (file), **VTL** (tape). | **SFTP** (SSH File Transfer Protocol), **FTPS** (FTP over SSL), **FTP**, **AS2** (for EDI). |
| **Storage Backend** | S3 (via file/volume gateways), S3 Glacier (via tape gateway), EBS (cached volumes). | **Directly to/from S3 or EFS.** |
| **Deployment Model** | **Hybrid.** Requires a software appliance (VM, hardware, or as a service) deployed in your data center or as an EC2 instance. | **Cloud-native.** Fully managed service, no infrastructure to manage. |
| **Data Access Pattern** | Low-latency, frequent access for on-premises applications. Often caches active data locally. | Internet-based file uploads/downloads. Episodic, bulk transfers. |
| **Authentication** | Integrated with AD (SMB), IAM (NFS), or CHAP (iSCSI). | Managed via AWS IAM, custom Lambda, or AD/LDAP connectors. |
| **Use Case Driver** | **"My on-premises application needs to use cloud storage as if it were local."** | **"My partners/users need to send me files via SFTP, and I want them stored directly in S3."** |

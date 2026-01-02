# EBS vs EFS

| Feature          | **EBS (Elastic Block Store)**                       | **EFS (Elastic File System)**                                         |
| ---------------- | --------------------------------------------------- | --------------------------------------------------------------------- |
| **Storage Type** | Block storage                                       | File storage                                                          |
| **Attachment**   | Attached to **one EC2 instance** at a time          | Shared with **multiple EC2 instances** (NFS protocol)                 |
| **Availability** | Tied to a **single AZ**                             | Multi-AZ, accessible across a **region**                              |
| **Performance**  | Measured in **IOPS (Input/Output per Second)** (majorly)      | Measured in **Throughput (MB/s)** (majorly)                                     |
| **Durability**   | 99.999% durability (data replicated within AZ)      | 99.999999999% durability (across AZs)                                 |
| **Latency**      | **Low latency**, suitable for block-level access    | **Higher latency**, due to NFS and multi-AZ                           |
| **Data Access**  | Through OS block device (e.g., `/dev/xvdf`)         | Via mount target (e.g., `/mnt/efs`)                                   |
| **Use Cases**    | OS boot volumes, RDS, NoSQL DBs, transactional apps | Web servers, container storage, Big Data pipelines                    |
| **Backup**       | Use **EBS Snapshots** to S3                         | **Automatic backups** (if enabled) to S3                              |
| **Scaling**      | Manual scaling (volume size, type, performance)     | **Auto-scalable** (grows and shrinks as needed)                       |
| **Encryption**   | KMS encryption supported                            | KMS encryption supported                                              |


## **💰 Pricing Comparison**

## **📊 Quick Price Summary (us-east-1)**

| Storage Type | Price per GB/Month | Throughput Cost | IOPS Cost | Best For |
|-------------|-------------------|----------------|-----------|----------|
| **EBS gp3** | $0.08/GB | Free up to 125 MB/s, then $0.04/MBps-month | Free up to 3,000 IOPS, then $0.005/IOPS-month | Databases, boot volumes |
| **EBS io2** | $0.125/GB | Included | $0.005/IOPS-month (provisioned) | High-performance databases |
| **EFS Standard** | $0.30/GB | Baseline free, extra $6/MBps-month (provisioned) | Included in throughput | Shared files, web content |
| **EFS IA** | $0.08/GB | Same as Standard | Same as Standard | Rarely accessed shared files |
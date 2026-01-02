# **Amazon Aurora - Complete MD Notes**

## **🎯 CORE ARCHITECTURE**

### **Shared Storage Cluster:**
```
6 Copies Across 3 AZs:
AZ1: [Copy 1] [Copy 2]
AZ2: [Copy 3] [Copy 4]
AZ3: [Copy 5] [Copy 6]

Write: 4/6 copies must acknowledge
Read: 3/6 copies sufficient
```

**Key:** No EBS volumes - All instances share single cluster volume

---

## **⚡ DEPLOYMENT OPTIONS**

### **1. Provisioned Aurora:**
```yaml
Max Readers: 15 per cluster 
Instance Types: Fixed size (r-class, t-class)
Best for: Predictable workloads
```

### **2. Aurora Serverless v2:**
```yaml
Scaling: Auto-scales ACU (0.5-256)
Readers: Auto-managed (no fixed limit)
Best for: Variable/unpredictable workloads
Cost: Pay per ACU-second
```

### **3. Global Database:**
```yaml
Cross-region: 1 primary + up to 5 secondaries (total MAX 6 clusters)
Replication: < 1 second RPO
Failover: < 1 minute RTO
Max: 15 readers per region
```

---

## **Latest Aurora Global Database Architecture (2024)**

### **Architecture Structure:**

### **1. Primary Region:**
- **1 Writer instance** (does all writes)
- **Up to 15 Read replicas** (16 total instances including writer)
- **1 Storage volume** (6-way replicated across 3 AZs)

### **2. Secondary Regions (1-5 more regions):**
- **Up to 16 Read-only replicas** per region
- **Each secondary has its own storage volume** (6-way replicated)
- **Total clusters:** 1 primary + up to 5 secondaries = **6 clusters max**

---


### **Visual Layout:**
```
PRIMARY REGION (us-east-1):
┌─────────────────────────────────────┐
│           Storage Volume A           │
│  (6 copies across 3 AZs, 128 TiB)   │
└─────────────────────────────────────┘
            │
    ┌───────┼───────┐
    ▼       ▼       ▼
[Writer] [Reader1] [Reader2] ... [Reader15]
  (1)     (2-16)   (max 15 readers total)


SECONDARY REGION 1 (eu-west-1):
┌─────────────────────────────────────┐
│           Storage Volume B           │
│   (async replica of Volume A)        │
└─────────────────────────────────────┘
            │
    ┌───────┼───────┐
    ▼       ▼       ▼
[Reader1] [Reader2] ... [Reader16]
(promotable) (read-only) (max 16 total)
         ↑
   (1 is writer-capable)

SECONDARY REGION 2 (ap-south-1):
┌─────────────────────────────────────┐
│           Storage Volume C           │
│   (async replica of Volume A)        │
└─────────────────────────────────────┘
            │
    ┌───────┼───────┐
    ▼       ▼       ▼
[Reader1] [Reader2] ... [Reader16]
(promotable) (read-only)
```


---

## **📊 STORAGE & PERFORMANCE**

### **Storage Architecture:**
- **Cluster Volume**: Auto-scales 10GB-128TB
- **IOPS**: Up to 256K (vs RDS 80K max)
- **Zero Impact Backups**: Continuous to S3
- **No Provisioned IOPS Needed**

### **Performance vs RDS:**
```
Aurora: 5x MySQL, 3x PostgreSQL throughput
Write: Parallel, distributed storage
Read: Low-latency replicas
```

---

## **🔄 REPLICATION & SCALING**

### **Reader Scaling:**
```
MySQL/PostgreSQL: Max 15 readers per cluster
Automatic Load Balancing: Reader endpoint
Custom Endpoints: Group specific readers
```

### **Write Forwarding (Global DB):**
```yaml
Feature: Readers can forward writes to writer
Use: Global apps with local write appearance
Latency: Extra 5-10ms per forwarded write
```

---

## **💾 BACKUP & RECOVERY**

### **Three Backup Types:**
```yaml
1. Automated Backups: 1-35 days, continuous PITR
2. Manual Snapshots: Unlimited retention
3. Backtrack: Rewind up to 72 hours (unique!)
```

### **Unique Features:**
```
Fast Clone: Copy-on-write, seconds
Backtrack: Rewind bad changes in minutes
Restore Speed: Minutes (not hours)
```

---

## **🔐 SECURITY**

### **Encryption:**
```yaml
At Rest: KMS (enabled by default)
In Transit: TLS 1.2 mandatory
Key Management: Customer managed keys (CMK)
```

### **Authentication:**
```yaml
Standard: Username/password
IAM Auth: MySQL/PostgreSQL (recommended)
Kerberos: AD integration
```

---

## **📈 MONITORING**

### **Monitoring Stack:**
```yaml
CloudWatch: Basic metrics (free)
Performance Insights: Query performance
Database Insights: Unified view (Standard/Advanced)
Enhanced Monitoring: OS-level (optional)
```

### **Key Metrics:**
```sql
AuroraReplicaLag: < 100ms ideal
VolumeBytesUsed: Storage utilization
DatabaseConnections: Per instance
```

---

## **💰 PRICING MODELS**

### **Two Storage Options:**
```yaml
1. I/O-Optimized: Predictable pricing
   • I/O costs >25% of total → Choose this
   • No separate I/O charges

2. Standard: Pay-per-request I/O
   • I/O costs <25% of total → Choose this
   • Separate I/O billing
```

### **Cost Components:**
```
Monthly Cost = 
  Instance hours × Rate
+ Storage GB × Rate
+ I/O operations × Rate (Standard only)
+ Additional features
```

---

## **🚀 MODERN FEATURES**

### **RDS Data API:**
```yaml
What: SQL over HTTP endpoint
Use: Serverless applications
Access: CLI, SDK, Query Editor
```

### **RDS Proxy:**
```yaml
Purpose: Connection pooling
Best for: Lambda, serverless
Benefits: Faster failover, IAM auth
```

### **DevOps Guru Integration:**
```yaml
AI-powered anomaly detection
Cost: $0.0042/hour per resource
Automatic recommendations
```

---

## **⚙️ CONFIGURATION ESSENTIALS**

### **Networking:**
```yaml
Network Type: IPv4 or Dual-stack
Public Access: Avoid in production
VPC: Cannot change after creation
Security Groups: Stateful firewall
```

### **High Availability:**
```yaml
Multi-AZ: Automatic 6-way replication
Failover: < 30 seconds
Cross-Region: Global Database needed
```

---

## **🎯 USE CASES**

### **Best For Aurora:**
```
✅ High-performance OLTP
✅ Variable workloads (Serverless v2)
✅ Global applications
✅ Need >5 read replicas
✅ Fast clone/backtrack needs
```

### **Stick with RDS When:**
```
✅ Need specific engine versions
✅ Lift-and-shift migrations
✅ Predictable, smaller workloads
✅ Budget constraints
```

---

## **⚠️ LIMITATIONS**

### **Hard Limits:**
```yaml
Max 15 readers (provisioned)
Max 35 days automated backups
Max 72 hours backtrack
Storage: 128TB max per cluster
```

### **Not Supported:**
```
❌ MySQL 5.6, 5.5
❌ PostgreSQL < 10.x
❌ Cross-account snapshot sharing (some limitations)
❌ Some MySQL/PostgreSQL extensions
```

---

## **🔧 BEST PRACTICES**

### **Production Setup:**
```yaml
Deployment: Multi-AZ (always)
Storage: I/O-Optimized
Instance: Graviton3 (r7g) for 25% better perf
Backup: 35 days + backtrack 24h
Monitoring: Performance Insights + CloudWatch Alarms
```

### **Development Setup:**
```yaml
Deployment: Single AZ (cost saving)
Instance: Serverless v2 or t-class
Backup: 7 days (minimum)
```

---

## **⚡ PERFORMANCE TIPS**

### **Optimize Reads:**
```sql
-- Use reader endpoint for SELECTs
-- Monitor replicaLag (< 100ms)
-- Implement connection pooling (RDS Proxy)
```

### **Optimize Writes:**
```sql
-- Batch writes where possible
-- Use appropriate indexes
-- Monitor writer instance metrics
```

---

## **🚨 CRITICAL SETTINGS**

### **Never Miss These:**
```bash
# Enable deletion protection
aws rds modify-db-cluster --deletion-protection

# Set max backup retention
aws rds modify-db-cluster --backup-retention-period 35

# Enable backtrack (if needed)
aws rds modify-db-cluster --backtrack-window 86400  # 24h
```

### **Monitoring Alarms:**
```yaml
Must Have:
• CPUUtilization > 80%
• FreeableMemory < 20%
• AuroraReplicaLag > 100000ms (100ms)
• DatabaseConnections > 80% of max
```

---

## **🎓 KEY DIFFERENCES FROM RDS**

| Feature | Aurora | RDS Standard |
|---------|--------|--------------|
| **Storage** | Shared cluster volume | EBS per instance |
| **Max Readers** | 15 | 5 |
| **Backup Impact** | Zero | Some during window |
| **Clone Speed** | Seconds | Minutes/Hours |
| **Backtrack** | Yes (72h) | No |
| **Global DB** | Built-in | Manual setup |
| **Cost** | Higher entry, better scaling | Lower entry |

---

## **📋 QUICK START CHECKLIST**

### **Create Production Cluster:**
```yaml
1. Storage: Aurora I/O-Optimized
2. Instance: db.r7g.large (Graviton3)
3. Deployment: Multi-AZ with 1-2 readers
4. Backup: 35 days + backtrack 24h
5. Security: IAM auth + no public access
6. Monitoring: Performance Insights + Alarms
7. Protection: Deletion protection ON
```

### **Maintenance:**
```yaml
Weekly: Check CloudWatch metrics
Monthly: Test backup restore
Quarterly: Review instance sizing
Yearly: Review retention policies
```

---

## **Final Verdict:**
**Aurora =** When you need high performance, automatic scaling, and advanced features  
**RDS =** When you need specific engines, simpler pricing, or lift-and-shift

**Aurora's killer features:** Fast clones, backtrack, 15 readers, Serverless v2, Global Database
# **AWS RDS - Complete Modern Notes** 🚀

## **🎯 CORE CONCEPT**

Amazon RDS is a **managed relational database service** that handles provisioning, backups, patching, monitoring, and failover.

**Key Distinction:** Uses **EC2-style EBS storage per instance** (not shared storage like Aurora).

---

## **🏗️ ARCHITECTURE & DEPLOYMENT**

### **Three Deployment Options:**

| Type | Instances | SLA | Key Feature | Cost |
|------|-----------|-----|-------------|------|
| **Single-AZ** | 1 | 99.5% | No redundancy | 1x |
| **Multi-AZ DB Instance** | 2 | 99.95% | Hidden standby | 2x |
| **Multi-AZ DB Cluster** | 3 | 99.95% | 2 readable standbys | 3x |

### **Modern Updates:**
- **Multi-AZ DB Cluster** (new): Primary + 2 readable standbys across AZs
- **Automatic failover** <35 seconds (vs 60-120s traditional)
- **Read scaling** built-in with cluster reader endpoint
- **MySQL 8.0.28+ only** currently

---

## **💾 STORAGE & PERFORMANCE**

### **Storage Types:**
- **gp3**: General Purpose SSD (default)
- **io2/io1**: Provisioned IOPS (high performance)
- **Magnetic**: Legacy (avoid)

### **Modern Features:**
- **Amazon RDS Optimized Writes** (2x write throughput at no extra cost)
- Requires: MySQL 8.0.30+ or PostgreSQL 13.12+/14.9+/15.4+
- Requires: db.m6gd/m6id/r6gd/r6id instances
- **io2 Block Express**: Up to 256,000 IOPS per instance

### **Instance Classes:**
```yaml
Graviton3 (Latest): db.m7g/r7g
• 25% better performance
• 20% lower cost
• Supports Optimized Writes
```

---

## **⚡ SCALING CAPABILITIES**

### **Write Scaling:**
❌ **NOT SUPPORTED** - Single writer instance only

### **Read Scaling:**
✅ **Supported via Read Replicas**
- Max **5 read replicas** per source DB
- **Asynchronous replication** (possible lag)
- **Manual routing** required (no auto-load balancer)
- Each replica has its own EBS storage

### **Cross-Region Read Replicas:**
- For disaster recovery or global reads
- **Manual promotion** required
- No automatic cross-region failover

---

## **🔐 MODERN SECURITY**

### **Authentication Options:**
1. **Password Only** (traditional)
2. **Password + IAM Database Auth** (recommended for MySQL/PostgreSQL)
3. **Password + Kerberos** (enterprise AD integration)

### **Credentials Management:**
```yaml
Self-managed: Traditional username/password
AWS Secrets Manager (Recommended):
• Automatic password generation & rotation
• IAM integration
• Additional cost applies
• Most secure option
```

### **Encryption:**
- **At rest**: KMS (AES-256) - BYOK supported
- **In transit**: SSL/TLS mandatory
- **Certificate Authority**: Choose custom or default (rds-ca-rsa2048-g1)

---

## **🔄 REPLICATION & HA**

### **Multi-AZ (Traditional):**
- **Synchronous replication** to standby in different AZ
- Standby is **NOT readable** (hidden)
- Automatic failover in **60-120 seconds**
- **Zero data loss** for most engines

### **Multi-AZ DB Cluster (New):**
- **Primary + 2 readable standbys** across AZs
- Both standbys are **READABLE**
- Automatic failover in **<35 seconds**
- **Reader endpoint** for load-balanced reads
- **Reduced write latency**

### **RDS Proxy** 🎯
```yaml
Purpose: Connection pooling layer
Benefits:
• Perfect for serverless (Lambda)
• Faster failover handling
• IAM/Secrets Manager integration
• Reduces "too many connections" errors
Cost: Additional $0.015/hour + per-connection
```

---

## **💾 BACKUP & RECOVERY**

### **Automated Backups:**
- **Daily snapshots** + transaction logs every 5 minutes
- Enables **Point-in-Time Recovery (PITR)** to any second
- Retention: **1-35 days**
- Free storage = 100% of DB size

### **Manual Snapshots:**
- User-triggered
- **Persistent until deleted**
- No PITR capability

### **Storage Auto-Scaling:**
- Monitors free space
- Automatically adds storage when <10% free
- No downtime during scaling

---

## **📊 MODERN MONITORING**

### **Monitoring Stack:**
1. **CloudWatch Basic** (free): CPU, memory, storage metrics
2. **Enhanced Monitoring** ($): OS-level metrics (1-60s granularity)
3. **Performance Insights** ($): Query performance, wait events
4. **Database Insights** (new unified view):
   - **Standard**: 7 days history (free)
   - **Advanced**: 15 months history + fleet monitoring (paid)

### **Log Exports to CloudWatch:**
✅ Audit log | ✅ Error log | ✅ General log  
✅ IAM auth error log | ✅ Slow query log

---

## **🔧 MAINTENANCE & OPERATIONS**

### **Maintenance Window:**
- Weekly 30-minute window
- Automatic **minor version upgrades**
- Can defer up to 34 days
- May cause brief connectivity loss

### **Connectivity Options:**
- **Connect to EC2 compute resource** (new automatic setup)
- **Public access** (avoid in production)
- **VPC security groups** (stateful firewall)
- **Subnet groups** control which AZs are used

### **Important Settings:**
- **Deletion protection** (ALWAYS enable for production)
- **Idle connection timeout** (1 min to 8 hours)
- **Parameter groups** (engine-specific settings)
- **Option groups** (engine features/extensions)

---

## **💰 COST STRUCTURE**

### **Pricing Components:**
```
Monthly Cost =
  Instance Hours × Instance Price
+ Storage GB × Storage Price
+ IOPS × IOPS Price (if provisioned)
+ Backup Storage (beyond free tier)
+ Data Transfer
+ Additional Features (Proxy, PI, etc.)
```

### **Free Tier (12 months):**
- 750 hours of db.t2.micro/db.t3.micro
- 20 GB storage
- 20 GB backup storage
- 10 million IOs

### **Cost Optimization:**
- Use **Graviton instances** (20% savings)
- **Schedule dev instances** (stop at night)
- Use **gp3 over io1/io2** for non-critical
- **Clean up old snapshots**
- Enable **storage auto-scaling**

---

## **🎯 USE CASES & RECOMMENDATIONS**

### **Best For:**
✅ Traditional enterprise workloads  
✅ Familiar engines (Oracle, SQL Server)  
✅ Predictable workloads  
✅ Lift-and-shift migrations  
✅ When you need specific RDBMS features

### **Not Best For:**
❌ Need massive write scaling  
❌ Unpredictable workloads (use Aurora)  
❌ Need automatic read load balancing  
❌ Require cross-region auto-failover

### **Quick Deployment Guide:**
```yaml
Production:
• Deployment: Multi-AZ DB Cluster (if MySQL)
• Instance: db.m7g.large (Graviton3)
• Storage: io2 with provisioned IOPS
• Security: Secrets Manager + IAM auth
• Protection: Deletion protection ON

Development:
• Deployment: Single-AZ
• Instance: db.t4g.micro (burstable)
• Storage: gp3 (auto-scaling)
• Auto-stop: Enable for cost savings
```

---

## **⚠️ KEY LIMITATIONS**

### **General Limitations:**
- **No root/OS access**
- **Storage scaling up only**
- **Max 5 read replicas**
- **Single writer instance**
- **VPC cannot be changed after creation**

### **Feature-Specific:**
- **Multi-AZ DB Cluster**: MySQL 8.0.28+ only
- **Optimized Writes**: Limited instance families/regions
- **Cross-region replicas**: Manual promotion only

---

## **📋 BEST PRACTICES CHECKLIST**

### **Always Do:**
```sql
1. Use Multi-AZ for production
2. Enable deletion protection
3. Use Secrets Manager for credentials
4. Enable Performance Insights
5. Test backup restores regularly
6. Monitor CloudWatch metrics
7. Use RDS Proxy for serverless apps
```

### **Never Do:**
```sql
1. Expose RDS publicly
2. Use default security groups
3. Forget to test failovers
4. Ignore storage auto-scaling
5. Skip credential rotation
```

---

## **🚀 QUICK REFERENCE TABLE**

| Aspect | RDS Standard | Notes |
|--------|--------------|-------|
| **Max Storage** | 128 TiB | io2 supports 256K IOPS |
| **Max Read Replicas** | 5 | Each has separate storage |
| **Failover Time** | 60-120s (Multi-AZ)<br><35s (Multi-AZ Cluster) | |
| **Write Scaling** | ❌ No | Single writer only |
| **Read Scaling** | ✅ Yes (manual) | No auto-load balancer |
| **Backup Retention** | 35 days | Enables PITR |
| **Engine Support** | MySQL, PostgreSQL, Oracle, SQL Server, MariaDB | Multi-AZ Cluster: MySQL only |
| **Modern Features** | Optimized Writes, Database Insights, Secrets Manager integration | |

---

## **🎓 FINAL TAKEAWAY**

**RDS is perfect when:**
- You want a managed RDBMS with minimal ops overhead
- You need traditional database engines
- Your workload fits vertical scaling
- You're migrating existing databases

**Modern RDS adds:**
- Better performance (Optimized Writes, Graviton3)
- Enhanced security (Secrets Manager, IAM auth)
- Improved monitoring (Database Insights)
- New deployment options (Multi-AZ DB Cluster)
- Better serverless integration (RDS Proxy)

**Remember:** Choose **Aurora** if you need horizontal scaling, automatic read load balancing, or higher performance at scale. Choose **RDS** for specific engine needs, predictable workloads, or lift-and-shift migrations.
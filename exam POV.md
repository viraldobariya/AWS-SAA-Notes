# AWS Databases – SAA-C03 Exam-Oriented Summary

> Concise, sorted, and architecture-focused notes for **AWS Certified Solutions Architect – Associate (SAA-C03)**

---

## 1. Choosing the Right Database (Exam Framework)

AWS exam questions always describe **requirements first**.  
Choose the database that best matches them.

### Key Decision Questions
- Is the workload **read-heavy, write-heavy, or balanced**?
- Does traffic **scale predictably or fluctuate**?
- **Latency requirements**? (microseconds, milliseconds)
- **Data model**? (relational, key-value, document, graph)
- **Queries**? SQL? joins? analytics?
- Is the database the **source of truth**?
- **Schema**? Fixed or flexible?
- **Cost / licensing** concerns?

---

## 2. Relational Databases (OLTP – SQL)

### Amazon RDS
**Managed engines:** MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, DB2

#### Key Exam Points
- Managed backups with **PITR up to 35 days**
- **Multi-AZ = High Availability (failover)**, NOT performance
- **Read Replicas = read scaling**
- Storage auto-scales (instance does NOT)
- IAM authentication + Secrets Manager supported

#### Use Cases
- Transactions
- SQL queries
- Joins
- Structured schema

**Exam Trap:**  
❌ Multi-AZ improves performance → **FALSE**

---

### Amazon Aurora (PostgreSQL / MySQL compatible)

#### Why Aurora Is Preferred
- Cloud-native DB
- **6 copies across 3 AZ**
- Storage auto-scales up to **128 TB**
- Faster than standard RDS
- Writer endpoint + Reader endpoint

#### Important Aurora Features (Highly Tested)
- **Aurora Serverless** – unpredictable/intermittent workloads
- **Aurora Global Database** – multi-region reads, < 1s replication
- **Aurora Cloning** – fast test/dev environments
- **Aurora Machine Learning** – integrates with SageMaker & Comprehend

#### Use Cases
- Same as RDS
- Higher performance
- Less operational overhead

---

## 3. In-Memory Databases (Caching)

### Amazon ElastiCache (Redis / Memcached)

#### Key Points
- In-memory → sub-millisecond latency
- Redis supports:
  - Replication
  - Persistence
  - Multi-AZ
- Requires application code changes

#### Use Cases
- Session storage
- Caching DB queries
- Leaderboards

**Exam Trap:**  
❌ Not a source of truth  
❌ No SQL, no joins

---

## 4. Serverless NoSQL Database

### Amazon DynamoDB

#### Core Exam Facts
- Fully serverless
- Single-digit millisecond latency
- Capacity modes:
  - On-Demand
  - Provisioned + Auto Scaling
- **Global Tables** = active-active multi-region
- **TTL supported**
- **DynamoDB Streams** → Lambda / Kinesis

#### Add-ons
- **DAX** → microsecond read latency
- PITR up to 35 days
- Export/import to S3 without consuming RCUs/WCUs

#### Use Cases
- Massive scale applications
- Flexible schema
- Serverless architectures

**Exam Trap:**  
❌ No joins  
❌ Max item size: 400 KB

---

## 5. Object Storage (Not a Traditional DB)

### Amazon S3

#### Key Points
- Object-based key/value store
- Max object size: **5 TB**
- Extremely durable (11 9s)
- Storage tiers + lifecycle policies

#### Use Cases
- Static assets
- Backups
- Big files

**Exam Trap:**  
❌ Frequent small reads → DynamoDB / ElastiCache preferred

---

## 6. Analytics Databases (OLAP)

### Amazon Redshift
- SQL-based data warehouse
- Columnar storage
- Used for analytics, NOT transactions

### Amazon Athena
- Serverless SQL on S3
- No infrastructure management
- Pay per query

---

## 7. Document Database

### Amazon DocumentDB (MongoDB-compatible)

#### Key Points
- JSON document storage
- MongoDB API compatible
- Storage auto-scales (10 GB increments)
- Replication across 3 AZ

#### Use Cases
- MongoDB workloads
- Flexible schema applications

---

## 8. Graph Database

### Amazon Neptune

#### Use Cases
- Social networks
- Fraud detection
- Recommendation engines
- Knowledge graphs

#### Neptune Streams (Exam Favorite)
- Ordered change log
- Near real-time data access
- Sync data with S3, OpenSearch, Lambda

---

## 9. Wide-Column Database

### Amazon Keyspaces (for Apache Cassandra)

#### Key Points
- Cassandra Query Language (CQL)
- Serverless & scalable
- Multi-AZ replication
- Optimized for time-series & IoT data

---

## 10. Ledger Database

### Amazon QLDB (Quantum Ledger Database)

#### Exam Keywords
- Immutable
- Cryptographically verifiable
- Centralized (not blockchain)

#### Use Cases
- Financial records
- Audit trails
- Compliance systems

---

## 11. Time-Series Database

### Amazon Timestream

#### Key Points
- Serverless time-series DB
- Automatic tiering (memory → disk)
- Built-in time analytics
- Much cheaper than relational DBs

#### Use Cases
- IoT data
- Metrics
- Monitoring
- Real-time analytics

---

## 🔥 SAA-C03 Quick Decision Table

| Requirement | AWS Service |
|------------|------------|
| SQL + joins + transactions | RDS / Aurora |
| Serverless key-value | DynamoDB |
| Microsecond caching | ElastiCache |
| Analytics on S3 | Athena |
| Data warehouse | Redshift |
| Large object storage | S3 |
| Graph relationships | Neptune |
| Immutable ledger | QLDB |
| Time-series data | Timestream |
| MongoDB workloads | DocumentDB |

---
<img width="1024" height="559" alt="image" src="https://github.com/user-attachments/assets/5c25001e-0e25-4191-845b-5572560d1605" />

---
## Final Exam Tip
> **Never choose a database by name — choose it by REQUIREMENT.**

---

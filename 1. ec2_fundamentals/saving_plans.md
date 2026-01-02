# **AWS Savings Plans Comprehensive Guide**

## **🧠 First: What Are Savings Plans?**
Savings Plans are a flexible alternative to RIs. You commit to spending a fixed $/hour amount (e.g., $5/hr) for 1 or 3 years, and AWS gives up to 72% discount on usage that matches.

You pay the committed discounted rate, even if you don't use it fully.

> **Important:** If you switch to less expensive instance types than planned, you might end up paying close to on-demand rates for that usage since you're committed to a minimum spend.

---

## **🔹 1. Compute Savings Plan — Most Flexible**
✅ **Apply to any compute usage:** EC2, Fargate, Lambda
✅ **Works across all instance types, sizes, families, regions, OS**
✅ **Switch freely between instance types/regions**
❌ **Slightly lower discount (~66% max)**

### **💡 Use if:**
You want maximum flexibility — your infrastructure might change in future.

**Example Commitment:**
```bash
# Commit to $5/hour for 3 years
aws savingsplans create-savings-plan \
  --savings-plan-offering-id "compute-sp-offering" \
  --commitment "5" \
  --currency-code "USD" \
  --plan-type "ComputeSavingsPlans"
```

**What it covers:**
- ✅ EC2 instances (any type, any region)
- ✅ AWS Lambda (any memory, any region)
- ✅ AWS Fargate (any CPU/memory, any region)
- ✅ Any future AWS compute service

---

## **🔸 2. EC2 Instance Savings Plan — Higher Discount, Less Flexible**
✅ **Higher discount (~72%)**
✅ **You choose:** Instance Family + Region (e.g., m5 in us-east-1)
✅ **You can switch size** (e.g., m5.large ↔ m5.4xlarge), OS, tenancy
❌ **Locked to that family and region**
❌ **Doesn't apply to Fargate/Lambda**

### **💡 Use if:**
You run EC2 only, and know which family and region you'll use for 1–3 years.

**Example Commitment:**
```bash
# Commit to $3/hour for m5 family in us-east-1
aws savingsplans create-savings-plan \
  --savings-plan-offering-id "ec2-sp-offering" \
  --commitment "3" \
  --currency-code "USD" \
  --plan-type "EC2InstanceSavingsPlans" \
  --payment-option "All Upfront"
```

---

## **🔹 3. Database Savings Plans — For Managed Databases**

### **✅ Coverage:**
- **RDS** (MySQL, PostgreSQL, Oracle, SQL Server, MariaDB)
- **Aurora** (MySQL/PostgreSQL compatible)
- **DynamoDB** (provisioned capacity)
- **ElastiCache** (Redis/Memcached)
- **Amazon DocumentDB**
- **Neptune**
- **Other managed database services**

### **✅ Flexibility:**
- Works **across all regions**
- Applies to **any instance family and size**
- Covers **different database engines**
- Supports **different deployment options** (Single-AZ, Multi-AZ)

### **❌ Limitations:**
- Only applies to **database services**
- Doesn't cover compute or ML services
- Slightly lower discount than EC2-specific plans

### **💡 Use if:**
You have significant database workloads that are stable but might change engine types or deployment patterns.

**Example Scenario:**
```yaml
Current: RDS MySQL m5.large in us-east-1
Future Plan: Might migrate to Aurora PostgreSQL c6g.xlarge

With Database Savings Plan:
- Commitment: $2/hour for 3 years
- Covers: Both MySQL and PostgreSQL
- Covers: Both m5 and c6g families
- Covers: Both RDS and Aurora
- Covers: Any region
```

**Cost Comparison:**
```
On-Demand Database: $0.348/hour
Database Savings Plan: $0.139/hour (60% discount)
Covers: MySQL → PostgreSQL, m5 → c6g, us-east-1 → us-west-2
```

---

## **🔸 4. SageMaker Savings Plans — For Machine Learning Workloads**

### **✅ Coverage:**
- **SageMaker Studio**
- **SageMaker Training Jobs**
- **SageMaker Inference Endpoints**
- **SageMaker Processing Jobs**
- **SageMaker Notebook Instances**
- **SageMaker Batch Transform**

### **✅ Flexibility:**
- Works **across all regions**
- Applies to **any instance family** (ml.m5, ml.c5, ml.p3, etc.)
- Covers **different SageMaker components**
- Applies to **both CPU and GPU instances**

### **❌ Limitations:**
- Only applies to **SageMaker services**
- Doesn't cover EC2 or database services
- ML instance families change rapidly

### **💡 Use if:**
You have predictable ML workloads and want to optimize SageMaker costs.

**Example Scenario:**
```yaml
ML Pipeline:
- Data Processing: ml.m5.xlarge (2 hours/day)
- Model Training: ml.p3.2xlarge (4 hours/day)  
- Inference: ml.c5.large (24/7)
- Total: ~$25/day on-demand

SageMaker Savings Plan:
- Commitment: $0.50/hour = $12/day
- Savings: ~52% discount
- Flexibility: Can shift between m5, p3, c5 instances
```

**Best For:**
- Companies with dedicated ML teams
- Production ML pipelines
- Regular model retraining schedules
- Stable inference workloads

---

## **📊 Comparison Table**

| Feature | Compute Savings Plan | EC2 Instance Savings Plan | Database Savings Plans | SageMaker Savings Plans |
|---------|---------------------|---------------------------|------------------------|-------------------------|
| **Max Discount** | ~66% | ~72% | ~65% | ~64% |
| **Flexibility** | Maximum | Moderate | High | High |
| **Applies To** | EC2, Lambda, Fargate | EC2 only | RDS, Aurora, DynamoDB, etc. | SageMaker only |
| **Region Flexibility** | Any region | Fixed region | Any region | Any region |
| **Instance Flexibility** | Any family/type | Fixed family | Any family | Any ML family |
| **Best For** | Dynamic mixed workloads | Stable EC2 infra | Database workloads | ML workloads |
| **Commitment Unit** | $/hour spend | $/hour spend | $/hour spend | $/hour spend |

---

## **🔑 Key Differences Explained**

### **1. Commitment Model:**
```
All Savings Plans: You commit to a $/hour spend
RI Alternative: You commit to specific instance configurations
```

### **2. Under-Usage Penalty:**
```yaml
Scenario: Commit to $5/hour, but only use $3/hour of covered services
Result: Still pay $5/hour (waste $2/hour)
Solution: Right-size commitment or use flexible plan
```

### **3. Coverage Priority:**
```
AWS applies Savings Plans in this order:
1. EC2 Instance Savings Plans (most specific)
2. Compute Savings Plans
3. Database Savings Plans
4. SageMaker Savings Plans
5. Any remaining usage → On-Demand rates
```

### **4. Exchange/Cancellation:**
```bash
# Cannot cancel Savings Plans (like Standard RI)
# Can modify: Increase commitment anytime
# Cannot decrease commitment before term ends
# No marketplace for selling (unlike RIs)
```

---

## **💰 Cost Optimization Strategies**

### **Strategy 1: Layered Approach**
```yaml
For Enterprise with mixed workloads:
- EC2 Instance Savings Plans: 40% (for stable EC2)
- Compute Savings Plans: 30% (for flexible compute)
- Database Savings Plans: 20% (for databases)
- SageMaker Savings Plans: 10% (for ML)
- Remaining: On-Demand or Spot
```

### **Strategy 2: Start Small, Scale Up**
```bash
# Month 1: Start with conservative commitment
aws savingsplans create-savings-plan \
  --commitment "1" \
  --term "ONE_YEAR"

# Month 3: Analyze usage, increase if needed
aws savingsplans describe-savings-plans-utilization
# If utilization > 80%, consider increasing commitment
```

### **Strategy 3: Hybrid with RIs**
```yaml
Perfect combination:
- Zonal RIs: For mission-critical capacity-guaranteed workloads
- Savings Plans: For flexible, variable workloads
- Spot Instances: For fault-tolerant, interruptible workloads
- On-Demand: For unpredictable spikes
```

---

## **🎓 Summary**

### **Compute Savings Plans:**
**Best for:** Organizations with dynamic, evolving infrastructure that uses multiple compute services.

### **EC2 Instance Savings Plans:**
**Best for:** Companies with stable EC2 workloads in specific families/regions wanting maximum discounts.

### **Database Savings Plans:**
**Best for:** Enterprises with significant database workloads across multiple engines and regions.

### **SageMaker Savings Plans:**
**Best for:** Data science teams with predictable ML workloads using SageMaker.

### **General Rule:**
```
More Flexibility = Slightly Lower Discount
More Specificity = Higher Discount
```

**Remember:** Savings Plans are about committing to **spend**, not to **specific resources**. This makes them ideal for organizations that want cost savings without locking into specific instance configurations.
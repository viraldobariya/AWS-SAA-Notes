# **AWS Reserved Instances Deep Dive**

## **🔹 Standard Reserved Instance (Standard RI)**

### **✅ Advantages:**
**Maximum Discounts (Up to 72%):**
- **1-year** term: ~40-45% discount
- **3-year** term: ~60-72% discount
- **All Upfront**: Highest discount (pay once)
- **Partial Upfront**: Medium discount + monthly
- **No Upfront**: Lower discount, monthly payments

**Marketplace Flexibility:**
```bash
# Sell your unused RI on Marketplace
aws ec2 describe-reserved-instances \
  --reserved-instances-ids ri-12345678

aws ec2 create-reserved-instances-listing \
  --reserved-instances-id ri-12345678 \
  --instance-count 1 \
  --price-schedules '[
    {
      "Term": 365,
      "Price": 0.05,
      "CurrencyCode": "USD"
    }
  ]'
```

**Capacity Reservation (Zonal RIs only):**
- Guarantees capacity in specific AZ
- Critical for production workloads
- Only with **Zonal RIs** (not Regional RIs)

### **❌ Limitations:**
**No Flexibility:**
- ❌ **Cannot change**: Instance type, family, OS, tenancy, region
- ❌ **Cannot merge/split**: 4x m5.large ≠ 1x m5.2xlarge
- ❌ **Cannot cancel**: Committed for full term (1 or 3 years)
- ❌ **Cannot modify scope**: Regional ↔ Zonal

**Scope Options:**
- **Regional**: Can be used in any AZ within region (no capacity reservation)
- **Zonal**: Locked to specific AZ (capacity reservation included)

### **💡 When to Use Standard RI:**
```yaml
Perfect for:
- Production databases (RDS/EC2)
- Static application servers
- VPN/NAT gateways
- Steady-state workloads
- Known instance types for 1-3 years

Example:
- 10x m5.2xlarge running 24/7 for 3 years
- Windows Server with SQL Enterprise
- SAP HANA production instances
```

---

## **🔸 Convertible Reserved Instance (Convertible RI)**

### **✅ Advantages:**
**Maximum Flexibility:**
- ✅ **Can change**: Instance type, family, size, OS, tenancy
- ✅ **Can change scope**: Regional ↔ Zonal
- ✅ **Can exchange**: Multiple times during term

**Exchange Rules:**
1. **Equal or higher price requirement:**
   ```bash
   # Exchange example:
   Current: m5.large Linux ($0.096/hour)
   New: m5.xlarge Linux ($0.192/hour) ✅ Allowed
   New: t3.medium Linux ($0.0416/hour) ❌ Not allowed
   ```

2. **Term remains same:**
   - 1-year RI → New 1-year term
   - 3-year RI → New 3-year term

3. **No marketplace sale:**
   - Cannot sell Convertible RIs
   - Must exchange through AWS

### **⚠️ Trade-offs:**
**Lower Discounts (Up to 54%):**
- **1-year**: ~31-40% discount
- **3-year**: ~45-54% discount
- Typically ~15% less savings than Standard RIs

**No Capacity Reservation:**
- ❌ Cannot reserve capacity
- ❌ Cannot guarantee AZ availability

### **💡 When to Use Convertible RI:**
```yaml
Perfect for:
- Development/Test environments
- Migrating workloads between instance families
- Evolving architecture (microservices, containers)
- Uncertain future requirements
- Organizations with changing tech stack

Example:
- Startup planning to move from m5 to c5 instances
- Company migrating from Linux to Windows
- Cloud migration in progress
- Containerization project underway
```

---

## **📊 Detailed Comparison Matrix**

| Feature | Standard RI | Convertible RI |
|---------|------------|----------------|
| **Max Discount** | Up to 72% | Up to 54% |
| **Instance Flexibility** | ❌ Fixed | ✅ Can change type/family/size |
| **OS Flexibility** | ❌ Fixed | ✅ Can change OS |
| **Tenancy Flexibility** | ❌ Fixed | ✅ Can change (default/dedicated) |
| **Scope Flexibility** | ❌ Fixed | ✅ Regional ↔ Zonal |
| **Marketplace Sale** | ✅ Yes | ❌ No |
| **Exchange** | ❌ No | ✅ Multiple times |
| **Capacity Reservation** | ✅ Zonal RIs only | ❌ No |
| **Term Flexibility** | ❌ Fixed | ❌ Fixed (but can exchange) |
| **Cancellation** | ❌ No (except All Upfront) | ❌ No |
| **Upfront Payment** | All/Partial/No | All/Partial/No |

---

## **💰 Cost Analysis Example**

### **Scenario: m5.2xlarge Linux in us-east-1**

**On-Demand:** $0.384/hour = **$3,363.84/year**

**Standard RI (3-year All Upfront):**
- Effective hourly: $0.109/hour
- **Savings:** 71.6%
- **Total 3-year cost:** $2,864.52
- **Annual equivalent:** $954.84/year

**Convertible RI (3-year All Upfront):**
- Effective hourly: $0.176/hour
- **Savings:** 54.2%
- **Total 3-year cost:** $4,613.28
- **Annual equivalent:** $1,537.76/year

**Annual Difference:**
```
Standard RI: $954.84/year
Convertible RI: $1,537.76/year
Price for Flexibility: +$582.92/year (+61%)
```

---

## **🔒 Capacity Reservation Deep Dive**

### **What is Capacity Reservation?**
Capacity reservation is a **physical guarantee** that AWS will have the resources available for you in a specific Availability Zone.

### **Key Characteristics:**

#### **1. Physical vs Financial Reservation:**
```
Zonal RI = Financial Discount + Physical Capacity Guarantee
Regional RI = Financial Discount Only (No Capacity Guarantee)
```

#### **2. How It Works Physically:**
```
AWS Data Center in us-east-1a:
Physical Servers:
  [Your][Your][Your][Open][Open][Other]  ← Your 3 reserved slots
  [Your][Your][Other][Other][Open][Open]  ← Your 2 reserved slots
  
Total: 5 physical capacity slots reserved
These slots CANNOT be allocated to other customers
Even if you run 0 instances, slots remain reserved
```

#### **3. Cost Implications:**
- **Zonal RI with capacity**: Pay for reserved slots whether used or not
- **Standalone capacity reservation**: Pay on-demand rate for reserved capacity
- **Regional RI without capacity**: Only pay when instances run

### **Capacity Reservation Scenarios:**

#### **Scenario 1: Mission-Critical Database**
```yaml
Requirement: Must guarantee 10x r5.2xlarge in us-east-1a
Risk: "Insufficient capacity" = $1M/hour business loss

Solution: Zonal RI for 10x r5.2xlarge
- Cost: 60% discount + capacity guarantee
- Physical: 10 slots reserved in us-east-1a
- Peace of mind: Always able to launch/replace instances
```

#### **Scenario 2: Black Friday Peak**
```yaml
Normal: 100 instances
Peak: Need 300 instances for 48 hours

Option A: Hope for capacity
  Risk: Website crashes during peak = PR disaster
  
Option B: Zonal RI for 300 instances
  Cost: Pay for 300 slots year-round = $XXX,XXX waste

Option C: Hybrid
  - Zonal RI: 100 instances (steady state)
  - Standalone capacity reservation: 200 instances (for 1 week)
  - Cost: Pay for 200 slots only when needed
```

### **Standalone Capacity Reservation (Without RI):**

```bash
# Reserve capacity WITHOUT RI discount
aws ec2 create-capacity-reservation \
  --instance-type m5.2xlarge \
  --instance-platform Linux/UNIX \
  --availability-zone us-east-1a \
  --instance-count 10 \
  --end-date-type unlimited

# Cancel anytime (no commitment)
aws ec2 cancel-capacity-reservation \
  --capacity-reservation-id cr-1234567890abcdef0
```

**Standalone vs Zonal RI Capacity:**

| Feature | Standalone Capacity Reservation | Zonal RI Capacity |
|---------|--------------------------------|-------------------|
| **Cost** | Full on-demand rate | 60-72% discount |
| **Time Commitment** | Cancel anytime | 1 or 3 years |
| **Discount** | 0% | 60-72% |
| **Best For** | Short-term needs | Long-term steady workloads |
| **Financial Risk** | Low (cancel anytime) | High (locked term) |

---

## **🔄 Exchange Process Deep Dive**

### **Convertible RI Exchange Rules:**

```bash
# Check exchange eligibility
aws ec2 get-reserved-instances-exchange-quote \
  --reserved-instance-ids ri-12345678 \
  --target-configurations '[
    {
      "InstanceType": "m5.xlarge",
      "Platform": "Linux/UNIX",
      "Scope": "Region",
      "InstanceCount": 1
    }
  ]'
```

**Exchange Constraints:**
1. **Total monthly payment cannot decrease**
2. **Remaining term stays same**
3. **Minimum 1-year term for exchanged RI**
4. **Can exchange multiple times per term**

**Example Exchange Path:**
```
Year 1: 10x m5.large (development)
Year 2: Exchange → 5x m5.xlarge + 2x c5.2xlarge (production)
Year 3: Exchange → 4x m5.4xlarge + 1x r5.xlarge (optimized)
```

---

## **🎯 Strategic Recommendations**

### **Use Standard RI When:**
1. **Workload is predictable** for 1-3 years
2. **Instance type won't change** (legacy applications)
3. **Need capacity reservation** in specific AZ
4. **Maximize savings** is primary goal
5. **Organization has stable architecture**

### **Use Convertible RI When:**
1. **Cloud migration in progress**
2. **Technology stack evolving**
3. **Uncertain about future needs**
4. **Mixed development/production use**
5. **Planning instance type upgrades**

---

### **Container/K8s Impact:**
- **Traditional VMs**: Standard RIs work well
- **Containers**: Consider Savings Plans (more flexible)
- **Serverless**: No RIs needed

---

## **🎓 Summary Decision Tree**

```
Start → Is workload predictable for 1-3 years?
    ├─ Yes → Need AZ capacity guarantee?
    │   ├─ Yes → Zonal Standard RI
    │   └─ No → Regional Standard RI
    │
    └─ No → Might change instance/OS/region?
        ├─ Yes → Convertible RI
        └─ No → Consider Savings Plans or On-Demand
```

**Remember:** The 15-20% premium for Convertible RIs is insurance against change. In dynamic environments, this can be worth every penny!
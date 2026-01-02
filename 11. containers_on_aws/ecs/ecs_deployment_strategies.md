## **Deployment Strategies in ECS** (How to update your app without downtime)

Think of it like **upgrading an airplane while it's flying** 🛫→🛬

---

## **1. Rolling Update** (Default - Safest)
**What**: Replace tasks **one by one**  
**Analogy**: Changing tires on a moving car, one wheel at a time

### **Example: Web App with 4 Tasks**
```
Before: [Task-v1] [Task-v1] [Task-v1] [Task-v1] ← All version 1
Step 1: [Task-v2] [Task-v1] [Task-v1] [Task-v1] ← 1st updated
Step 2: [Task-v2] [Task-v2] [Task-v1] [Task-v1] ← 2nd updated  
Step 3: [Task-v2] [Task-v2] [Task-v2] [Task-v1] ← 3rd updated
Step 4: [Task-v2] [Task-v2] [Task-v2] [Task-v2] ← All version 2
```

**Real scenario**: Updating your blog website
- Some users see old version, some see new version temporarily
- If new version has bug, only 25% of traffic affected at first

---

## **2. Blue/Green Deployment** (Zero Downtime - Expensive)
**What**: Launch **complete parallel environment**, switch traffic all at once  
**Analogy**: Building a new parallel runway, moving all planes at once

### **Example: E-commerce Site**
```
BLUE (Current - Live): [v1] [v1] [v1] [v1] ← Customers shopping here
GREEN (New - Idle):    [v2] [v2] [v2] [v2] ← Deployed but no traffic

✅ Test GREEN environment fully
✅ Run automated tests
✅ Flip load balancer → All traffic goes to GREEN
BLUE (Old - Idle):     [v1] [v1] [v1] [v1] ← Keep for rollback
GREEN (New - Live):    [v2] [v2] [v2] [v2] ← All customers now here
```

**Cost**: Double resources during deployment  
**Risk**: Very low (easy rollback)  
**Real use**: Banking apps, critical systems where any downtime = lost money

---

## **3. Canary Deployment** (Test with Real Users)
**What**: Send **small percentage** of traffic to new version first  
**Analogy**: Giving new food to 10% of restaurant customers first

### **Example: Mobile Game Backend**
```
Step 1: Launch new version alongside old
[Task-v1] [Task-v1] [Task-v1] [Task-v2] ← 25% traffic to v2

Step 2: Monitor for 1 hour
✅ Check error rates: v2 has 0.1% errors (good!)
✅ Check performance: v2 is 20% faster (great!)
✅ Check user feedback: No complaints

Step 3: Send 100% traffic to v2
[Task-v2] [Task-v2] [Task-v2] [Task-v2] ← All users now on new version
```

**Real scenario**: Facebook testing new feature with 1% of users first

---

## **4. Linear Deployment** (Gradual & Predictable)
**What**: Shift traffic **gradually over time** with defined steps  
**Analogy**: Dimming lights slowly from 0% to 100%

### **Example: API Service**
```
Configuration:
- Increase by: 10% every 5 minutes
- Start: 0% traffic to new version

Timeline:
00:00: 10% traffic → v2, 90% → v1  ← Test begins
00:05: 20% traffic → v2, 80% → v1
00:10: 30% traffic → v2, 70% → v1
... every 5 minutes ...
01:00: 100% traffic → v2, 0% → v1  ← Complete
```

**Use case**: Very cautious updates, regulatory compliance needs

---

## **"Min Running Tasks %" Explained**
**What**: **Minimum number of healthy tasks** that must remain during deployment

### **Example: Airline Booking System (4 tasks running)**
```yaml
Min Running Tasks: 50%
Max Running Tasks: 200%
```

**Deployment behavior**:
```
Step 1: Scale UP new tasks first
[Task-v1] [Task-v1] [Task-v1] [Task-v1] [Task-v2] ← Add 1 v2 (total 5)

Step 2: Replace old tasks one by one (ensuring 50% healthy = 2 tasks)
[Task-v1] [Task-v1] [Task-v2] [Task-v2] [Task-v2] ← Now 3 v2, 2 v1
[Task-v2] [Task-v2] [Task-v2] [Task-v2] [Task-v2] ← Complete!

⚠️ Never below 2 healthy tasks during process
```

**Without Min Running Tasks** (dangerous):
```
[Task-v1] [Task-v1] [Task-v1] [Task-v1]
↓ Kill all v1 at once
[ ] [ ] [ ] [ ] ← DOWNTIME! No tasks running
↑ Start v2
[Task-v2] [Task-v2] [Task-v2] [Task-v2]
```

---

## **Real Company Examples:**

### **Netflix** (Streaming Service)
- Uses **Canary**: New version to 1% of users in one region
- Monitors: "Are people finishing shows?" "Any buffering?"
- If metrics good → Roll out to all users

### **Amazon** (Shopping Cart)
- Uses **Blue/Green** for checkout process
- Can't risk cart failures during holiday sales
- Parallel environments ensure 100% uptime

### **Startup** (Budget-conscious)
- Uses **Rolling Update** with 25% min healthy
- Cheap, simple, works for most apps
- If bug affects 25% users, okay for MVP

---

## **Which to Choose?**

| Strategy | When to Use | Cost | Risk |
|----------|-------------|------|------|
| **Rolling** | Default, small apps | Low | Medium |
| **Blue/Green** | Critical systems, zero downtime | High | Very Low |
| **Canary** | Testing with real users | Medium | Low |
| **Linear** | Highly regulated (banks, healthcare) | Medium | Very Low |

**Simple rule**:
- **Dev/Test**: Rolling update
- **Production**: Canary (most companies)
- **Money/Banking**: Blue/Green
- **Healthcare/Govt**: Linear

**Min Running Tasks %**: Always set to at least **50%** for production!
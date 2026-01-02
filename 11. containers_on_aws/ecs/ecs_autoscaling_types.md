# **ECS Auto Scaling - Essential Notes**

## **📊 Service Auto Scaling**
- Controls **number of tasks** (containers) in a service
- Uses **AWS Application Auto Scaling** service
- Works for **both Fargate and EC2**

## **📈 Scaling Metrics**
Three metrics available:
1. **CPU Utilization** (% of CPU used)
2. **Memory Utilization** (% of RAM used) 
3. **ALB Request Count Per Target** (requires ALB attachment)

## **🎯 Scaling Policies**
- **Target Tracking**: Maintain metric at target value (simplest)
- **Step Scaling**: Scale in steps based on thresholds
- **Scheduled Scaling**: Predictable time-based scaling

## **⚡ Fargate Advantage**
- **No EC2 instances** to manage
- **Only service scaling** needed
- AWS handles infrastructure automatically
- **Easier setup** (exam favorite)

## **🖥️ EC2 Scaling Complexity**
Two layers needed:

### **1. Service Scaling** (Tasks)
- Scales containers up/down

### **2. Cluster Scaling** (EC2 Instances)

Specify at Cluster level while creating cluster. EC2 has one ASG at Cluster level only.

Two approaches:

#### **Traditional ASG Scaling**
- Scale EC2 based on CPU/RAM metrics
- Doesn't understand ECS task placement

#### **Capacity Provider (Recommended)**
- **Intelligent scaling** - detects when tasks can't be placed
- Auto-adds EC2 when **insufficient capacity**
- **Smarter** than traditional ASG

## **🎯 Key Distinction**
- **Service Scaling** = Task count (Fargate & EC2)
- **Cluster Scaling** = EC2 instances (EC2 only)

## **🚀 Best Practice**
Use **Capacity Providers** for EC2 - they understand ECS needs better than manual ASG scaling.

## **📝 Exam Focus**
- Know the **3 scaling metrics**
- Understand **Fargate advantage** (no EC2 management)
- Remember **Capacity Provider** is recommended for EC2
- Service scaling works for **both launch types**
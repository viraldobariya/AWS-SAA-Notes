## 🧩 Deep Dive: AWS Target Groups and Auto Scaling Groups (tg\_asg.md)

---

### 🔁 Overview

This document unifies everything you need to know about **Target Groups (TG)** and **Auto Scaling Groups (ASG)** in AWS — including how they work together, best practices, and common gotchas.

---

## 🎯 Target Groups (TG)

### ✅ What is a Target Group?

A **Target Group** is a dynamic list of resources (EC2, IP, Lambda) that a **Load Balancer (ALB/NLB)** forwards traffic to.

### 🔗 Role in Load Balancing

* ALB/NLB is connected to a **Target Group**
* Target Group contains **targets** (e.g., EC2 instances)
* LB routes incoming requests to healthy targets inside the TG

### 🎛 Target Types:

| Target Type | Description                             | Use Case                         |
| ----------- | --------------------------------------- | -------------------------------- |
| `instance`  | EC2 instances (by ID)                   | Most common — backed by ASG      |
| `ip`        | IP addresses (on-prem, containers)      | Advanced or hybrid networking    |
| `lambda`    | AWS Lambda functions                    | Event-based / serverless routing |
| `alb`       | Forward traffic from one ALB to another | Rare chaining use case           |

### 🧪 Health Checks

TG performs health checks to each registered target. Unhealthy targets are automatically removed from the pool (unless marked manually).

### 📉 If Target Group Has No Targets

* It still exists
* But **useless** → ALB has no backend to send requests → Clients get 5xx errors

### 🔐 Access

Security groups of EC2s must allow inbound traffic from the load balancer.

---

## 📈 Auto Scaling Group (ASG)

### ✅ What is an ASG?

An **Auto Scaling Group** automatically manages EC2 instance lifecycle based on scaling policies. It ensures the number of running EC2s is within defined `min`, `max`, and `desired` limits.

### 🧱 Components:

* **Launch Template / Launch Configuration**: Defines how to create EC2s
* **Scaling Policies**: Trigger scale-in/out based on metrics (CPU, load, custom)
* **Health Check + Replacement**: Detects failed EC2s and replaces them
* **Lifecycle Hooks**: Run scripts during instance launch/termination (e.g., config, draining)

### ⚙️ Key Use Case:

* Provision a fleet of EC2s that automatically scales
* Attach those EC2s to a target group so a Load Balancer can route traffic

### 💡 ASG *requires* target group **only if** traffic needs to be routed via Load Balancer.

Otherwise, ASG can run in the background doing non-load-balanced jobs too.

#### YouTube Video Processing with ASG Scaling

##### Architecture:

```
┌────────┐    Upload     ┌───────┐    Event     ┌─────────┐
│ Users  │ ────────────> │  S3   │ ──────────>  │  SQS    │
└────────┘    Videos     └───────┘   Message    └─────────┘
                                              │
                                              │ Queue Depth
                                              ↓
                                       ┌─────────────┐
                                       │    ASG      │
                                       │ (Scales by  │
                                       │ Queue Size) │
                                       └─────────────┘
                                              │
                                              │ Launches
                                              ↓
                                       ┌─────────────┐    pull SQS   ┌─────────────┐
                                       │   Workers   │ ────────────> │ S3 Processed│
                                       │ (EC2 Nodes) │  Process&Save │   Videos    │
                                       └─────────────┘               └─────────────┘
```

---

## 🔗 TG & ASG — The Real Relationship

### 🔁 How they connect:

```text
User → ALB → Target Group → EC2s (launched by ASG)
```

### 🔄 What "Attach TG to ASG" means:

* When an ASG is associated with a TG:

  * Every new EC2 launched by ASG is **automatically registered** to the TG
  * Every terminated EC2 is **deregistered** from the TG

### ⚠️ Without ASG:

You must **manually** register EC2s to the TG. It's static, not scalable.

---

## 🧠 Key Design Principles

### ✅ One ASG → One Target Group (Best Practice)

* Simple, isolated scaling and routing
* Easier monitoring and debugging

### ✅ One ASG → Multiple Target Groups (Allowed)

* Reuse same EC2s for different ALBs (e.g., `/api`, `/admin`, `/mobile`)
* Each ALB still forwards traffic to its own TG, but all TGs point to same EC2s
* Must use same port config in all TGs

### ✅ Target Group without ASG

* Technically valid
* But **must manually register EC2s/IPs/Lambdas**
* Not scalable, not fault-tolerant

### ❌ Target Group with 0 targets

* Will return **5xx errors** from ALB
* Health checks fail

### 🔥 Can Multiple ASGs Share One TG?

Technically **yes**, but:

* Difficult to trace traffic origin per ASG
* Potential for overloading
* Health checks interfere
* Usually avoided in production

---

## 📌 Summary Table

| Component        | Mandatory?   | Scalable?        | Common Use Case                |
| ---------------- | ------------ | ---------------- | ------------------------------ |
| Target Group     | ✅ For ALB    | 🚫 Not by itself | Required for ALB routing       |
| ASG (without TG) | ❌ Optional   | ✅ Yes            | Background jobs, async workers |
| TG + ASG         | ✅ Best Combo | ✅ Yes            | Web backend behind ALB         |

---

## ✅ Best Practices Checklist

* [x] Always use ASG for any production EC2 backend
* [x] Let ASG register EC2s into TG
* [x] Never leave TG with 0 targets in production
* [x] Use separate TGs if you need different routing logic (paths, ports)
* [x] Monitor health via TG, scale via ASG

---

## 📌 Final Word

* **Target Group** = dynamic list of backends
* **ASG** = dynamic manager of EC2s
* Together, they create a **self-healing, auto-scaling, load-balanced architecture** in AWS

Use them **together** to unlock the real power of cloud-native infrastructure.

---

*End of tg\_asg.md*

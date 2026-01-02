# **AWS EC2 Dedicated Tenancy Comparison**

## 🧩 Goal: Isolation from Other Tenants

Both **Dedicated Instances** and **Dedicated Hosts** give you EC2 on **hardware not shared with other AWS customers** (unlike default EC2 which is multi-tenant).

But they differ in **control, use cases, licensing, and visibility**.

---

## 🏠 1. Dedicated Instances — "No Sharing, But No Control"

> 🧱 **Physically isolated EC2 instances**, but you don't manage the host.

### ✅ Key Features:

* Your EC2 runs on **hardware not shared** with others.
* You **don't see or control** the physical server.
* AWS places your instances on **isolated hardware**, but **you don't reserve the host**.
* **Charged per instance**, **hourly billing** (no savings with host-level reservations).
* **Good for simple isolation** needs (e.g., security compliance).
* **No affinity control** — AWS manages placement entirely.

### ❌ Limitations:

* No control over **instance placement** (e.g., can't use existing host capacity).
* Not suitable for **Bring Your Own License (BYOL)** for Windows/SQL.
* Less cost-efficient at scale (pay per instance premium vs. whole host).

---

## 🧩 2. Dedicated Host — "Full Physical Server Visibility"

> 🏗️ You **reserve the entire physical server**, get control of placement, and can run multiple EC2s on it.

### ✅ Key Features:

* You **control the host** — can launch instances on specific sockets/cores.
* Enables **BYOL (Bring Your Own License)** for OS/software like Windows Server, SQL Server.
* You can **reuse your licenses** that require per-socket/per-core compliance.
* Visibility into **host-level metrics**, inventory, and usage.
* Charged **per host**, not per instance — often **cheaper** if fully utilized.
* **Affinity control** — keep instances on same physical server after stop/start cycles.
* **Host pooling** — instances can auto-move between YOUR dedicated hosts for HA.

### 💡 Use Cases:

* Software licensing compliance (BYOL).
* Regulatory/compliance requirements needing host-level isolation.
* Full control over **instance placement** (e.g., affinity, anti-affinity).
* Predictable performance from consistent hardware.

---

## 🔍 Key Differences — Side-by-Side

| Feature                    | Dedicated Instance                | Dedicated Host                 |
| -------------------------- | --------------------------------- | ------------------------------ |
| Isolation                  | ✅ Physical isolation              | ✅ Physical isolation           |
| Billing model              | 💵 Per-instance + dedicated fee   | 💵 Per-host + instance fees     |
| BYOL Licensing             | ❌ Not supported                   | ✅ Supported                    |
| Host visibility            | ❌ No                              | ✅ Yes                          |
| Instance placement control | ❌ No                              | ✅ Yes (Affinity, etc.)         |
| Cost efficiency            | ❌ Less (especially underutilized) | ✅ Better if host is fully used |
| Control over cores/sockets | ❌ No                              | ✅ Yes                          |
| Affinity/Tenancy control   | ❌ No                              | ✅ Host/Default options         |
| Host pooling               | ❌ Single placement                | ✅ Multiple hosts pool          |
| Restart flexibility        | ❌ AWS-managed placement           | ✅ Can move between YOUR hosts  |

---

## 💰 Cost Dynamics Deep Dive

### **Dedicated Hosts:**
**You pay ONLY the host rate. Instance charges are ZERO.**

```
CORRECT FORMULA:
Total Cost = Host Hourly Rate ($2.424/hr) + $0 for instances
```

### **Dedicated Instances:**
**You pay instance rate + ~10% premium.**

```
CORRECT FORMULA:
Total Cost = Instance Rate ($0.096) + 10% premium ($0.0096) = $0.1056/hr
```

---

## 🎯 Affinity & Placement Intelligence

### **Dedicated Host Affinity Options:**
- **Affinity=host**: Instance always returns to same physical server (for licensing/compliance)
- **Affinity=default**: Instance can restart on any of YOUR dedicated hosts (for HA/flexibility)
- **HostId specification**: Pin to specific physical server
- **Auto-placement**: AWS picks from your host pool

### **Real Impact:**
| Scenario | Affinity=host | Affinity=default |
|----------|---------------|------------------|
| Stop → Start | Same host | Any YOUR host |
| Host failure | Instance stuck | Moves to another YOUR host |
| Maintenance | Manual move needed | Auto-relocation possible |
| Licensing compliance | ✅ Required | ❌ May violate |

---

## 🚀 When to Choose What?

### **Choose Dedicated Instance When:**
- You need **simple hardware isolation** for compliance
- Don't have **BYOL requirements**
- Running **few instances** (1-3)
- Don't need **placement control**
- OK with **higher per-instance cost**

### **Choose Dedicated Host When:**
- You have **BYOL licenses** (Windows, SQL Server, etc.)
- Need **host-level visibility & control**
- Running **many instances** (4+)
- Require **predictable hardware** (same physical server)
- Want **cost optimization** at scale
- Need **affinity control** for compliance

---

## ✅ Summary

| Type                   | When to Use                                                                                                                                                  |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Dedicated Instance** | You need **hardware isolation**, but **don't care** about licensing or placement. Simple compliance cases. 1-3 instances, simple needs.                      |
| **Dedicated Host**     | You need **licensing control (BYOL)**, **visibility**, **placement control**, **affinity management**, or to **optimize costs** at scale. Best for enterprise-grade needs with 4+ instances. |

### **Pro Tip:**
Start with **Dedicated Instances** for proof-of-concept or small deployments. Migrate to **Dedicated Hosts** when you scale, need BYOL, or require fine-grained placement control. Use **Reserved Dedicated Hosts** for production workloads to maximize savings.
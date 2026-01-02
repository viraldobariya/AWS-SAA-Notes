<!-- Here’s **AWS ElastiCache** in **short but deeply structured** form:

---

## ⚡ AWS ElastiCache — In Short (But In-Depth)

### 🧠 What It Is

* **In-memory caching service** by AWS for ultra-fast data access.
* Supports:

  * **Redis** (feature-rich, pub/sub, TTL, sorted sets, streams, persistence, Lua, etc.)
  * **Memcached** (simple, multi-threaded, no persistence)

---

### 🚀 Why Use It?

* **Sub-millisecond latency** for read/write.
* **Reduces DB load** by caching frequent queries (e.g., session data, product pages).
* Ideal for **real-time applications**: gaming, ad tech, chat apps, dashboards, leaderboards.

---

### 🏗️ Architecture Highlights

| Feature           | Redis                           | Memcached                    |
| ----------------- | ------------------------------- | ---------------------------- |
| Persistence       | ✅ Yes (Snapshots, AOF)          | ❌ No persistence             |
| Replication       | ✅ Yes (Multi-AZ, auto failover) | ❌ No built-in replication    |
| Clustering        | ✅ Yes (sharded with slots)      | ✅ Yes (client-side sharding) |
| Pub/Sub & Streams | ✅ Supported                     | ❌ Not supported              |
| Multi-threaded    | ❌ (single-threaded)             | ✅ (multi-threaded)           |
| Backup & Restore  | ✅ Snapshots, automated backups  | ❌ Not available              |
| Security          | IAM, TLS, VPC, KMS, Auth        | VPC only                     |
| Auto Discovery    | ✅ Yes (for clusters)            | ✅ Yes                        |

---

### 🔄 Use Cases

* ✅ Session storage
* ✅ Caching query/API results
* ✅ Leaderboards / counters / TTL-based data
* ✅ Message queues / pub-sub (Redis)
* ✅ Real-time analytics

---

### 🛠️ Management Features

* **Fully managed** (patching, monitoring, failover handled by AWS)
* Supports **CloudWatch metrics**
* Integrated with **VPC, IAM, KMS, Security Groups**
* **Scaling:** Vertical (instance type), Horizontal (Redis Cluster sharding)

---

### 🧪 Key Limits

* **Redis Max Nodes per Cluster**: 90 (with sharding)
* **Max Data per Node**: \~637 GiB (Redis, varies by instance type)
* **Eviction Policies:** allkeys-lru, volatile-lru, noeviction, etc.

---

Let me know if you want Redis vs Memcached **only** comparison, or ElastiCache deep-dive use cases. -->


# **AWS ElastiCache - Cheat Sheet**

## **1. What is it?**
Managed **Redis** or **Memcached** in AWS. In-memory data store/cache.

## **2. Two Engines:**

### **Redis**
- **Single-node** or **Cluster mode** (sharding)
- Multi-AZ with auto-failover
- **Backups/snapshots** supported
- Data persistence (AOF/RDB)
- Supports complex data types (sets, lists, sorted sets)

### **Memcached**
- **Simple key-value** only
- No persistence (purely cache)
- Multi-node for **horizontal scaling**
- **No backups**, no replication
- Use for: simple caching, session stores

---

## **3. Key Features:**
- **Auto-failover** (Redis only)
- **Read Replicas** (Redis only)
- **Multi-AZ** deployment
- **Backup & Restore** (Redis only)
- **Security:** VPC, IAM, Redis AUTH, TLS encryption

---

## **4. Use Cases:**
- **Database caching** (reduce RDS load)
- **Session stores** (web apps)
- **Real-time analytics**
- **Leaderboards** (Redis sorted sets)
- **Pub/Sub messaging** (Redis)

---

## **5. Limits:**
- **Redis:** Up to 500 nodes per cluster (shards + replicas)
- **Memcached:** Up to 100 nodes
- Max memory: **6.1 TB** (R6gd nodes)
- **No SSH access** to nodes
- Data **fits in RAM** only

---

## **6. Pricing:**
- Pay per **node-hour**
- **No data transfer charges** within AZ
- Backup storage costs extra
- **Reserved Nodes** for discount (1-3 year term)

---

## **7. Key Commands:**
```bash
# Check if working
redis-cli -h <endpoint> ping

# Basic ops
SET key value
GET key
DEL key
```

---

## **8. Gotchas:**
- **Memcached nodes are independent** (no replication)
- **Redis Cluster mode** changes how keys work (hash tags)
- **Backup/restore creates NEW cluster**
- **Vertical scaling requires downtime** (change node type)
- **No VPC peering across regions** - use Global Datastore (Redis only)

---

## **9. When to Choose:**

| Choose **Redis** if: | Choose **Memcached** if: |
|---------------------|-------------------------|
| Need persistence | Pure cache, no persistence |
| Need replication | Simple key-value only |
| Complex data types | Multi-threaded performance |
| Pub/Sub needed |  |

---

**TLDR:** Redis = Swiss Army knife with persistence. Memcached = Simple, fast cache.
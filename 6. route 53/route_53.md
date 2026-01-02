# **DNS & AWS Route 53 - Complete Guide**

## **1. DNS Basics**

### **What is DNS?**
The "phonebook of the internet" - translates `example.com` → `192.0.2.1`

### **DNS Resolution Hierarchy:**
```
Your Computer → Recursive Resolver → Root → TLD → Authoritative → Website
        ↓              ↓              ↓      ↓          ↓           ↓
     Browser      (Google/ISP)      (.)   (.com)   (example.com)   Server
```

### **DNS Components:**
| Component | What it Does | Managed By |
|-----------|-------------|------------|
| **Recursive Resolver** | Does all the DNS lookup work for you | Your ISP, Google (8.8.8.8), CloudFlare (1.1.1.1) |
| **Root Nameservers** | Points to TLD servers | ICANN (13 logical clusters, hundreds of servers) |
| **TLD Servers** | Manages domain extensions (.com, .in, .org) | Registry operators (Verisign for .com, NIXI for .in) |
| **Authoritative Nameservers** | Final authority for a domain's records | Domain owner (you) via Route 53/other providers |

### **Zone File:**
- Text file containing **all DNS records** for a domain
- **Stored on:** Authoritative Nameservers
- **Managed by:** Domain owner
- **Example record:** `www IN A 192.0.2.1`

---

## **2. AWS Route 53 - Complete Overview**

### **What is Route 53?**
AWS managed **DNS + Domain Registrar + Health Check + Traffic Routing** service.

### **Core Capabilities:**
1. **Domain Registration** (buy domains)
2. **DNS Hosting** (hosted zones)
3. **Traffic Routing** (7 routing policies)
4. **Health Checking** (monitor endpoints)
5. **DNSSEC** (DNS security)
6. **Private Hosted Zones** (VPC internal DNS)

---

## **3. Route 53 DNS Records**

| Record Type | Purpose | Special Notes |
|------------|---------|--------------|
| **A** | Domain → IPv4 | Most common |
| **AAAA** | Domain → IPv6 | For IPv6 |
| **CNAME** | Alias to another domain | ❌ **Cannot use at root (@)** |
| **ALIAS (AWS-only)** | Like CNAME but for AWS resources | ✅ **Can use at root (@)** + **free queries** |
| **NS** | Nameservers for domain | Provided by Route 53 when you create hosted zone |
| **MX** | Mail servers | For email (Gmail, Office 365, etc.) |
| **TXT** | Text records | Domain verification, SPF, DKIM |
| **SOA** | Start of Authority | Auto-created, contains admin info |
| **PTR** | Reverse DNS (IP → domain) | For email server reputation |

### **TTL (Time to Live):**
- **Purpose:** How long DNS answers are cached
- **Low TTL (60s):** Fast changes, more DNS queries, higher cost
- **High TTL (86400s):** Slow changes, fewer queries, lower cost

---

## **4. Route 53 Routing Policies (7 Types)**

| Policy | Use Case | Example |
|--------|----------|---------|
| **Simple** | Single resource | Company homepage to one server |
| **Weighted** | Split traffic by % | A/B testing (80% v1, 20% v2) |
| **Failover** | Active-passive DR | Primary in us-east-1, backup in eu-west-1 |
| **Latency-based** | Lowest network delay | Global app with regional deployments |
| **Geolocation** | Country-specific routing | French users → Paris, German users → Frankfurt |
| **Geoproximity** | Location + adjustable bias | Shift 60% traffic to us-east-1, 40% to eu-west-1 |
| **Multivalue** | Multiple healthy IPs randomly | Simple load balancing without ELB |

---

## **5. Health Checks**

### **Features:**
- Monitor endpoints via **HTTP/HTTPS/TCP**
- **Integrated with failover** routing
- Can trigger **CloudWatch alarms**
- **String matching** for response validation
- Monitor **other health checks** (calculated health checks)

### **Pricing:** $0.50/month per health check endpoint

---

## **6. Hosted Zones**

### **Types:**
1. **Public Hosted Zone:** Internet-facing DNS records
2. **Private Hosted Zone:** VPC internal DNS (`.internal`, `.local` domains)

### **Limits:**
- Max 10,000 hosted zones (soft limit)
- Max 10,000 records per hosted zone
- $0.50/month per hosted zone

---

## **7. Domain Registration with Route 53**

### **Features:**
- Register new domains (.com, .in, .org, etc.)
- **Auto-renewal** option
- **WHOIS privacy** (hide your contact info)
- **DNSSEC** signing
- **Transfer existing domains** from other registrars

### **Pricing:** ~$12/year for .com domains

---

## **8. DNS Resolution Flow in Route 53 Context**

**User visits `www.your-app.com`:**
1. User's browser checks **local cache**
2. Asks **ISP resolver** → Checks **Route 53 Recursive Resolver** (if using Route 53 as resolver)
3. Resolver asks **Root servers** → "Where are .com servers?"
4. Root says: "Ask these .com TLD servers"
5. TLD says: "For your-app.com, ask these Route 53 nameservers"
6. **Route 53 Authoritative Server** returns the IP based on routing policy
7. All parties cache the answer based on TTL

---

## **9. Important Limits & Pricing**

| Item | Limit/Cost |
|------|-----------|
| **Domains per account** | 50 (can increase via support) |
| **Hosted zones** | $0.50/month each |
| **DNS queries** | $0.40 per million (first 1B), then $0.20 |
| **Health checks** | $0.50/month per endpoint |
| **Rate limiting** | 5 requests/second per source IP |
| **Max records/zone** | 10,000 |

---

## **10. Pro Tips & Best Practices**

1. **Use ALIAS records for AWS resources** (ELB, CloudFront, S3) - they're free and auto-update
2. **Set appropriate TTLs:**
   - **Development:** 60-300 seconds
   - **Production:** 300-3600 seconds
   - **Static assets:** 86400+ seconds
3. **Enable DNSSEC** for security-critical domains
4. **Use private hosted zones** for internal microservices communication
5. **Health check from multiple regions** for accurate monitoring
6. **Use weighted routing with 0%** to safely test new endpoints
7. **For global apps:** Combine **latency-based + geolocation** routing

---

## **11. Common Use Cases**

1. **Single-page application:** ALIAS record → CloudFront/S3
2. **Global SaaS app:** Latency-based → regional ELBs
3. **Disaster recovery:** Failover routing between regions
4. **Blue/green deployments:** Weighted routing with health checks
5. **Compliance:** Geolocation routing to keep data in-region
6. **Internal services:** Private hosted zones in VPC

---

**Quick Decision Guide:**
- Buying domains? → **Route 53 Registrar**
- Hosting DNS? → **Route 53 Hosted Zones**
- Global traffic routing? → **Latency/Geolocation policies**
- Need failover? → **Failover routing + Health checks**
- Internal service discovery? → **Private Hosted Zones**
# **AWS Network Load Balancer (NLB) - Quick Notes**

## **Overview**
- **Layer 4** load balancer (TCP/UDP/TLS)
- **Extreme performance**, low latency
- Handles **millions of requests/sec**
- Preserves **source IP address**

## **Core Components**

### **1. Target Groups**
- **Types**: Instance, IP, ALB
- Health checks: TCP, HTTP, HTTPS
- **Sticky sessions**: Not supported (stateless)
- **Target deregistration delay**: 0-3600s

### **2. Listeners**
- **Protocols**: TCP, UDP, TLS
- **Port ranges**: Can listen on range of ports
- Each listener forwards to **one target group**

### **3. Target Types**
- **Instance**: EC2 instances (auto-registration)
- **IP**: Private IP addresses (on-prem, peered VPC)
- **ALB**: Forward to Application Load Balancer

## **Key Features**

### **Preserves Source IP**
```yaml
Client IP: 203.0.113.1 → NLB → Target sees: 203.0.113.1
# (No X-Forwarded-For header needed)
```

### **Static IP per AZ**
- Each NLB gets **one Elastic IP per Availability Zone**
- Or use **subnet IPs** (no EIP needed for internal NLB)
- Predictable IPs for firewall rules

### **Zonal Isolation**
- Each AZ has its own NLB node
- Failure in one AZ doesn't affect others
- Cross-zone load balancing **disabled by default** (enable for better distribution)

## **Health Checks**
- Configured at **target group level**
- **Protocols**: TCP, HTTP, HTTPS
- **Unhealthy threshold**: 2-10 consecutive failures
- **Health check port**: Can differ from traffic port

## **TLS Termination**
- **TLS listeners** (not SSL)
- Terminate TLS at NLB
- Uses **SSL policies** (like ALB)
- **SNI support**: Multiple certificates
- **Backend encryption**: Optional re-encryption

## **Monitoring & Logging**
- **Access logs**: VPC Flow Logs (not ALB-style logs)
- **CloudWatch metrics**: Active flow count, processed bytes, new flows
- No request-level logging (L4 only)

## **Limits**
- Max 50 target groups per NLB
- Max 10 certificates per listener (TLS)
- Max port range: 1-100 ports in single listener
- **No HTTP/2** (only TCP/TLS at L4)

## **Pricing**
- **NLCU-based** (Network Load Balancer Capacity Units)
- Factors: New connections, active connections, processed bytes
- **No charge** for:
  - First 15 NLCUs per month
  - Data processing within same AZ

## **Use Cases**

### **1. Extreme Performance**
```yaml
# Gaming servers, financial trading
- UDP/TCP protocols
- Millions of requests/sec
- Microsecond latency
```

### **2. Preserve Client IP**
```yaml
# Security appliances, firewalls
- Client IP preserved end-to-end
- No proxy protocol needed
```

### **3. IP-Based Targeting**
```yaml
# Hybrid architectures
- On-prem servers via Direct Connect
- Peered VPC resources
- Non-AWS endpoints
```

### **4. TCP/UDP Services**
```yaml
# Database, custom protocols
- MySQL, Redis, MQTT
- VoIP, streaming protocols
- Any non-HTTP service
```

## **Best Practices**

### **1. Security**
```yaml
- Use TLS termination for encryption
- Security groups on targets (not NLB)
- Enable deletion protection
- Use private NLB for internal services
```

### **2. Architecture**
```yaml
- Enable cross-zone load balancing
- Use IP targets for non-EC2 backends
- Assign Elastic IPs for public NLB
- Use multiple listeners for different ports
```

### **3. Performance**
```yaml
- Disable TCP slow start if needed
- Adjust deregistration delay
- Monitor NLCU usage
- Use UDP for latency-sensitive apps
```

## **Common Patterns**

### **NLB + ALB Pattern**
```
NLB (Public, TLS:443) → ALB (Private, HTTP:80) → EC2
# Benefits: Static IPs + HTTP routing
```

### **Multi-Port Services**
```
Single NLB with multiple listeners:
- TCP:3306 → MySQL targets
- TCP:6379 → Redis targets  
- TCP:5432 → PostgreSQL targets
```

### **Hybrid Architecture**
```
On-prem clients → NLB (Public IPs) → 
  ├── EC2 targets (AWS)
  ├── On-prem servers (via DX)
  └── Peered VPC resources
```

## **Quick Commands**

```bash
# Create NLB
aws elbv2 create-load-balancer \
  --name my-nlb \
  --type network \
  --scheme internet-facing \
  --subnets subnet-xxx subnet-yyy

# Create target group (IP targets)
aws elbv2 create-target-group \
  --name my-tg \
  --protocol TCP \
  --port 80 \
  --target-type ip \
  --vpc-id vpc-xxx

# Register IP targets
aws elbv2 register-targets \
  --target-group-arn arn:... \
  --targets Id=10.0.1.1,Port=80 Id=10.0.2.1,Port=80

# Create listener
aws elbv2 create-listener \
  --load-balancer-arn arn:... \
  --protocol TCP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=arn:...
```

## **NLB vs ALB Quick Comparison**

| Feature | NLB | ALB |
|---------|-----|-----|
| **Layer** | L4 (TCP/UDP/TLS) | L7 (HTTP/HTTPS) |
| **Source IP** | Preserved | X-Forwarded-For header |
| **Routing** | Port-based | Content-based |
| **Performance** | Millions RPS | Thousands RPS |
| **IP Address** | Static per AZ | Dynamic DNS |
| **Target Types** | Instance, IP, ALB | Instance, IP, Lambda |
| **SSL/TLS** | Terminates TLS | Terminates SSL/TLS |
| **Logging** | Flow Logs | Access Logs |
| **Cost** | NLCU | LCU |

---

**TL;DR**: NLB = Extreme performance + Static IPs + Client IP preservation + TCP/UDP services. Use when you need speed, predictable IPs, or non-HTTP protocols.
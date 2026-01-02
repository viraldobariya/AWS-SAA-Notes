# **AWS Application Load Balancer (ALB) - Quick Notes**

## **Overview**
- **Layer 7** load balancer (HTTP/HTTPS only)
- **Intelligent routing** based on content of request
- Supports **multiple applications** on single ALB

## **Core Components**

### **1. Target Groups**
- Logical grouping of targets (EC2, Lambda, IP addresses)
- Health checks configured at target group level
- **Types**: Instance, IP, Lambda

### **2. Listeners**
- Check for connection requests on **protocol:port**
- **Supported**: Protocols : HTTP, HTTPS , Port : 1 - 65535
- Each listener has **rules** for routing decisions

### **3. Rules**
- Processed in **priority order** (1-50000, lower = higher priority)
- **Conditions**: Host header, Path pattern, HTTP headers, Query strings, Source IP
- **Actions**: Forward to Target Group(s) (weight based routing), Redirect, Fixed Response, Authenticate

## **Key Features**

### **Content-Based Routing**
```yaml
Listener: HTTPS:443
├── Rule: Host=api.example.com → api-target-group
├── Rule: Path=/images/* → static-target-group  
├── Rule: Header X-Mobile=true → mobile-target-group
└── Default: web-target-group
```

### **Host-Based Routing**
- Route traffic to different services based on domain
- Example: `api.example.com` vs `web.example.com` on same ALB

### **Path-Based Routing**
- Route based on URL path
- Example: `/api/*` to backend, `/static/*` to S3 via Lambda

## **Health Checks**
- Configured at **target group level**
- Protocol: HTTP, HTTPS
- Path, port, success codes configurable
- Unhealthy targets removed from rotation

## **SSL/TLS**
- Terminate SSL at ALB (offloads encryption from backend)
- **SSL certificates** from ACM (AWS Certificate Manager)
- Supports multiple certificates via SNI (Server Name Indication)

## **Authentication**
- **Authenticate action** in rules
- Integrates with: Cognito, OIDC, SAML
- Redirect unauthenticated users to IdP

## **Monitoring & Logging**
- **Access logs**: Store in S3
- **CloudWatch metrics**: Request count, latency, error rates
- **Request tracing**: Add X-Amzn-Trace-Id header

## **Limits**
- Max 50 rules per listener
- Max 100 target groups per ALB
- Max 1000 certificates per ALB
- Max 32 target groups per rule

## **Pricing**
- **LCU-based** (Load Balancer Capacity Units)
- Factors: New connections, active connections, processed bytes
- **No charge** for:
  - First 15 LCUs per month (always free tier)
  - AWS PrivateLink for private ALBs

## **Best Practices**

### **1. Security**
```yaml
- Use HTTPS listeners only
- Redirect HTTP → HTTPS
- Use security groups (ALB SG → Target SG)
- Enable deletion protection
```

### **2. Architecture**
```yaml
- Use path-based routing for microservices
- Separate internal/external ALBs
- Use ALB with Auto Scaling Groups
- Enable cross-zone load balancing
```

### **3. Performance**
```yaml
- Enable HTTP/2
- Use keep-alive
- Configure idle timeout (1-400s)
- Use WAF with ALB for protection
```

## **Common Patterns**

### **Microservices**
```
ALB → /service1/* → Service1 Target Group
    → /service2/* → Service2 Target Group
    → /service3/* → Service3 Target Group
```

### **Blue-Green Deployment**
```yaml
Listener Rule:
- Path=/v2/* → NewVersion-TG (10% traffic)
- Default → CurrentVersion-TG (90% traffic)
```

### **A/B Testing**
```yaml
Listener Rule:
- Header X-Experiment=B → VersionB-TG (50%)
- Default → VersionA-TG (50%)
```

## **Integration**
- **ECS**: Dynamic port mapping
- **EKS**: Kubernetes Ingress Controller
- **Lambda**: Serverless applications
- **WAF**: Web Application Firewall
- **Route 53**: DNS routing

## **Quick Commands**
```bash
# Create target group
aws elbv2 create-target-group --name my-tg --protocol HTTP --port 80

# Create listener
aws elbv2 create-listener --load-balancer-arn arn:... --protocol HTTP --port 80

# Add rule
aws elbv2 create-rule \
  --listener-arn arn:aws:elasticloadbalancing:region:account:listener/app/alb-name/... \
  --priority 10 \
  --conditions Field=host-header,Values=api.example.com \
  --actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:region:account:targetgroup/api-tg/...```

---

**TL;DR**: ALB = Smart HTTP routing + SSL termination + host/path routing + integrates with AWS ecosystem.
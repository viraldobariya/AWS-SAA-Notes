# **AWS Gateway Load Balancer (GWLB) - Short Notes**

## **What It Is**
- For **3rd party virtual appliances** (firewalls, IDS, etc.)
- **Transparent traffic inspection** at L3/L4
- Uses **GENEVE encapsulation** (UDP 6081)

## **Key Features**
- **Traffic symmetry**: Same appliance for both directions
- **Preserves client IPs**: Appliances see original packets
- **Auto-scaling**: Automatically registers new appliance instances
- **Static IPs**: Per Availability Zone

## **How It Works**
```
Traffic → GWLB Endpoint → GWLB → Appliance → GWLB → GWLB Endpoint → Destination
        (encapsulate)                    (inspect)                 (decapsulate)
```

## **Use Cases**
- Security appliances (firewalls, IPS)
- Network monitoring tools
- Compliance/audit tools

## **Quick Facts**
- **Port**: UDP 6081 only
- **Targets**: IP addresses only
- **Pricing**: GWLCU-based (capacity units)
- **HA**: Built-in with multi-AZ deployment

## **Deployment Flow**
1. Deploy virtual appliances
2. Create GWLB and target group
3. Create Gateway Load Balancer Endpoint (GWLBE)
4. Update route tables to send traffic to GWLBE

**TL;DR**: GWLB = Traffic inspection gateway for security appliances with GENEVE encapsulation.
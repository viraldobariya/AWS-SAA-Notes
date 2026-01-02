# **AWS RDS Backup - Complete Essentials**

## **🔄 How Backups Actually Work**

### **Incremental Chain with Re-baseling:**
```
Day 1: [FULL SNAPSHOT] ← Initial base
Day 2-6: [Daily Incrementals] ← Only changes
Day 7: [NEW FULL SNAPSHOT] ← Re-baseline!
Day 8-13: [Daily Incrementals] ← Changes since Day 7
Day 14: [NEW FULL SNAPSHOT] ← Re-baseline again!
...
Every ~7 days: AWS takes new full snapshot automatically
```

**Plus:** Transaction logs every 5 minutes

---

## **🚨 THE 35-DAY HARD LIMIT**

### **Sliding Window Deletion:**
```yaml
Today = Day 35: [Day 1 Full | Day 2 Inc | ... | Day 35 Inc]
Tomorrow = Day 36: [Day 2 Inc | ... | Day 36 Inc]
                      ↑
              Day 1 Full AUTO-DELETED 💥
```

**Critical:** Day 1 full = Foundation for entire chain  
**Remove it = Chain broken = Cannot restore anything**

---

## **⚡ Re-baseling Benefits (Every ~7 Days)**

### **Faster Recovery Within 35 Days:**
```
Without re-baseling:
Restore Day 35 = Day 1 full + 34 increments (SLOW)

With re-baseling:
Restore Day 35 = Day 28 full + 7 increments (FAST!)
```

**But:** Still subject to 35-day deletion limit

---

## **📊 Backup Types Quick Compare**

| Feature | Automated Backups | Manual Snapshots |
|---------|------------------|------------------|
| **Type** | Incremental chain | Independent full |
| **PITR** | ✅ Yes (within 35d) | ❌ No |
| **Retention** | 1-35 days (HARD) | Unlimited |
| **Auto-delete** | ✅ After 35d | ❌ Manual only |
| **Best For** | Daily ops recovery | Long-term retention |

---

## **🎯 Recovery Scenarios**

### **✅ Within 35 Days (PITR works):**
```
Database crash Day 20:
• Use latest full before Day 20 (Day 14 or Day 7)
• Apply increments to Day 19
• Apply logs to crash time
• Complete recovery ✓
```

### **❌ Beyond 35 Days (IMPOSSIBLE):**
```
Need Day 40 data:
• Day 1-4 already auto-deleted
• Day 5+ are useless incrementals (no base)
• Cannot recover 💀
```

---

## **⚡ Recommended Setup**

### **Production Minimum:**
```yaml
Automated Backups:
• Retention: 35 days (MAX)
• PITR: Enabled
• Re-baseling: Automatic (~7 days)

Manual Backups:
• Monthly snapshots (keep 1 year)
• Critical for >35 day needs
```

**Why 35 days?** Multiple full snapshots + buffer + maximum allowed

---

## **⚠️ Critical Warnings**

1. **35 days = HARD limit** (AWS auto-deletes, no recovery)
2. **Incrementals useless** without full snapshot
3. **Termination = Auto backups deleted immediately**
4. **Test restores quarterly** (don't assume it works)

---

## **🔧 Quick CLI Commands**

```bash
# Set max retention (35 days)
aws rds modify-db-instance \
    --db-instance-identifier mydb \
    --backup-retention-period 35

# Manual snapshot (for >35 days)
aws rds create-db-snapshot \
    --db-snapshot-identifier "monthly-$(date +%Y-%m)" \
    --db-instance-identifier mydb
```

---

## **🎓 Key Takeaways**

### **Automated Backups:**
- **35-day maximum** (periodic re-baseling for efficiency)
- **Chain dependency** (full + increments)
- **Best for:** Daily operational recovery

### **Manual Snapshots:**
- **Independent full copies**
- **Keep indefinitely**
- **Critical for:** Anything beyond 35 days

### **Golden Rule:**
**"For data needed beyond 35 days, you MUST create manual snapshots or exports. AWS auto-deletes everything older automatically."**
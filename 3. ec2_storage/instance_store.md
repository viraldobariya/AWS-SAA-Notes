# **EC2 Instance Store**

## **What is it?**
Physical storage directly attached to the physical server hosting your EC2 instance. Not network storage like EBS.

## **Key Characteristics**
- **Ephemeral storage** - Data lost if instance is stopped/terminated
- **High performance** - Direct hardware connection (no network hops)
- **Included with instance** - No extra storage cost

## **Performance Comparison**
| Storage Type | Max IOPS | Latency | Persistence |
|-------------|----------|---------|-------------|
| Instance Store | 3.3M+ | μs (microseconds) | Lost on stop/terminate |
| EBS gp3 | 16,000 | ms (milliseconds) | Persistent |
| EBS io2 | 256,000 | sub-ms | Persistent |

## **When to Use**
**Use for:**
- Buffers, caches
- Scratch data
- Temporary processing
- High-performance computing

**Don't use for:**
- Database persistent data
- Application logs
- Long-term storage
- Critical data without backup

## **Instance Types**
Look for 'd' suffix:
- i3/i3en (NVMe SSD)
- r5d (memory optimized)
- m5d (general purpose)
- c5d (compute optimized)

## **Important Notes**
1. **No backups** - Can't take snapshots
2. **No encryption** - At hardware level
3. **Your responsibility** - To backup/replicate data
4. **Host failure** = Data loss
5. **Reboot safe** - Data persists on reboot

## **Exam Tips**
- **High performance requirement** → Instance Store
- **Ephemeral/temporary data** → Instance Store  
- **Need persistence** → EBS
- **Shared storage** → EFS
- "Hardware-attached" or "local storage" → Instance Store
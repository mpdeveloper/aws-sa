# AWS Solutions Architect Interview Guide
## 2. Foundational Knowledge - Basic Architecture

---

### 6. Design a simple 3-tier web application architecture on AWS.

**One-Liner Answer:**
*ALB → EC2 (web/app servers) in public subnets → RDS (database) in private subnets, all across multiple AZs for high availability.*

**Detailed Diagram:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                    3-TIER WEB APPLICATION                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                        INTERNET                                     │
│                           │                                         │
│                           ▼                                         │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   PRESENTATION TIER                         │   │
│  │                                                             │   │
│  │    ┌──────────────┐              ┌──────────────┐          │   │
│  │    │     ALB      │              │  CloudFront  │          │   │
│  │    │ (Multi-AZ)   │              │     CDN      │          │   │
│  │    └──────────────┘              └──────────────┘          │   │
│  │           │                               │                 │   │
│  └───────────┼───────────────────────────────┼─────────────────┘   │
│              │                               │                     │
│              ▼                               ▼                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                  APPLICATION TIER                           │   │
│  │                                                             │   │
│  │ AZ-A                              AZ-B                     │   │
│  │ ┌─────────────┐                  ┌─────────────┐          │   │
│  │ │PUBLIC SUBNET│                  │PUBLIC SUBNET│          │   │
│  │ │             │                  │             │          │   │
│  │ │ ┌─────────┐ │                  │ ┌─────────┐ │          │   │
│  │ │ │  EC2    │ │                  │ │  EC2    │ │          │   │
│  │ │ │Web/App  │ │ ◄──Auto Scaling──► │Web/App  │ │          │   │
│  │ │ │Server   │ │                  │ │Server   │ │          │   │
│  │ │ └─────────┘ │                  │ └─────────┘ │          │   │
│  │ └─────────────┘                  └─────────────┘          │   │
│  │        │                                │                 │   │
│  └────────┼────────────────────────────────┼─────────────────┘   │
│           │                                │                     │
│           ▼                                ▼                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    DATABASE TIER                            │   │
│  │                                                             │   │
│  │ AZ-A                              AZ-B                     │   │
│  │ ┌─────────────┐                  ┌─────────────┐          │   │
│  │ │PRIVATE      │                  │PRIVATE      │          │   │
│  │ │SUBNET       │                  │SUBNET       │          │   │
│  │ │             │                  │             │          │   │
│  │ │ ┌─────────┐ │                  │ ┌─────────┐ │          │   │
│  │ │ │   RDS   │ │ ◄──Replication──► │   RDS   │ │          │   │
│  │ │ │ Primary │ │                  │ │Standby  │ │          │   │
│  │ │ │ Master  │ │                  │ │(Multi-AZ│ │          │   │
│  │ │ └─────────┘ │                  │ └─────────┘ │          │   │
│  │ └─────────────┘                  └─────────────┘          │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                 ADDITIONAL COMPONENTS                       │   │
│  │                                                             │   │
│  │  Security: WAF, Security Groups, NACLs                     │   │
│  │  Monitoring: CloudWatch, CloudTrail                        │   │
│  │  Storage: S3 for static assets, EBS for EC2               │   │
│  │  Backup: RDS automated backups, S3 versioning             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Architecture Components:**
- **Presentation Tier**: ALB + CloudFront for global content delivery
- **Application Tier**: Auto Scaling EC2 instances across multiple AZs
- **Database Tier**: RDS with Multi-AZ deployment for high availability

---

### 7. How would you ensure high availability for a web application?

**One-Liner Answer:**
*Multi-AZ deployment + Auto Scaling + Health checks + Load balancers = no single point of failure.*

**Detailed Diagram:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                  HIGH AVAILABILITY DESIGN                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    MULTIPLE REGIONS                         │   │
│  │                                                             │   │
│  │  ┌──────────────┐              ┌──────────────┐            │   │
│  │  │   PRIMARY    │    Route 53  │   SECONDARY  │            │   │
│  │  │   REGION     │ ◄──Failover──► │   REGION    │            │   │
│  │  │  us-east-1   │    Routing   │  us-west-2   │            │   │
│  │  └──────────────┘              └──────────────┘            │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                             │                                       │
│                             ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                 MULTIPLE AVAILABILITY ZONES                 │   │
│  │                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │    AZ-A     │  │    AZ-B     │  │    AZ-C     │        │   │
│  │  │             │  │             │  │             │        │   │
│  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │        │   │
│  │  │ │   ALB   │ │  │ │   ALB   │ │  │ │   ALB   │ │        │   │
│  │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │        │   │
│  │  │      │      │  │      │      │  │      │      │        │   │
│  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │        │   │
│  │  │ │   EC2   │ │  │ │   EC2   │ │  │ │   EC2   │ │        │   │
│  │  │ │Instance │ │  │ │Instance │ │  │ │Instance │ │        │   │
│  │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │        │   │
│  │  │      │      │  │      │      │  │      │      │        │   │
│  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │        │   │
│  │  │ │   RDS   │ │  │ │   RDS   │ │  │ │   RDS   │ │        │   │
│  │  │ │ Replica │ │  │ │ Primary │ │  │ │ Replica │ │        │   │
│  │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   HEALTH CHECKS & MONITORING                │   │
│  │                                                             │   │
│  │  ELB Health Checks ──► Remove unhealthy instances          │   │
│  │  Auto Scaling ──────► Replace failed instances             │   │
│  │  CloudWatch ────────► Monitor metrics & alarms             │   │
│  │  Route 53 ──────────► DNS failover between regions        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   REDUNDANCY LAYERS                         │   │
│  │                                                             │   │
│  │  ✓ No single point of failure                              │   │
│  │  ✓ Automatic failover mechanisms                           │   │
│  │  ✓ Load distribution across resources                      │   │
│  │  ✓ Data replication and backup                             │   │
│  │  ✓ Monitoring and alerting                                 │   │
│  └─────────────────────────────────────────────────────────────┘

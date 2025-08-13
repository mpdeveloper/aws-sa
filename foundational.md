# AWS Solutions Architect Interview Guide
## 1. Foundational Knowledge - AWS Core Services

---

### 1. Explain the difference between EC2, ECS, and Lambda. When would you use each?

**One-Liner Answer:**
*EC2 = Virtual machines for full control, ECS = Docker containers for microservices, Lambda = serverless functions for event-driven tasks.*

**Detailed Diagram:**
```
┌────────────────────────────────────────────────────────────┐
│                    COMPUTE SERVICES COMPARISON             │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │    EC2      │    │    ECS      │    │   LAMBDA    │     │
│  │             │    │             │    │             │     │
│  │ ┌─────────┐ │    │ ┌─────────┐ │    │ ┌─────────┐ │     │
│  │ │   OS    │ │    │ │Container│ │    │ │Function │ │     │
│  │ │         │ │    │ │         │ │    │ │ Code    │ │     │
│  │ └─────────┘ │    │ └─────────┘ │    │ └─────────┘ │     │
│  │             │    │             │    │             │     │
│  │ Full Control│    │ Orchestrated│    │Event-Driven │     │
│  │ Persistent  │    │ Scalable    │    │Stateless    │     │
│  │ 24/7 Running│    │ Containers  │    │Pay per Use  │     │
│  └─────────────┘    └─────────────┘    └─────────────┘     │
│                                                            │
│  Use Cases:          Use Cases:         Use Cases:         │
│  • Web servers      • Microservices    • API backends      │
│  • Databases        • Batch jobs       • File processing   │
│  • Legacy apps      • CI/CD pipelines  • Triggers          │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**When to Use Each:**
- **EC2**: Traditional applications, need OS control, persistent workloads
- **ECS**: Containerized applications, microservices, scalable workloads
- **Lambda**: Event-driven processing, serverless applications, cost optimization

---

### 2. What are the different S3 storage classes and their use cases?

**One-Liner Answer:**
*Standard for frequent access, IA for monthly access, Glacier for yearly archives, Deep Archive for decade storage - cheaper as access frequency decreases.*

**Detailed Diagram:**
```
┌────────────────────────────────────────────────────────────────┐
│                    S3 STORAGE CLASSES                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│ Cost ↑                                      Access Speed ↑     │
│      │                                                   │     │
│      │  ┌─────────────┐  ┌─────────────┐  ┌────────────┐ │     │
│      │  │  STANDARD   │  │STANDARD-IA  │  │ONEZONE-IA  │ │     │
│      │  │             │  │             │  │            │ │     │
│      │  │ Frequent    │  │  Monthly    │  │ Monthly    │ │     │
│      │  │ Access      │  │  Access     │  │Non-Critical│ │     │
│      │  │ 99.999999%  │  │ 99.999999%  │  │99.999999%  │ │     │
│      │  │ Durability  │  │ Durability  │  │Durability  │ │     │
│      │  │ Millisecond │  │ Millisecond │  │Millisecond │ │     │
│      │  │ Retrieval   │  │ Retrieval   │  │Retrieval   │ │     │
│      │  └─────────────┘  └─────────────┘  └────────────┘ │     │
│      │                                                   │     │
│      │  ┌─────────────┐  ┌─────────────┐                 │     │
│      │  │   GLACIER   │  │DEEP ARCHIVE │                 │     │
│      │  │             │  │             │                 │     │
│      │  │   Archival  │  │Long-term    │                 │     │
│      │  │ 1-5 minutes │  │Archive      │                 │     │
│      │  │to 12 hours  │  │12 hours     │                 │     │
│      │  │ Retrieval   │  │Retrieval    │                 │     │
│      │  │             │  │             │                 │     │
│      ↓  └─────────────┘  └─────────────┘                 ↓     │
│                                                                │
│ Lifecycle Transition Flow:                                     │
│ Standard → Standard-IA → Glacier → Deep Archive                │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

**Cost vs Access Pattern:**
- **Standard**: $0.023/GB/month, immediate access
- **Standard-IA**: $0.0125/GB/month, retrieval fee
- **Glacier**: $0.004/GB/month, 1-5 min to 12 hours
- **Deep Archive**: $0.00099/GB/month, 12+ hours

---

### 3. How does Auto Scaling work and what are its benefits?

**One-Liner Answer:**
*CloudWatch metrics trigger scaling policies to add/remove EC2 instances automatically - maintaining performance while optimizing costs.*

**Detailed Diagram:**
```
┌────────────────────────────────────────────────────────────────┐
│                    AUTO SCALING ARCHITECTURE                   │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌─────────────┐    Metrics    ┌─────────────┐                 │
│  │ CLOUDWATCH  │◄──────────────┤   EC2       │                 │
│  │   ALARMS    │               │ INSTANCES   │                 │
│  │             │               │             │                 │
│  │ CPU > 80%   │               │ ┌─────┐     │                 │
│  │ Memory>70%  │    Triggers   │ │ EC2 │     │                 │
│  │ Custom      │       │       │ └─────┘     │                 │
│  └─────────────┘       │       │             │                 │
│                        │       │ ┌─────┐     │                 │
│                        ▼       │ │ EC2 │     │                 │
│  ┌─────────────┐  Scale Out/In │ └─────┘     │                 │
│  │AUTO SCALING │◄──────────────┤             │                 │
│  │   GROUP     │               │ ┌─────┐     │                 │
│  │             │               │ │ EC2 │     │                 │
│  │Min: 2       │               │ └─────┘     │                 │
│  │Desired: 3   │               │             │                 │
│  │Max: 10      │               └─────────────┘                 │
│  └─────────────┘                                               │
│                                                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                  SCALING POLICIES                       │   │
│  │                                                         │   │
│  │  Target Tracking: Maintain 50% CPU utilization          │   │
│  │  Step Scaling: Add 2 instances when CPU > 80%           │   │
│  │  Scheduled: Scale up at 9 AM, scale down at 6 PM        │   │
│  │  Predictive: Use ML to predict and pre-scale            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                │
│  Benefits:                                                     │
│  ✓ Automatic capacity management                               │
│  ✓ Cost optimization (pay for what you use)                    │
│  ✓ High availability and fault tolerance                       │
│  ✓ Better performance during peak loads                        │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

**Key Components:**
- **Auto Scaling Group**: Logical collection of EC2 instances
- **Launch Template**: Defines instance configuration
- **Scaling Policies**: Rules for when to scale
- **CloudWatch Alarms**: Triggers based on metrics

---

### 4. Explain VPC, subnets, and security groups with a practical example.

**One-Liner Answer:**
*VPC is your private cloud network, subnets divide it by AZ/purpose, security groups are stateful firewalls - like having floors (subnets) in a building (VPC) with door guards (security groups).*

**Detailed Diagram:**
```
┌────────────────────────────────────────────────────────────────────┐
│                        VPC ARCHITECTURE                            │
│                      (10.0.0.0/16)                                 │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌─────────────────────┐    ┌─────────────────────┐                │
│  │   AVAILABILITY      │    │   AVAILABILITY      │                │
│  │     ZONE A          │    │     ZONE B          │                │
│  │                     │    │                     │                │
│  │ ┌─────────────────┐ │    │ ┌─────────────────┐ │                │
│  │ │ PUBLIC SUBNET   │ │    │ │ PUBLIC SUBNET   │ │                │
│  │ │ (10.0.1.0/24)   │ │    │ │ (10.0.2.0/24)   │ │                │
│  │ │                 │ │    │ │                 │ │                │
│  │ │ ┌─────┐ ┌─────┐ │ │    │ │ ┌─────┐ ┌─────┐ │ │                │
│  │ │ │ EC2 │ │ ALB │ │ │    │ │ │ EC2 │ │ ALB │ │ │                │
│  │ │ │Web  │ │     │ │ │    │ │ │Web  │ │     │ │ │                │
│  │ │ └─────┘ └─────┘ │ │    │ │ └─────┘ └─────┘ │ │                │
│  │ │                 │ │    │ │                 │ │                │
│  │ │      │ IGW      │ │    │ │      │ IGW      │ │                │
│  │ └──────┼──────────┘ │    │ └──────┼──────────┘ │                │
│  │        │            │    │        │            │                │
│  │ ┌──────┼──────────┐ │    │ ┌──────┼──────────┐ │                │
│  │ │ PRIVATE SUBNET  │ │    │ │ PRIVATE SUBNET  │ │                │
│  │ │ (10.0.3.0/24)   │ │    │ │ (10.0.4.0/24)   │ │                │
│  │ │                 │ │    │ │                 │ │                │
│  │ │ ┌─────┐ ┌─────┐ │ │    │ │ ┌─────┐ ┌─────┐ │ │                │
│  │ │ │ EC2 │ │ RDS │ │ │    │ │ │ EC2 │ │ RDS │ │ │                │
│  │ │ │App  │ │ DB  │ │ │    │ │ │App  │ │ DB  │ │ │                │
│  │ │ └─────┘ └─────┘ │ │    │ │ └─────┘ └─────┘ │ │                │
│  │ │        │        │ │    │ │        │        │ │                │
│  │ │      NAT GW     │ │    │ │      NAT GW     │ │                │
│  │ └─────────────────┘ │    │ └─────────────────┘ │                │
│  └─────────────────────┘    └─────────────────────┘                │
│                                                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                 SECURITY GROUPS                             │   │
│  │                                                             │   │
│  │  Web Tier SG:        App Tier SG:        DB Tier SG:        │   │
│  │  Port 80,443 ←──     Port 8080 ←──       Port 3306 ←──      │   │
│  │  from Internet       from Web SG         from App SG        │   │
│  │                                                             │   │
│  │  ✓ Stateful (return traffic automatic)                      │   │
│  │  ✓ Default deny all inbound                                 │   │
│  │  ✓ Default allow all outbound                               │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

**Real-World Example: E-commerce Application**
- **Public Subnets**: Web servers, Load balancers (Internet-facing)
- **Private Subnets**: Application servers, Databases (Internal only)
- **Security Groups**: Layer-based access control
- **NAT Gateway**: Outbound internet access for private instances

---

### 5. What are the different ELB types (ALB, NLB, CLB)?

**One-Liner Answer:**
*ALB for HTTP/HTTPS with advanced routing, NLB for TCP/UDP with extreme performance, CLB for basic load balancing (legacy).*

**Detailed Diagram:**
```
┌────────────────────────────────────────────────────────────────┐
│                    ELASTIC LOAD BALANCER TYPES                 │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              APPLICATION LOAD BALANCER (ALB)            │   │
│  │                    Layer 7 (HTTP/HTTPS)                 │   │
│  │                                                         │   │
│  │  Internet ──► ALB ──┬──► Target Group 1 (EC2)           │   │
│  │               │     │    /api/* → Microservice A        │   │
│  │               └──┬──► Target Group 2 (Lambda)           │   │
│  │                  │    /images/* → Image Service         │   │
│  │                  └──► Target Group 3 (IP)               │   │
│  │                       Header-based routing              │   │
│  │                                                         │   │
│  │  Features: Path routing, Host routing, HTTP headers     │   │
│  │           WebSocket, HTTP/2, SSL termination            │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              NETWORK LOAD BALANCER (NLB)                │   │
│  │                    Layer 4 (TCP/UDP)                    │   │
│  │                                                         │   │
│  │  Internet ──► NLB ──┬──► Target Group (EC2)             │   │
│  │               │     │    Ultra-low latency              │   │
│  │               │     │    Millions of RPS                │   │
│  │               └──►  └──► Static IP addresses            │   │
│  │                          Preserve source IP             │   │
│  │                                                         │   │
│  │  Features: Extreme performance, Static IP, TCP/UDP      │   │
│  │           Source IP preservation, PrivateLink           │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              CLASSIC LOAD BALANCER (CLB)                │   │
│  │                 Layer 4 & 7 (Legacy)                    │   │
│  │                                                         │   │
│  │  Internet ──► CLB ──┬──► EC2 Instance 1                 │   │
│  │                     │                                   │   │
│  │                     ├──► EC2 Instance 2                 │   │
│  │                     │                                   │   │
│  │                     └──► EC2 Instance 3                 │   │
│  │                                                         │   │
│  │  Features: Basic load balancing, EC2 Classic support    │   │
│  │           Simple health checks, SSL offloading          │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   WHEN TO USE WHAT                      │   │
│  │                                                         │   │
│  │  ALB: Web applications, microservices, containers       │   │
│  │  NLB: Gaming, IoT, real-time apps, extreme performance  │   │
│  │  CLB: Legacy applications, EC2-Classic (deprecated)     │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

**Decision Matrix:**

| Requirement | ALB | NLB | CLB |
| ------------ | ----- | ----- | ----- |
| Cost | Medium | Higher | Lower |
| Performance (RPS) | 100K+ | Millions | 10K+ |
| Static IP | ❌ | ✅ | ❌ |
| WebSocket support | ✅ | ✅ | ❌ |
| HTTP/HTTPS routing | ✅ | ❌ | Basic |


---

**Key Takeaways for Interviews:**
1. **Understand the hierarchy**: VPC → Subnets → Security Groups
2. **Know the use cases**: When to choose each service
3. **Think about cost**: Not all solutions need the most expensive option
4. **Consider scalability**: Design for growth from day one
5. **Security first**: Always implement proper access controls

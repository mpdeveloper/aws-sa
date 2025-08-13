# AWS Solutions Architect Interview Guide
## 3. Intermediate Level - Advanced Architecture & Design Patterns

---

### 11. Design a microservices architecture for an e-commerce platform. Include API Gateway, containers, databases, and monitoring.

**One-Liner Answer:**
*API Gateway → ALB → ECS/Fargate containers → separate databases per service (RDS/DynamoDB) → CloudWatch + X-Ray for monitoring.*

**Detailed Diagram:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                E-COMMERCE MICROSERVICES ARCHITECTURE                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                           CLIENTS                                   │
│              (Web, Mobile, Partners, Internal)                      │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    API GATEWAY                              │   │
│  │  • Authentication & Authorization                           │   │
│  │  • Rate Limiting & Throttling                              │   │
│  │  • Request/Response Transformation                         │   │
│  │  • API Documentation & Versioning                          │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                APPLICATION LOAD BALANCER                    │   │
│  │              (Path-based Routing)                           │   │
│  └─────────────────────────────────────────────────────────────┘   │
│             │            │            │            │                │
│             ▼            ▼            ▼            ▼                │
│  ┌─────────────┐┌─────────────┐┌─────────────┐┌─────────────┐      │
│  │   USER      ││  PRODUCT    ││   ORDER     ││  PAYMENT    │      │
│  │  SERVICE    ││  SERVICE    ││  SERVICE    ││  SERVICE    │      │
│  │             ││             ││             ││             │      │
│  │┌─────────┐  ││┌─────────┐  ││┌─────────┐  ││┌─────────┐  │      │
│  ││ECS Task │  │││ECS Task │  │││ECS Task │  │││ECS Task │  │      │
│  ││Container│  │││Container│  │││Container│  │││Container│  │      │
│  │└─────────┘  ││└─────────┘  ││└─────────┘  ││└─────────┘  │      │
│  │Auto Scaling ││Auto Scaling ││Auto Scaling ││Auto Scaling │      │
│  └─────────────┘└─────────────┘└─────────────┘└─────────────┘      │
│           │              │              │              │           │
│           ▼              ▼              ▼              ▼           │
│  ┌─────────────┐┌─────────────┐┌─────────────┐┌─────────────┐      │
│  │   RDS       ││  DynamoDB   ││   RDS       ││  DynamoDB   │      │
│  │ (PostgreSQL)││ (NoSQL)     ││ (MySQL)     ││ (NoSQL)     │      │
│  │             ││             ││             ││             │      │
│  │• User data  ││• Product    ││• Order data ││• Payment    │      │
│  │• Profiles   ││  catalog    ││• Order      ││  tokens     │      │
│  │• Preferences││• Inventory  ││  history    ││• Audit logs │      │
│  └─────────────┘└─────────────┘└─────────────┘└─────────────┘      │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                  SUPPORTING SERVICES                        │   │
│  │                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │NOTIFICATION │  │   SEARCH    │  │   ANALYTICS │        │   │
│  │  │  SERVICE    │  │  SERVICE    │  │   SERVICE   │        │   │
│  │  │             │  │             │  │             │        │   │
│  │  │• Email/SMS  │  │• ElasticSearch│ │• Data Lake  │        │   │
│  │  │• Push       │  │• Product     │  │• Reports    │        │   │
│  │  │  Notifications│ │  Indexing   │  │• Metrics    │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                INTER-SERVICE COMMUNICATION                  │   │
│  │                                                             │   │
│  │  Synchronous:              Asynchronous:                   │   │
│  │  • HTTP/REST APIs          • SQS/SNS for events           │   │
│  │  • Service Mesh (Istio)    • EventBridge for integration  │   │
│  │  • Service Discovery       • Kinesis for streaming        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   MONITORING & OBSERVABILITY                │   │
│  │                                                             │   │
│  │  • CloudWatch: Metrics, Logs, Alarms                       │   │
│  │  • X-Ray: Distributed Tracing                              │   │
│  │  • ELK Stack: Centralized Logging                          │   │
│  │  • Prometheus + Grafana: Custom Metrics                    │   │
│  │  • AWS Config: Configuration Compliance                    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Microservices Design Principles:**
- **Single Responsibility**: Each service owns one business domain
- **Database per Service**: Data isolation and independence
- **Decentralized Governance**: Teams own their services
- **Failure Isolation**: Service failures don't cascade
- **Technology Diversity**: Choose best tool for each service

---

### 12. A company has a legacy monolithic application running on-premises. Design a migration strategy to AWS with minimal downtime.

**One-Liner Answer:**
*Lift-and-shift to EC2 first, then strangler pattern to gradually extract microservices using API Gateway and database replication.*

**Detailed Diagram:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                    LEGACY MIGRATION STRATEGY                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    PHASE 1: ASSESS                          │   │
│  │                  (Weeks 1-2)                                │   │
│  │                                                             │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │   │
│  │  │ APPLICATION  │  │ DEPENDENCIES │  │ PERFORMANCE  │     │   │
│  │  │ DISCOVERY    │  │ MAPPING      │  │ BASELINE     │     │   │
│  │  │              │  │              │  │              │     │   │
│  │  │• AWS App     │  │• Database    │  │• Current     │     │   │
│  │  │  Discovery   │  │  connections │  │  metrics     │     │   │
│  │  │• Code        │  │• External    │  │• Load        │     │   │
│  │  │  analysis    │  │  services    │  │  patterns    │     │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                 PHASE 2: LIFT AND SHIFT                    │   │
│  │                    (Weeks 3-8)                             │   │
│  │                                                             │   │
│  │  ON-PREMISES                      AWS CLOUD                │   │
│  │  ┌──────────────┐     Migration   ┌──────────────┐        │   │
│  │  │   MONOLITH   │ ──────────────► │   MONOLITH   │        │   │
│  │  │              │                 │              │        │   │
│  │  │ ┌──────────┐ │                 │ ┌──────────┐ │        │   │
│  │  │ │   APP    │ │                 │ │   APP    │ │        │   │
│  │  │ │ SERVER   │ │    DMS          │ │ SERVER   │ │        │   │
│  │  │ └──────────┘ │ ┌────────────┐  │ │  (EC2)   │ │        │   │
│  │  │              │ │            │  │ └──────────┘ │        │   │
│  │  │ ┌──────────┐ │ │  DATABASE  │  │              │        │   │
│  │  │ │ DATABASE │ │─┤ MIGRATION  ├──┤ ┌──────────┐ │        │   │
│  │  │ │          │ │ │  SERVICE   │  │ │   RDS    │ │        │   │
│  │  │ └──────────┘ │ └────────────┘  │ │ DATABASE │ │        │   │
│  │  └──────────────┘                 │ └──────────┘ │        │   │
│  │                                   └──────────────┘        │   │
│  │                                                             │   │
│  │  Tools: AWS Application Migration Service (MGN)            │   │
│  │        AWS Database Migration Service (DMS)                │   │
│  │        AWS Schema Conversion Tool (SCT)                    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │               PHASE 3: STRANGLER PATTERN                   │   │
│  │                  (Months 3-12)                             │   │
│  │                                                             │   │
│  │        ┌─────────────────────────────────────────┐         │   │
│  │        │            API GATEWAY                  │         │   │
│  │        │        (Traffic Router)                 │         │   │
│  │        └─────────────────────────────────────────┘         │   │
│  │                 │            │            │                 │   │
│  │                 ▼            ▼            ▼                 │   │
│  │     ┌─────────────┐ ┌─────────────┐ ┌─────────────┐       │   │
│  │     │NEW MICRO    │ │NEW MICRO    │ │  LEGACY     │       │   │
│  │     │SERVICE 1    │ │SERVICE 2    │ │ MONOLITH    │       │   │
│  │     │             │ │             │ │             │       │   │
│  │     │• User Mgmt  │ │• Orders     │ │• Remaining  │       │   │
│  │     │• Lambda/ECS │ │• Lambda/ECS │ │  Functions  │       │   │
│  │     │• DynamoDB   │ │• RDS        │ │• EC2        │       │   │
│  │     └─────────────┘ └─────────────┘ └─────────────┘       │   │
│  │                                                             │   │
│  │  Migration Strategy:                                       │   │
│  │  1. Route new features to microservices                   │   │
│  │  2. Gradually extract functions from monolith             │   │
│  │  3. Update API Gateway routing rules                      │   │
│  │  4. Retire monolith components incrementally              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                PHASE 4: OPTIMIZE                           │   │
│  │                 (Ongoing)                                  │   │
│  │                                                             │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐         │   │
│  │  │PERFORMANCE  │ │    COST     │ │  SECURITY   │         │   │
│  │  │             │ │             │ │             │         │   │
│  │  │• Auto       │ │• Reserved   │ │• WAF        │         │   │
│  │  │  Scaling    │ │  Instances  │ │• KMS        │         │   │
│  │  │• Caching    │ │• Spot       │ │• VPC        │         │   │
│  │  │• CDN        │ │  Instances  │ │• IAM        │         │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                  MINIMAL DOWNTIME TACTICS                  │   │
│  │                                                             │   │
│  │  • Blue/Green Deployment for application migration         │   │
│  │  • Database replication with cutover during low traffic    │   │
│  │  • DNS routing for gradual traffic shifting               │   │
│  │  • Feature flags for controlled rollouts                  │   │
│  │  • Rollback procedures for each phase                     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Migration Timeline:**
- **Assessment**: 1-2 weeks
- **Lift & Shift**: 3-8 weeks
- **Modernization**: 3-12 months
- **Optimization**: Ongoing

---

### 13. How would you implement a disaster recovery solution with RTO of 4 hours and RPO of 1 hour?

**One-Liner Answer:**
*Pilot Light strategy: automated backups every hour + pre-configured AMIs + CloudFormation templates for rapid infrastructure recreation.*

**Detailed Diagram:**
```
┌─────────────────────────────────────────────────────────────────────┐
│              DISASTER RECOVERY - PILOT LIGHT STRATEGY              │
│                    RTO: 4 Hours | RPO: 1 Hour                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   PRIMARY REGION (us-east-1)                │   │
│  │                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │APPLICATION  │  │  DATABASE   │  │   STORAGE   │        │   │
│  │  │    TIER     │  │    TIER     │  │    TIER     │        │   │
│  │  │             │  │             │  │             │        │   │
│  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │        │   │
│  │  │ │   ALB   │ │  │ │   RDS   │ │  │ │   S3    │ │        │   │
│  │  │ └─────────┘ │  │ │ Primary │ │  │ │ Primary │ │        │   │
│  │  │      │      │  │ └─────────┘ │  │ └─────────┘ │        │   │
│  │  │ ┌─────────┐ │  │      │      │  │      │      │        │   │
│  │  │ │   EC2   │ │  │      │      │  │      │      │        │   │
│  │  │ │Instance │ │  │ Automated   │  │ Cross-Region│        │   │
│  │  │ │ Cluster │ │  │ Snapshots   │  │ Replication │        │   │
│  │  │ └─────────┘ │  │ Every Hour  │  │ Continuous  │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  │                           │               │                │   │
│  │                           ▼               ▼                │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │               │                     │
│                              ▼               ▼                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                 DISASTER RECOVERY REGION (us-west-2)        │   │
│  │                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │   PILOT     │  │  DATABASE   │  │   STORAGE   │        │   │
│  │  │   LIGHT     │  │   BACKUP    │  │   REPLICA   │        │   │
│  │  │             │  │             │  │             │        │   │
│  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │        │   │
│  │  │ │  AMI    │ │  │ │   RDS   │ │  │ │   S3    │ │        │   │
│  │  │ │Templates│ │  │ │Snapshots│ │  │ │ Replica │ │        │   │
│  │  │ └─────────┘ │  │ │(Hourly) │ │  │ │Bucket   │ │        │   │
│  │  │      │      │  │ └─────────┘ │  │ └─────────┘ │        │   │
│  │  │ ┌─────────┐ │  │             │  │             │        │   │
│  │  │ │CloudFor-│ │  │ ┌─────────┐ │  │ ┌─────────┐ │        │   │
│  │  │ │mation   │ │  │ │Database │ │  │ │EBS      │ │        │   │
│  │  │ │Templates│ │  │ │Security │ │  │ │Snapshots│ │        │   │
│  │  │ └─────────┘ │  │ │Groups   │ │  │ │         │ │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  │                                                             │   │
│  │         🔥 DISASTER OCCURS 🔥                              │   │
│  │                    │                                        │   │
│  │                    ▼                                        │   │
│  │                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │ RECOVERY    │  │ DATABASE    │  │ STORAGE     │        │   │
│  │  │INFRASTRUCTURE│  │ RESTORATION │  │ AVAILABLE   │        │   │
│  │  │             │  │             │  │             │        │   │
│  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │        │   │
│  │  │ │Launch   │ │  │ │Restore  │ │  │ │   S3    │ │        │   │
│  │  │ │CloudFor-│ │  │ │RDS from │ │  │ │ Data    │ │        │   │
│  │  │ │mation   │ │  │ │Latest   │ │  │ │Already  │ │        │   │
│  │  │ │Stack    │ │  │ │Snapshot │ │  │ │Available│ │        │   │
│  │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │        │   │
│  │  │      │      │  │      │      │  │      │      │        │   │
│  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │        │   │
│  │  │ │Deploy   │ │  │ │Update   │ │  │ │Attach   │ │        │   │
│  │  │ │EC2      │ │  │ │DNS to   │ │  │ │EBS      │ │        │   │
│  │  │ │from AMI │ │  │ │Point to │ │  │ │Volumes  │ │        │   │
│  │  │ └─────────┘ │  │ │DR Region│ │  │ └─────────┘ │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  │                                                             │   │
│  │                 ⏱️ RECOVERY TIMELINE ⏱️                    │   │
│  │  Hour 0: Disaster detected, alerts triggered              │   │
│  │  Hour 1: DR team mobilized, recovery initiated            │   │
│  │  Hour 2: Infrastructure deployment in progress            │   │
│  │  Hour 3: Database restoration, DNS updates                │   │
│  │  Hour 4: Application fully operational ✅                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   AUTOMATION & MONITORING                   │   │
│  │                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │ CLOUDWATCH  │  │   LAMBDA    │  │    SNS      │        │   │
│  │  │   ALARMS    │  │ AUTOMATION  │  │NOTIFICATIONS│        │   │
│  │  │             │  │             │  │             │        │   │
│  │  │• Health     │  │• Trigger DR │  │• Alert DR   │        │   │
│  │  │  Checks     │  │  procedures │  │  team       │        │   │
│  │  │• Response   │  │• Launch     │  │• Status     │        │   │
│  │  │  Time       │  │  CloudForm  │  │  updates    │        │   │
│  │  │• Custom     │  │• Update DNS │  │• Escalation │        │   │
│  │  │  Metrics    │  │• Run tests  │  │  procedures │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                      COST OPTIMIZATION                      │   │
│  │                                                             │   │
│  │  Pilot Light Benefits:                                     │   │
│  │  • Minimal running costs in DR region                      │   │
│  │  • Only pay for storage replication                        │   │
│  │  • Quick recovery compared to backup/restore               │   │
│  │  • Automated testing capabilities                          │   │
│  │                                                             │   │
│  │  Monthly DR Costs: ~10-15% of primary infrastructure      │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**DR Strategy Options:**
- **Backup & Restore**: Cheapest, slowest (RTO: 24+ hours)
- **Pilot Light**: Low cost, moderate speed (RTO: 4-8 hours) ⭐
- **Warm Standby**: Higher cost, faster (RTO: 30 minutes - 1 hour)
- **Multi-Site Active/Active**: Highest cost, instant failover

---

### 14. Design a data lake architecture for processing both batch and real-time data streams.

**One-Liner Answer:**
*Kinesis Data Streams → Lambda/Kinesis Analytics (real-time) + S3 → Glue/EMR (batch) → Athena/QuickSight for analytics.*

**Detailed Diagram:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                        DATA LAKE ARCHITECTURE                      │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    DATA INGESTION                           │   │
│  │                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │ REAL-TIME   │  │   BATCH     │  │ STREAMING   │        │   │
│  │  │   DATA      │  │   DATA      │  │    LOGS     │        │   │
│  │  │             │  │             │  │             │        │   │
│  │  │• IoT        │  │• Database   │  │• Application│        │   │
│  │  │  Sensors    │  │  Exports    │  │  Logs       │        │   │
│  │  │• APIs       │  │• CSV Files  │  │• Web        │        │   │
│  │  │• Events     │  │• Backups    │  │  Clickstream│        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  │         │               │               │                 │   │
│  │         ▼               ▼               ▼                 │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │  KINESIS    │  │   BATCH     │  │  KINESIS    │        │   │
│  │  │    DATA     │  │ INGESTION   │  │    DATA     │        │   │
│  │  │  STREAMS    │  │             │  │ FIREHOSE    │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│         │               │               │                         │
│         ▼               ▼               ▼                         │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                      DATA LAKE STORAGE                      │   │
│  │                         (Amazon S3)                        │   │
│  │                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │   RAW       │  │  PROCESSED  │  │  CURATED    │        │   │
│  │  │   ZONE      │  │    ZONE     │  │    ZONE     │        │   │
│  │  │             │  │             │  │             │        │   │
│  │  │• Original   │  │• Cleaned    │  │• Business   │        │   │
│  │  │  Format     │  │• Validated  │  │  Ready      │        │   │
│  │  │• All Data   │  │• Structured │  │• Aggregated │        │   │
│  │  │• Immutable  │  │• Partitioned│  │• Optimized  │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  │                                                             │   │
│  │  Storage Classes: Standard → IA → Glacier → Deep Archive  │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    DATA PROCESSING                          │   │
│  │                                                             │   │
│  │  ┌─────────────┐              ┌─────────────┐              │   │
│  │  │ REAL-TIME   │              │   BATCH     │              │   │
│  │  │ PROCESSING  │              │ PROCESSING  │              │   │
│  │  │             │              │             │              │   │
│  │  │┌─────────┐  │              │┌─────────┐  │              │   │
│  │  ││ LAMBDA  │  │              ││   EMR   │  │              │   │
│  │  ││Functions│  │              ││ Hadoop/ │  │              │   │
│  │  ││         │  │              ││ Spark   │  │              │   │
│  │  │└─────────┘  │              │└─────────┘  │              │   │
│  │  │             │              │             │              │   │
│  │  │┌─────────┐  │              │┌─────────┐  │              │   │
│  │  ││KINESIS  │  │              ││  GLUE   │  │              │   │
│  │  ││Analytics│  │              ││  ETL    │  │              │   │
│  │  │└─────────┘  │              │└─────────┘  │              │   │
│  │  │             │              │             │              │   │
│  │  │• Stream     │              │• Scheduled  │              │   │
│  │  │  Processing │              │  Jobs       │              │   │
│  │  │• Real-time  │              │• Large-scale│              │   │
│  │  │  Alerts     │              │  Processing │              │   │
│  │  └─────────────┘              └─────────────┘              │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   DATA CATALOGING                           │   │
│  │                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │  AWS GLUE   │  │  METADATA   │  │    DATA     │        │   │
│  │  │  CATALOG    │  │  MANAGEMENT │  │ GOVERNANCE  │        │   │
│  │  │             │  │             │  │             │        │   │
│  │  │• Schema     │  │• Data       │  │• Access     │        │   │
│  │  │  Discovery  │  │  Lineage    │  │  Control    │        │   │
│  │  │• Automated  │  │• Quality    │  │• Data       │        │   │
│  │  │  Crawling   │  │  Monitoring │  │  Privacy    │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                  DATA ANALYTICS & CONSUMPTION               │   │
│  │                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │   ATHENA    │  │  REDSHIFT   │  │ QUICKSIGHT  │        │   │
│  │  │             │  │             │  │             │        │   │
│  │  │• Serverless │  │• Data       │  │• Business   │        │   │
│  │  │  Queries    │  │  Warehouse  │  │  Intelligence│       │   │
│  │  │• Pay per    │  │• OLAP       │  │• Dashboards │        │   │
│  │  │  Query      │  │• Complex    │  │• Reports    │        │   │
│  │  │             │  │  Analytics  │  │             │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  │                                                             │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │   │
│  │  │ MACHINE     │  │   API       │  │ THIRD-PARTY │        │   │
│  │  │ LEARNING    │  │ CONSUMERS   │  │ INTEGRATIONS│        │   │
│  │  │             │  │             │  │             │        │   │
│  │  │• SageMaker  │  │• REST APIs  │  │• Partner    │        │   │
│  │  │• Model      │  │• GraphQL    │  │  Systems    │        │   │
│  │  │  Training   │  │• Data       │  │• External   │        │   │
│  │  │• Inference  │  │  Products   │  │  Analytics  │        │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Data Lake Design Principles:**
- **Schema on Read**: Store data in native format, apply schema when reading
- **Decoupled Storage & Compute**: Scale independently based on needs
- **Multiple Processing Engines**: Use best tool for each workload
- **Data Governance**: Catalog, lineage, and access control from day one

---

### 15. Create a multi-region deployment strategy for a global application with data sovereignty requirements.

**One-Liner Answer:**
*Active-Active regions with Route 53 latency routing + regional data stores + cross-region replication only for non-sensitive data.*

**Detailed Diagram:**
```
┌─────────────────────────────────────────────────────────────────────┐
│              GLOBAL MULTI-REGION ARCHITECTURE                      │
│                   (Data Sovereignty Compliant)                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                          GLOBAL DNS                                 │
│                      ┌─────────────┐                               │
│                      │  ROUTE 53   │                               │
│                      │   HEALTH    │                               │
│                      │   CHECKS    │                               │
│                      └─────────────┘                               │
│                           │                                         │
│         Latency-based     │      Geographic                         │
│         Routing          │       Routing                           │
│              ┌──────────┼──────────┐                               │
│              │          │          │                               │
│              ▼          ▼          ▼                               │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                  │
│  │   US-EAST   │ │   EU-WEST   │ │  AP-SOUTH   │                  │
│  │   REGION    │ │   REGION    │ │   REGION    │                  │
│  │ (Virginia)  │ │ (Ireland)   │ │ (Mumbai)    │                  │
│  └─────────────┘ └─────────────┘ └─────────────┘                  │
│        │              │              │                             │
│        ▼              ▼              ▼                             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                  │
│  │    US       │ │     EU      │ │   ASIA      │                  │
│  │ APPLICATION │ │ APPLICATION │ │APPLICATION  │                  │
│  │ DEPLOYMENT  │ │ DEPLOYMENT  │ │ DEPLOYMENT  │                  │
│  │             │ │             │ │             │                  │
│  │┌─────────┐  │ │┌─────────┐  │ │┌─────────┐  │                  │
│  ││   CDN   │  │ ││   CDN   │  │ ││   CDN   │  │                  │
│  ││CloudFront│  │ ││CloudFront│  │ ││CloudFront│  │                  │
│  │└─────────┘  │ │└─────────┘  │ │└─────────┘  │                  │
│  │     │       │ │     │       │ │     │       │                  │
│  │┌─────────┐  │ │┌─────────┐  │ │┌─────────┐  │                  │
│  ││   ALB   │  │ ││   ALB   │  │ ││   ALB   │  │                  │
│  │└─────────┘  │ │└─────────┘  │ │└─────────┘  │                  │
│  │     │       │ │     │       │ │     │       │                  │
│  │┌─────────┐  │ │┌─────────┐  │ │┌─────────┐  │                  │
│  ││   ECS   │  │ ││   ECS   │  │ ││   ECS   │  │                  │
│  ││Fargate  │  │ ││Fargate  │  │ ││Fargate  │  │                  │
│  ││Cluster  │  │ ││Cluster  │  │ ││Cluster  │  │                  │
│  │└─────────┘  │ │└─────────┘  │ │└─────────┘  │                  │
│  └─────────────┘ └─────────────┘ └─────────────┘                  │
│        │              │              │                             │
│        ▼              ▼              ▼                             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐                  │
│  │    US       │ │     EU      │ │   ASIA      │                  │
│  │    DATA     │ │    DATA     │ │   DATA      │                  │
│  │   STORES    │ │   STORES    │ │  STORES     │                  │
│  │             │ │             │ │             │                  │
│  │┌─────────┐  │ │┌─────────┐  │ │┌─────────┐  │                  │
│  ││   RDS   │  │ ││   RDS   │  │ ││   RDS   │  │                  │
│  ││  (US     │  │ ││  (EU     │  │ ││ (APAC   │  │                  │
│  ││

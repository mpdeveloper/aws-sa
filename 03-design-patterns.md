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
│  │

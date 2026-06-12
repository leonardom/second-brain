# Azure Reference — Services & Patterns

> Tags: #scalability #reliability #security

A curated reference for commonly used Azure services, best practices, and architecture patterns.

---

## Compute

### Virtual Machines (VMs)
- Use **Virtual Machine Scale Sets (VMSS)** for auto-scaling groups; prefer **Flexible orchestration** (supports mixing instance types)
- Use **Spot VMs** for stateless, fault-tolerant workloads (up to 90% cost savings); set eviction policy to `Deallocate` to preserve disk
- **Proximity Placement Groups** reduce latency for tightly coupled workloads
- Prefer **Azure Compute Units (ACU)** as a normalized metric for comparing VM performance across SKUs
- Use **Ephemeral OS Disks** to eliminate OS disk I/O costs and reduce boot time

### Azure Functions
- Max execution: 10 min (Consumption) / unlimited (Premium/Dedicated) | Max memory: 1.5 GB (Consumption) / 14 GB (Premium)
- **Cold start mitigation**: Premium Plan (pre-warmed instances), Always Ready instances, or Dedicated plan
- Use **Durable Functions** for stateful workflows, fan-out/fan-in, and long-running orchestrations
- **Triggers**: HTTP, Timer, Service Bus, Event Hubs, Blob, Cosmos DB change feed, and more
- Use [Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools) for local development

### AKS vs Container Apps vs App Service

| | **AKS** | **Container Apps** | **App Service** |
|-|:---:|:---:|:---:|
| Kubernetes ecosystem | ✅ | Partial (KEDA/Dapr) | ❌ |
| Operational complexity | High | Low | Low |
| Scale to zero | ❌ | ✅ | ❌ |
| Serverless pricing | ❌ | ✅ | ❌ |
| Best for | Full K8s control | Microservices / event-driven | Web apps, APIs |

- **AKS** — use when you need the full Kubernetes ecosystem or multi-cloud portability
- **Azure Container Apps** — serverless containers with built-in KEDA scaling and Dapr integration
- **App Service** — simplest option for web apps/APIs; built-in autoscale, deployment slots, and managed TLS

---

## Storage

### Azure Blob Storage
- **Access Tiers**: Hot → Cool → Cold → Archive (rehydration from Archive takes hours)
- Enable **Lifecycle Management Policies** to transition blobs between tiers automatically
- Use **Azure CDN** or **Azure Front Door** to serve blobs globally with low latency
- Enable **Versioning** and **Soft Delete** (configurable retention) for accidental-deletion protection
- Use **Shared Access Signatures (SAS)** for time-limited, scoped access (prefer **User Delegation SAS** — backed by Entra ID, no storage account key needed)
- **ZRS (Zone-Redundant Storage)** for HA within a region; **GRS/GZRS** for cross-region DR

### Azure SQL / SQL Managed Instance

| | **Azure SQL Database** | **SQL Managed Instance** |
|-|:---:|:---:|
| Compatibility | Mostly SQL Server | Near 100% SQL Server |
| VNet integration | Private endpoint | Native VNet |
| SQL Agent | ❌ | ✅ |
| Best for | New cloud-native apps | Lift-and-shift from on-prem |

- Use **Serverless tier** for intermittent/unpredictable workloads (auto-pause, per-second billing)
- Enable **Geo-Replication** or **Failover Groups** for regional failover
- Use **Elastic Pools** to share DTUs/vCores across multiple databases (cost-effective for multi-tenant SaaS)
- Enable **Always Encrypted** for column-level encryption of sensitive data

### Azure Cosmos DB
- Globally distributed, multi-model NoSQL database with **five consistency levels**:
  1. Strong
  2. Bounded Staleness
  3. Session *(default — consistent within a session; good balance)*
  4. Consistent Prefix
  5. Eventual
- Choose the **API** that fits your model: NoSQL (JSON), MongoDB, Cassandra, Gremlin (graph), Table
- **Partition key** design is critical — pick a key with high cardinality and even write distribution
- Use **Cosmos DB Change Feed** for event-driven processing (similar to DynamoDB Streams)
- **Request Units (RUs)** — normalize compute cost; provision RU/s or use **Serverless** (pay-per-request)
- Use **TTL** at container or item level for automatic data expiry

### Azure Cache for Redis
- Managed Redis; use for session state, caching, pub/sub, leaderboards
- **Tiers**: Basic (no SLA, dev/test) → Standard (replication, SLA) → Premium (clustering, persistence, VNet)
- Enable **geo-replication** (Premium) for active-passive cross-region cache
- Use **Redis Modules** (RediSearch, RedisJSON) on Enterprise tier

---

## Networking

### Virtual Network (VNet) Design (Best Practice)
```
VNet (e.g., 10.0.0.0/16)
├── Public Subnet    (10.0.0.0/24)  — Application Gateway, Azure Firewall, Bastion
├── App Subnet       (10.0.1.0/24)  — App servers, AKS nodes, Container Apps
└── Data Subnet      (10.0.2.0/24)  — Databases, Redis, private endpoints
```

- Use **Network Security Groups (NSGs)** on subnets and NICs for stateful allow/deny rules
- Use **Azure Private Endpoints** to access PaaS services (Storage, SQL, Cosmos DB, Key Vault) over private IP — eliminates public internet exposure
- Use **Service Endpoints** as a lighter alternative when Private Endpoints are not required
- **Azure Bastion** — browser-based SSH/RDP to VMs without exposing a public IP or jump box
- Use **VNet Peering** (same region) or **Global VNet Peering** (cross-region) to connect VNets; traffic stays on the Microsoft backbone

### Load Balancing

| | **Application Gateway** | **Azure Load Balancer** | **Azure Front Door** | **Traffic Manager** |
|-|:---:|:---:|:---:|:---:|
| Layer | 7 (HTTP/HTTPS) | 4 (TCP/UDP) | 7 (global CDN+LB) | DNS-based (global) |
| WAF | ✅ | ❌ | ✅ | ❌ |
| SSL termination | ✅ | ❌ | ✅ | ❌ |
| Global routing | ❌ | ❌ | ✅ | ✅ |
| Best for | Regional web apps | Internal/external L4 | Global web apps, CDN | Multi-region DNS failover |

### Azure DNS
- Host DNS zones in Azure for tight integration with other Azure services
- Use **Private DNS Zones** for name resolution within VNets (required with Private Endpoints)
- Supports **Alias Records** to point to Azure resources (Traffic Manager, Front Door, Public IPs) with automatic IP tracking

---

## Security

### Microsoft Entra ID (formerly Azure AD) Best Practices
- **Least privilege** — use Azure RBAC; assign roles at the narrowest scope (resource > resource group > subscription)
- Use **Managed Identities** for Azure services (System-assigned or User-assigned) — no credentials to manage
- Enable **MFA** and **Conditional Access Policies** for all human users
- Use **Privileged Identity Management (PIM)** for just-in-time privileged access with approval workflows
- Regularly review with **Access Reviews** and **Entra ID Recommendations**
- **App Registrations** + **Service Principals** for external app access; prefer federated credentials (OIDC) over client secrets

### Secrets Management
- Use **Azure Key Vault** for secrets, keys, and certificates (automatic rotation supported)
- Access Key Vault via **Managed Identity** — never store vault credentials in app config
- Enable **Soft Delete** and **Purge Protection** to guard against accidental or malicious deletion
- Use **Key Vault References** in App Service / Azure Functions to inject secrets at runtime without code changes

### Encryption
- **At rest**: all Azure storage services encrypt by default with Microsoft-managed keys; use **Customer-Managed Keys (CMK)** in Key Vault for compliance requirements
- **In transit**: enforce TLS 1.2+ via Azure Application Gateway, Front Door, or App Service TLS policies
- **Azure Disk Encryption** — encrypts VM OS and data disks using BitLocker (Windows) / dm-crypt (Linux), keys stored in Key Vault

### Defender for Cloud
- Unified security posture management (CSPM) and workload protection (CWPP)
- **Secure Score** — quantifies security posture; follow recommendations to improve
- Enable **Microsoft Defender plans** for VMs, Storage, SQL, Containers, Key Vault, and DNS
- Use **Just-In-Time (JIT) VM Access** to lock down management ports (RDP/SSH) and open them on-demand

---

## Messaging & Events

| Service | Pattern | Use Case |
|---------|---------|----------|
| **Service Bus** | Queue / Topic (pub/sub) | Reliable enterprise messaging, ordered delivery, sessions |
| **Event Grid** | Event routing (push) | React to Azure resource changes, serverless event routing |
| **Event Hubs** | Event streaming | High-throughput ingestion, telemetry, Kafka-compatible |
| **Storage Queues** | Simple queue | Lightweight task queuing, very high volume at low cost |
| **Azure Web PubSub** | WebSocket pub/sub | Real-time bidirectional communication |

**Service Bus vs Event Grid vs Event Hubs**

| | Service Bus | Event Grid | Event Hubs |
|-|:---:|:---:|:---:|
| Message ordering | ✅ (sessions) | ❌ | Per-partition |
| Replay | ❌ | ❌ | ✅ (retention up to 90 days) |
| Pull vs Push | Pull | Push | Pull |
| Max message size | 256 KB / 100 MB (Premium) | 1 MB | 1 MB |
| Kafka-compatible | ❌ | ❌ | ✅ |

**Service Bus Dead Letter Queue (DLQ)**: messages that exceed max delivery count or TTL are moved automatically — monitor DLQ depth as an operational metric.

---

## Cost Optimization Tips

1. Right-size VMs using **Azure Advisor** cost recommendations
2. Use **Azure Reservations** (1- or 3-year) for predictable workloads — up to 72% savings on VMs, SQL, Cosmos DB
3. Use **Azure Hybrid Benefit** to apply existing Windows Server / SQL Server licenses to Azure VMs (up to 40% savings)
4. Enable **auto-shutdown** for dev/test VMs; use **Dev/Test subscriptions** for discounted rates
5. Use **Spot VMs** for fault-tolerant batch workloads
6. Use **Azure Cost Management + Budgets** with alerts; tag all resources for cost allocation
7. Use **Private Endpoints** to avoid data egress charges for traffic to PaaS services
8. Use **Azure Container Apps** or **Functions (Consumption)** to pay only for actual execution — no idle compute

---

## Azure Well-Architected Framework — 5 Pillars

| Pillar | Key Questions |
|--------|--------------|
| **Reliability** | How do you design for failure? Target SLOs? Recover from outages? |
| **Security** | How do you protect data? Manage identities? Detect and respond to threats? |
| **Cost Optimization** | Aware of spend? Right pricing model? Eliminate waste? |
| **Operational Excellence** | How do you deploy safely? Observe system health? Learn from incidents? |
| **Performance Efficiency** | Right SKUs and tiers? Autoscaling configured? Bottlenecks identified? |

> Azure also has a **Sustainability** pillar and the [Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/) includes guidance for each.

---

## AWS → Azure Service Mapping (Quick Reference)

| Category | AWS | Azure Equivalent |
|----------|-----|-----------------|
| VM | EC2 | Azure Virtual Machines |
| Serverless functions | Lambda | Azure Functions |
| Container orchestration | EKS | AKS |
| Managed containers | ECS (Fargate) | Container Apps |
| Object storage | S3 | Azure Blob Storage |
| Relational DB | RDS / Aurora | Azure SQL / SQL MI |
| NoSQL DB | DynamoDB | Cosmos DB |
| In-memory cache | ElastiCache | Azure Cache for Redis |
| Virtual network | VPC | Virtual Network (VNet) |
| L7 load balancer | ALB | Application Gateway |
| Global L7 + CDN | CloudFront + ALB | Azure Front Door |
| DNS | Route 53 | Azure DNS |
| Identity & access | IAM | Microsoft Entra ID + Azure RBAC |
| Secrets management | Secrets Manager | Azure Key Vault |
| Message queue | SQS | Service Bus (Queue) |
| Pub/sub messaging | SNS | Service Bus (Topic) / Event Grid |
| Event streaming | Kinesis / MSK | Event Hubs |
| Security posture | Security Hub | Defender for Cloud |
| Monitoring & logs | CloudWatch | Azure Monitor + Log Analytics |
| Infrastructure as Code | CloudFormation | ARM Templates / Bicep |

---

## References

- [Azure Well-Architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/)
- [Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/)
- [Azure Charts (service comparison tool)](https://azurecharts.com/)
- [Microsoft Azure YouTube Channel](https://www.youtube.com/@MicrosoftAzure)
- Book: *Microsoft Azure Infrastructure Services for Architects* — John Savill

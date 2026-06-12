# AWS Reference — Services & Patterns

> Tags: #scalability #reliability #security

A curated reference for commonly used AWS services, best practices, and architecture patterns.

---

## Compute

### EC2
- Use **Auto Scaling Groups** (ASG) with **Launch Templates** (not Launch Configurations)
- Prefer **Spot Instances** for stateless, fault-tolerant workloads (up to 90% cost savings)
- Use **Graviton (ARM)** instances for ~20% better price/performance on most workloads
- Lifecycle hooks allow custom actions on scale-in/scale-out events

### Lambda
- Max execution: 15 min | Max memory: 10 GB | Max package: 250 MB (unzipped)
- **Cold start mitigation**: Provisioned Concurrency, SnapStart (Java), keep-warm pings
- Use **Lambda Layers** for shared dependencies
- Prefer **ARM64 (Graviton2)** — ~20% cheaper, ~19% faster for most runtimes
- Set `POWERTOOLS_LOG_LEVEL` and use AWS Lambda Powertools for structured logging

### ECS vs EKS

| | ECS (Fargate) | EKS |
|-|:---:|:---:|
| Operational complexity | Low | High |
| Kubernetes ecosystem | ❌ | ✅ |
| Cost at small scale | Lower | Higher |
| Portability | AWS-only | Multi-cloud |

---

## Storage

### S3
- **Storage Classes**: Standard → Standard-IA → Glacier Instant → Glacier Flexible → Deep Archive
- Enable **Intelligent-Tiering** for unpredictable access patterns (automatic cost optimization)
- Use **S3 Transfer Acceleration** for global uploads
- **Lifecycle Policies** — transition or expire objects automatically
- Enable **Versioning** + **MFA Delete** for critical buckets
- Block all public access by default; use **pre-signed URLs** for temporary access

### RDS / Aurora
- **Aurora** is 5x MySQL / 3x Postgres performance at similar cost
- **Aurora Serverless v2** scales in fine-grained increments (0.5 ACU steps)
- Use **RDS Proxy** to pool Lambda connections and avoid connection exhaustion
- Enable **automated backups** (1–35 day retention) and **point-in-time recovery**
- Use **Multi-AZ** for production; **Read Replicas** for read scaling

### DynamoDB
- Design access patterns first, schema second
- **Partition key** must distribute load evenly; avoid hot partitions
- **Global Secondary Indexes (GSI)** for alternate access patterns
- Use **TTL** for automatic expiry of session/cache data
- **DynamoDB Streams** + Lambda for change data capture
- **On-Demand** vs **Provisioned**: on-demand for unpredictable spikes, provisioned + Auto Scaling for stable workloads

---

## Networking

### VPC Design (Best Practice)
```
VPC (e.g., 10.0.0.0/16)
├── Public Subnets   (10.0.0.0/20 per AZ)  — ALB, NAT Gateway, Bastion
├── Private Subnets  (10.0.16.0/20 per AZ) — App servers, ECS tasks
└── Isolated Subnets (10.0.32.0/20 per AZ) — Databases, ElastiCache
```

- Deploy across **3 AZs** minimum for production
- Use **NAT Gateway** (per-AZ for HA) for outbound internet from private subnets
- Use **VPC Endpoints** for S3, DynamoDB, SSM to avoid data transfer costs and improve security
- **Security Groups**: stateful, prefer deny-by-default + allowlisting
- **NACLs**: stateless, use for subnet-level broad rules

### Load Balancing

| | ALB | NLB | CLB |
|-|:---:|:---:|:---:|
| Layer | 7 (HTTP/HTTPS) | 4 (TCP/UDP/TLS) | 4/7 (legacy) |
| WebSocket | ✅ | ✅ | ❌ |
| gRPC | ✅ | ✅ | ❌ |
| Static IP | ❌ | ✅ | ❌ |
| Use case | Web apps, APIs | High-perf, static IP | Avoid (legacy) |

### Route 53
- **Routing Policies**: Simple, Weighted, Latency-based, Failover, Geolocation, Geoproximity, IP-based
- Use **Health Checks** with failover routing for active-passive DR
- **Private Hosted Zones** for internal DNS within VPC

---

## Security

### IAM Best Practices
- **Least privilege** — grant minimum permissions needed
- Use **IAM Roles** for services (never embed access keys in code)
- Enable **MFA** for all human users
- Use **Permission Boundaries** to limit delegated role creation
- Regularly audit with **IAM Access Analyzer** and **Credential Reports**
- Prefer **AWS Managed Policies** as a baseline; narrow with inline policies

### Secrets Management
- Use **AWS Secrets Manager** for DB credentials, API keys (automatic rotation)
- Use **Parameter Store** (SSM) for config values (free tier for standard parameters)
- Never store secrets in environment variables directly — reference Secrets Manager ARN

### Encryption
- **At rest**: SSE-S3, SSE-KMS (customer managed keys via AWS KMS), or client-side
- **In transit**: enforce TLS 1.2+ via ACM certificates + HTTPS listeners
- **KMS**: use CMKs for audit trail (`CloudTrail` logs all key usage)

---

## Messaging & Events

| Service | Pattern | Use Case |
|---------|---------|----------|
| **SQS** | Queue (P2P) | Task distribution, decoupling |
| **SNS** | Pub/Sub (fan-out) | Broadcast to multiple subscribers |
| **EventBridge** | Event bus (routing) | Cross-service event routing, SaaS integration |
| **Kinesis Data Streams** | Event streaming | Real-time analytics, ordered stream |
| **MSK (Kafka)** | Event streaming | High-throughput, replay, ecosystem |

**SNS + SQS Fan-out Pattern**: SNS topic → multiple SQS queues → multiple consumers (each at their own pace, isolated failures)

---

## Cost Optimization Tips

1. Right-size EC2 instances using **Compute Optimizer** recommendations
2. Use **Savings Plans** (Compute or EC2) for predictable workloads (up to 66% savings)
3. Use **S3 Intelligent-Tiering** for objects > 128 KB with unknown access patterns
4. Use **Spot Instances** + **On-Demand** mix in ASGs
5. Set **billing alerts** via CloudWatch and **AWS Budgets**
6. Use **VPC Endpoints** to eliminate NAT Gateway data processing charges for AWS services
7. Enable **S3 requester pays** for shared datasets accessed externally

---

## Well-Architected Framework — 6 Pillars

| Pillar | Key Questions |
|--------|--------------|
| **Operational Excellence** | How do you deploy? Respond to events? Learn from failures? |
| **Security** | How do you protect data? Manage identities? Detect incidents? |
| **Reliability** | How do you recover from failures? Manage change? Scale? |
| **Performance Efficiency** | Right resource types? Monitor performance? Evolve with technology? |
| **Cost Optimization** | Aware of spend? Optimize over time? Right pricing model? |
| **Sustainability** | Minimize environmental impact? Maximize utilization? |

---

## References

- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [AWS re:Invent talks on YouTube](https://www.youtube.com/@AWSEventsChannel)
- Book: *AWS Certified Solutions Architect Study Guide* — Ben Piper & David Clinton

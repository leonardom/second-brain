# Event-Driven Architecture (EDA)

> Tags: #scalability #reliability #tradeoffs

Event-Driven Architecture (EDA) is a software design pattern where services communicate by producing, routing, and consuming **events** — immutable records of something that happened.

---

## Core Concepts

| Concept | Description |
|---------|-------------|
| **Event** | An immutable fact: "OrderPlaced", "PaymentProcessed" |
| **Producer** | The service that emits the event |
| **Consumer** | The service that reacts to the event |
| **Event Broker** | Infrastructure that routes events (Kafka, SQS, EventBridge) |
| **Event Schema** | Contract defining event structure (CloudEvents, Avro, Protobuf) |

---

## Event Types

### Domain Events
Record business facts in the past tense. They are part of the domain model.
```
OrderPlaced { orderId, customerId, items, totalAmount, placedAt }
PaymentFailed { paymentId, orderId, reason, failedAt }
```

### Integration Events
Cross-service notifications. Decouple bounded contexts.
```
UserAccountCreated { userId, email, createdAt }
```

### Commands (not events, but related)
A request for something to happen — directed to a specific service.
```
ProcessPayment { orderId, amount, paymentMethod }
```

---

## Messaging Patterns

### Pub/Sub
- Producer publishes to a **topic**; multiple consumers subscribe independently
- Each consumer gets its own copy of the message
- Best for: broadcast notifications, fan-out scenarios
- Examples: Kafka topics, SNS, Google Pub/Sub

### Point-to-Point (Queue)
- Message is delivered to **one** consumer
- Best for: task distribution, work queues
- Examples: SQS, RabbitMQ queues

### Event Streaming
- Events are stored durably and can be **replayed**
- Consumers maintain their own offset/position
- Best for: event sourcing, audit logs, stream processing
- Examples: Apache Kafka, AWS Kinesis

---

## Event Sourcing

Store the **sequence of events** as the source of truth instead of current state.

```
Events:
  AccountOpened   { accountId, owner, openedAt }
  MoneyDeposited  { accountId, amount, depositedAt }
  MoneyWithdrawn  { accountId, amount, withdrawnAt }

Current State = fold(all events for accountId)
```

**Pros:**
- Full audit trail
- Ability to replay and rebuild state
- Enables temporal queries ("what was the balance on date X?")

**Cons:**
- Eventual consistency — snapshots needed as event log grows
- Schema evolution is harder (old events must remain valid)
- Higher read complexity (always fold or use a projection)

---

## CQRS (Command Query Responsibility Segregation)

Often paired with event sourcing. Separate the write side (commands) from the read side (queries).

```
Write Side:  Command → Aggregate → Domain Event → Event Store
Read Side:   Event → Projector → Read Model (optimized for queries)
```

**Benefits:**
- Read models can be denormalized for performance
- Read and write sides scale independently
- Multiple read models for different use cases

---

## Broker Comparison

| Feature | Kafka | AWS SQS | AWS EventBridge | RabbitMQ |
|---------|:-----:|:-------:|:---------------:|:--------:|
| Message retention | Days–forever | 14 days | N/A (route only) | Until consumed |
| Replay | ✅ | ❌ | ❌ | ❌ |
| Ordering | Per-partition | FIFO queue only | ❌ | Per-queue |
| Throughput | Very high | High | Medium | Medium |
| Managed (AWS) | MSK | Native | Native | Amazon MQ |

---

## Designing for Reliability

### At-Least-Once Delivery
- Messages may be delivered more than once
- **Consumers must be idempotent** — processing the same event twice must have the same effect
- Use a deduplication key (e.g., `eventId`) stored in a processed-events table

### Dead Letter Queue (DLQ)
- Messages that fail after N retries are moved to a DLQ
- Monitor DLQ size as an operational metric
- Replay from DLQ after the bug is fixed

### Ordering
- Kafka guarantees ordering within a partition — use a consistent partition key (e.g., `orderId`)
- Avoid global ordering — it limits throughput drastically

### Outbox Pattern
Prevent the dual-write problem (DB write + message publish are not atomic):

1. Write the event to an **outbox table** in the same DB transaction as the domain change
2. A background process (or CDC like Debezium) reads the outbox and publishes to the broker
3. Mark events as published after successful delivery

---

## Schema Evolution

Use a schema registry to manage compatibility:

- **Backward compatible** — new consumers can read old messages (add optional fields)
- **Forward compatible** — old consumers can read new messages (remove optional fields)
- **Full compatible** — both directions

Tools: Confluent Schema Registry (Avro), AWS Glue Schema Registry, Protobuf with reserved fields

---

## Common Pitfalls

- **Event spaghetti** — undocumented event flows; use an event catalog
- **Fat events** — events carrying full entity state; prefer thin events + event sourcing
- **Tight event coupling** — consumers depending on internal fields; version events explicitly
- **Missing idempotency** — leads to double processing of orders, payments, etc.
- **No observability** — instrument producer latency, consumer lag, DLQ depth

---

## References

- [Martin Fowler — Event-Driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)
- [CloudEvents Specification](https://cloudevents.io/)
- [Kafka Documentation](https://kafka.apache.org/documentation/)
- Book: *Designing Event-Driven Systems* — Ben Stopford (free PDF from Confluent)
- Book: *Implementing Domain-Driven Design* — Vaughn Vernon

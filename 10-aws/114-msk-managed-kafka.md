<!-- #genai: AWS interview note - Amazon MSK managed Kafka, brokers, partitions, replication, consumer groups, provisioned vs serverless for the cloud round -->

## Question
Explain Amazon MSK (Managed Streaming for Apache Kafka) — core Kafka concepts and what AWS manages for you.

### Short Explanation
MSK is **managed Apache Kafka**: AWS runs the brokers, ZooKeeper/KRaft, patching, and multi-AZ replication, so you keep the Kafka API without operating the cluster. Interviewers check that you understand **topics/partitions/consumer groups**, **replication**, and **when Kafka fits** vs SQS/SNS/Kinesis.

## Answer
- Kafka is a **distributed, durable, append-only log** for high-throughput event streaming; producers write to **topics** split into **partitions**, and **consumer groups** read them.
- **MSK** removes the ops burden: provisions **brokers across AZs**, handles replication, patching, and monitoring; you connect with standard Kafka clients.
- Choose **MSK Provisioned** for control/tuning, **MSK Serverless** when you want to skip capacity planning.

## Detailed Explanation

### Core Kafka concepts
| Concept | Meaning |
|---|---|
| **Topic** | Named stream of records |
| **Partition** | Ordered, append-only shard of a topic — the unit of **parallelism** |
| **Offset** | Position of a record within a partition |
| **Producer** | Writes records (optionally keyed → same key lands on same partition) |
| **Consumer group** | Set of consumers sharing the load; each partition is read by **one** consumer in the group |
| **Replication factor** | Copies of each partition across brokers for durability |
| **Retention** | Records kept for a time/size window regardless of consumption |

Ordering is guaranteed **per partition**, not per topic. Parallelism is bounded by partition count.

### What MSK manages
- Provisions and spreads **brokers across AZs**, with **replication** for durability/HA.
- Handles broker patching, ZooKeeper/**KRaft**, storage autoscaling, and **CloudWatch/Prometheus** metrics.
- Integrations: **IAM auth / SASL-SCRAM / TLS**, encryption at rest (KMS), **MSK Connect** (managed Kafka Connect), and schema registry (Glue).

### Provisioned vs Serverless
| | Provisioned | Serverless |
|---|---|---|
| Capacity | You pick broker type/count | Auto-scales |
| Tuning | Full broker config control | Limited |
| Cost model | Per broker-hour | Per throughput/storage |
| Use when | Steady, high, tunable load | Spiky/unknown load, less ops |

### Kafka vs SQS/SNS/Kinesis (common follow-up)
- **Kafka/MSK**: replayable log, high throughput, multiple independent consumer groups, ordering per partition.
- **SQS**: simple queue, one logical consumer set, no replay.
- **SNS**: pub/sub fan-out, push.
- **Kinesis**: AWS-native streaming, similar model, fully serverless — pick it if you don't need the Kafka ecosystem.

### Things they probe
- **Rebalancing**: adding/removing consumers reassigns partitions → brief pause; keep partition counts sane.
- **Hot partitions** from a bad partition key skew load — choose keys carefully.
- Consumers commit **offsets**; at-least-once by default → make consumers **idempotent**.

## Summary
- MSK = managed Kafka: **brokers across AZs, replication, patching, IAM/TLS auth** — you keep the Kafka API.
- Know **topic → partitions → consumer groups**, ordering **per partition**, and **replication factor** for durability.
- **Provisioned** for control, **Serverless** for hands-off scaling; pick Kafka over SQS/SNS when you need **replay, high throughput, and multiple consumer groups**.

## Related
- [Core services, secrets & cost](./108-core-services-secrets-and-cost.md)
- [Service communication](./110-service-communication.md)

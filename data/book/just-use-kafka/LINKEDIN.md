Today's Spec-Driven System Design Interview: Just Use Kafka — Kafka-Centric Systems Design.

The lesson is not "put Kafka everywhere."

It is learning when a partitioned, replicated, append-only log is the right backbone: producers publish once, consumers read independently, slow services stop blocking the write path, and replay becomes a design primitive.

The walkthrough starts with service calls, polling jobs, direct database reads, and synchronous chains that grow with every new consumer.

Then it rebuilds around Kafka's canonical use cases: log aggregation, pub/sub microservices, stream processing, CDC/event sourcing, delivery semantics, and production operations.

The value is the boundary work. Kafka is not just a broker in the diagram. It forces explicit choices about partition ordering, consumer groups, retention, schema compatibility, idempotent producers, transactions, state stores, lag, DLQs, governance, and disaster recovery.

Modern choices make those trade-offs concrete: Apache Kafka, Redpanda, Pulsar, or NATS JetStream for the event backbone; MSK, Kinesis, Pub/Sub, or Event Hubs when managed services fit; Kafka Streams or Flink for stateful processing; Debezium and Kafka Connect for CDC.

Managed services can remove broker ops. They do not remove partition discipline, schema compatibility, lag SLOs, idempotent side effects, replay policy, or the question that matters: where does the authoritative commit happen?

Try the interactive walkthrough:
https://zeljkoobrenovic.github.io/system-design-interviews/book/interview.html#just-use-kafka

Explore the project/book catalog:
https://zeljkoobrenovic.github.io/system-design-interviews/book/index.html

Free source code:
https://github.com/zeljkoobrenovic/system-design-interviews

#SystemDesign #SystemDesignInterview #SoftwareArchitecture #DistributedSystems #Kafka

Today's Spec-Driven System Design Interview: Just Use Redis — Redis-Centric Systems Design!

The lesson is not "put everything in Redis." It is learning which Redis data structure belongs on which pressure point, and where Redis must stop being the source of truth.

This walkthrough starts with the boring baseline: one relational database for sessions, carts, product reads, counters, rankings, and checkout side effects. Then each bottleneck maps to a structure: Strings for sessions and cache-aside reads, atomic INCR for view counters and rate limits, Hashes for per-user carts, Sorted Sets for trending products, Lists/Streams for async work, and Bitmaps for retention.

The sharp edge is correctness. Redis can make the hot path fast, but committed orders, idempotency keys, order_items, and the transactional outbox still belong in the durable database. A Redis lock can reduce duplicate work; it cannot be the boundary that proves exactly-once order creation. That distinction is where many interview answers get stronger.

The case also connects the fundamentals to modern choices: Redis/Valkey/Dragonfly or managed Redis for hot state, SQS/Pub/Sub/Kafka/Redis Streams for delivery semantics, a managed relational database for RPO 0 order records, and observability around p99, hit rate, evictions, queue age, retries, AOF fsync latency, replica lag, and failover.

Try the interactive walkthrough:
https://zeljkoobrenovic.github.io/spec-driven-system-design-interviews/book/interview.html#just-use-redis

Explore the project/book catalog:
https://zeljkoobrenovic.github.io/spec-driven-system-design-interviews/book/index.html

Free source code:
https://github.com/zeljkoobrenovic/spec-driven-system-design-interviews

#SystemDesign #SystemDesignInterview #SoftwareArchitecture #Redis #Scalability

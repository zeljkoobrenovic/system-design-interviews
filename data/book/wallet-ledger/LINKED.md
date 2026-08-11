Today's Spec-Driven System Design Interview: Wallet / Ledger — System Design.

The naive design is one balance column per account: read, subtract, write back. It is simple, fast, and wrong for money in three ways — there is no history to audit, two concurrent transfers can silently lose an update, and nothing exists to reconcile against.

This interview walks the fix step by step: model every movement as balanced double-entry postings in an append-only journal, make transfers idempotent (scoped key plus request hash, committed in the same transaction as the posting), materialize balances for O(1) reads, guard them with concurrency control, publish downstream events through a transactional outbox, and reconcile continuously. When a transfer spans two shards, a saga over per-shard clearing accounts keeps every leg balanced.

At ~10k transactions per second (about 6 durable rows each), these fundamentals are what keep money conserved. The same moves map onto today's stacks: PostgreSQL, CockroachDB, Aurora, or Spanner for the ledger; Kafka, Kinesis, or Pub/Sub behind the outbox; Temporal or Step Functions for the cross-shard saga. The tools change the operations story, not the invariants.

Try the interactive walkthrough:
https://zeljkoobrenovic.github.io/spec-driven-system-design-interviews/book/interview.html#wallet-ledger

Explore the full catalog:
https://zeljkoobrenovic.github.io/spec-driven-system-design-interviews/book/index.html

Free source code:
https://github.com/zeljkoobrenovic/spec-driven-system-design-interviews

#SystemDesign #SystemDesignInterview #SoftwareArchitecture #DistributedSystems #Fintech

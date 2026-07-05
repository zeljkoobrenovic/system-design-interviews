# Review: Kafka Use Cases - Event Streaming Platform Design

Reviewed file: `data/book/kafka-use-cases/interview.json`  
Review date: 2026-07-05

## Executive Summary

This is a strong book-style Kafka walkthrough. The dataset has a clear teaching
spine: start from point-to-point coupling, introduce the partitioned replicated
commit log, then reuse that one backbone for ingestion, pub/sub, stream
processing, CDC/event sourcing, delivery semantics, and platform operations.
The current file has also addressed several concerns that an earlier review
called out: CDC is now separated from event sourcing, exactly-once is scoped to
Kafka processing effects, retry/DLQ appears in the main flow, and Step 7 now
covers governance, quotas, schemas, DR, tiered storage, and lag-based
operations.

The remaining issues are mostly integration and polish, not conceptual
breakers. The highest-impact problem is renderer-facing: several structured
views reference links whose endpoints are not present in the view's node list,
so the diagrams can show implicit untyped nodes or confusing edges. The
second-largest gap is wrap-up coverage: the requirements list now includes
latency, operability, DR, and governance/security, but `satisfies` does not yet
prove those requirements in the Design vs. Requirements section.

| Axis | Rating | Notes |
| --- | ---: | --- |
| System design soundness | 4.4 / 5 | Kafka mechanics, CDC boundaries, EOS scope, replication, retention, and sizing are mostly accurate. |
| Production realism | 4.1 / 5 | Stronger than a basic Kafka tour; platform metadata, operational workflows, and SLO specificity can still be made more explicit. |
| Pedagogical flow | 4.5 / 5 | The step sequence is coherent and teaches one new Kafka use case at a time. |
| Step-to-final coherence | 4.3 / 5 | Final design includes the introduced components; some wrap-up requirement mappings lag behind the expanded design. |
| Dataset/rendering fit | 3.8 / 5 | JSON parses and most references resolve, but four view-link endpoint mismatches should be fixed before publishing. |
| Overall | 4.2 / 5 | A credible Kafka "golden hammer" chapter with a few concrete fixes needed for rendering and production-grade completeness. |

## What Works Well

- The "one log, many use cases" framing is effective. It prevents the chapter
  from becoming an unstructured list of Kafka features.
- Step 1 explicitly says Kafka is not a replacement for every synchronous RPC,
  which is important for a Golden Hammers chapter.
- Step 2 explains the producer client/library, partitions, retention caveat,
  idempotent producer, batching, compression, and schema registry at the right
  point.
- Step 3 has a good consumer-group explanation, especially the distinction
  between more members in one group and a new group for another workload.
- Step 4 teaches stateful stream processing instead of treating every consumer
  as a simple event handler. Event time, watermarks, changelog-backed state, and
  state-size limits are all present.
- Step 5 now makes the important distinction: CDC keeps the DB authoritative,
  while event sourcing makes the event append authoritative.
- Step 6 now has the right exactly-once caveat: Kafka transactions give
  exactly-once processing effects within Kafka, while external side effects need
  idempotency keys or equivalent dedupe.
- Step 7 is much more production-realistic than a typical interview answer:
  partitions, replication, ISR, KRaft, tiered storage, quotas, ACLs, schema
  compatibility, audit, DR, RPO/RTO, and lag SLOs are all named.

## Highest-Impact Issues

### 1. Several step and option diagrams have hidden link endpoints

The structured `view.links` fields reference high-level links whose endpoints
are not included in the same view's `nodes`. The renderer builds diagrams from
the selected nodes and links, so this can produce implicit Mermaid nodes with
default labels/shapes or edges that do not match the intended architecture.

Current mismatches found by reference checking:

- Step `pubsub`, option `Orchestrated / RPC`: link `prod-source`
  (`Producers -> SourceDB`) is included, but `SourceDB` is not in the option's
  `view.nodes`.
- Step `stream`: link `cons-serving` (`Consumers -> Serving`) is included, but
  `Consumers` is not in `view.nodes`.
- Step `cdc`, option `Application dual-write`: links `prod-gw`
  (`Producers -> Gateway`) and `gw-broker` (`Gateway -> Broker`) are included,
  but `Gateway` is not in the option's `view.nodes`.

Concrete fix: either add the missing endpoint nodes to those views or choose
links whose endpoints are already visible. For the stream step, the likely fix
is to use a link that reflects `Broker -> Serving` for aggregate output, or add
`Consumers` if that edge is intentionally showing a sink consumer. For the dual
write option, add `Gateway` because the option is explicitly showing an app
writing both the DB and Kafka.

### 2. `satisfies` no longer covers all declared non-functional requirements

The `requirements.nonFunctional` list includes nine requirements, including:
low latency/freshness, operability/consumer lag, availability + DR, and
governance + security. The `satisfies.nonFunctional` wrap-up currently covers
only five: throughput, durability/order, decoupling, retention/replay, and
horizontal scale.

Why it matters: the rendered "Design vs. Requirements" section is supposed to
close the loop for the interview. Right now the dataset declares important
platform requirements in the opening, explains many of them in Step 7, but does
not prove them in the wrap-up.

Concrete fix: add `satisfies.nonFunctional` items for:

- Low latency and bounded freshness, tied to batching choices, partitioning,
  consumer lag, and Step `scale-ops`.
- Operability, tied to lag, under-replicated partitions, DLQ depth, quotas,
  schema compatibility failures, and Step `scale-ops`.
- Availability + DR, tied to RF=3, ISR, KRaft, cross-region replication,
  tiered-storage restore, and Step `scale-ops`.
- Governance + security, tied to `ControlPlane`, ACLs, auth, quotas, audit,
  schemas, connector approval, and Step `scale-ops`.

### 3. The platform data model is thinner than the platform design

The data model currently covers `topic`, `record`, and `consumer group`. That
is enough for Kafka basics, but the chapter now frames the target as a
company-wide platform with schemas, quotas, ACLs, connector approvals, ISR,
retention classes, offset resets, and DR.

Concrete additions that would support the later steps:

- `schema_subject` / `schema_version`: subject, version, compatibility mode,
  owner, status.
- `topic_policy`: owner, retention class, partition count, RF, compaction flag,
  PII classification, quota class.
- `acl_or_quota`: principal, topic/resource, operation, limit, audit metadata.
- `partition_replica`: topic, partition, leader, replicas, ISR, high watermark.
- `connector_task`: source/sink, status, offsets/checkpoints, secret reference.

This does not need to become a full Kafka internals schema. A few concise
entities would make the admin/control-plane claims concrete and help the final
design feel like an operable platform rather than just a broker cluster.

### 4. The anti-golden-hammer lesson should be stronger in the wrap-up

The baseline step correctly says direct RPC is still right for request/response
reads and immediate consistent commands. That is good, but because this dataset
is placed in a Golden Hammers category, the final teaching point should more
explicitly say when Kafka is the wrong tool.

Useful additions:

- A trap or follow-up: "When would you not use Kafka?"
- Contrast with simpler queues for task distribution where replay and
  independent consumer groups are not needed.
- Contrast with RPC for synchronous command/query paths.
- Contrast with a database or object store for mutable queryable state.
- Contrast with Redis/pubsub or lightweight brokers for low-durability ephemeral
  notifications.

The goal is not to weaken Kafka. It is to help candidates avoid the common
mistake of using Kafka as a database, RPC layer, workflow engine, and queue for
every problem.

## System Design Soundness

The core design is sound. Producers and collector agents publish through a
Kafka producer client to a replicated broker cluster; schema registry protects
event compatibility; consumer groups decouple downstream services; stream
processors handle stateful aggregations and results topics; Kafka Connect and
CDC move changes and sinks; retry/DLQ topics handle poison records; and the
control plane governs topics, ACLs, quotas, schemas, and connectors.

The requirements are well aligned with Kafka's strengths: independent
producers/consumers, fan-out, replay, high throughput, per-partition ordering,
CDC, stream processing, and tunable delivery semantics. The low-latency,
operability, DR, and governance requirements are also appropriate, but they
need matching `satisfies` entries as noted above.

Capacity is directionally credible. The dataset derives `~1 GB/s` raw ingest
from `~1M events/s * ~1 KB`, estimates about `~100` partitions for a hot topic
at `~10 MB/s` per partition, accounts for RF=3 write amplification, and calls
out `~86 TB/day` raw plus `~259 TB/day` pre-compression replicated writes. That
is strong enough for an interview walkthrough. A next pass could make the
latency and freshness SLOs more concrete by defining example targets per topic
class, such as produce p99, end-to-end freshness, and max consumer lag.

The API section works as a conceptual Kafka API: `send`, `subscribe`, `poll`,
admin operations, and `commit`. The admin row is valuable because it keeps the
company-wide platform framing visible. If the data model grows, the admin API
could mirror it by naming topic policy, schema compatibility, ACL/quota, offset
reset, and connector lifecycle as first-class operations.

## Step-by-Step Pedagogical Review

### Step 1: Naive: Point-to-Point Integrations

This is a strong opening. It shows the N x M integration problem, cascading
failures, no replay, and analytics pressure on the operational DB. The step
also avoids a common overcorrection by saying synchronous calls are still right
when a command or query needs an immediate answer.

Suggested improvement: add a later wrap-up trap that returns to this point and
asks when Kafka should not be used.

### Step 2: Ingestion & Log Aggregation

This step introduces the durable log, partitions, retention, replay, schema
registry, and producer client clearly. The retention caveat is especially
important: offline consumers recover only if their downtime fits within
retention, compaction, or tiered-storage replay policy.

Suggested improvement: no structural change needed. Keep the wording that
`Gateway` is a producer client library, not a deployable hop.

### Step 3: Async Messaging - Event-Driven Microservices

The consumer-group explanation is accurate and teachable. The options compare a
real choice: event choreography versus orchestrated/RPC control. The retry/DLQ
trap is exactly the right operational correction for consumer groups.

Suggested improvement: fix the `Orchestrated / RPC` option view by adding
`SourceDB` or replacing `prod-source` with a link whose endpoints are already
visible.

### Step 4: Stream Processing - Real-Time Aggregations

This is one of the better steps. It distinguishes stream processing from simple
per-event consumption, introduces event time and watermarks, and calls out state
size as a practical limit. The Kafka Streams versus Flink comparison is compact
but useful.

Suggested improvement: fix the view link mismatch. The current `cons-serving`
link introduces a hidden `Consumers` endpoint in a diagram that otherwise shows
`Broker`, `Stream`, `Serving`, and `Metrics`.

### Step 5: CDC & Event Sourcing

The current version handles the source-of-truth boundary well. It explains CDC,
transactional outbox, application dual-write, event sourcing, and log compaction
without collapsing them into one pattern. The CDC sequence flow makes the DB
commit boundary visible.

Suggested improvement: fix the `Application dual-write` option by adding
`Gateway` to the view nodes or choosing a simpler direct app-to-broker link if
that is the intended visual.

### Step 6: Delivery Semantics

This step is strong. It distinguishes at-most-once, at-least-once, idempotent
producer, transactions, `read_committed`, offset-commit ordering, and external
side-effect idempotency. The caveat is honest: exactly-once stops at the Kafka
boundary unless the external sink participates through idempotency or a
transactional pattern.

Suggested improvement: consider adding a tiny example of an idempotent sink
schema, such as `(idempotency_key, effect_type, processed_at)`, if the data
model is expanded.

### Step 7: Scale & Operate

This is the right closing step. It now includes partition sizing, RF=3 write
amplification, ISR, KRaft, retention, compaction, tiered storage, lag,
under-replicated partitions, governance, auth/ACLs, quotas, PII classification,
audit, offset reset, and DR.

Suggested improvement: convert the strongest operational claims into
`satisfies.nonFunctional` items and, optionally, a few data model entities for
topic policy, ACL/quota, schema version, and replica state.

## Final Design Review

The final design integrates the steps well. It includes producers, collectors,
producer client, broker cluster, controller/metadata quorum, schema registry,
control plane, consumer services, stream processor, Kafka Connect/CDC, retry/DLQ
topics, source DB, warehouse, serving store, and monitoring. The final prose
correctly says that Kafka is a shared event backbone, not a literal solution to
every state or workflow problem.

The final design should carry two items more visibly:

- Design vs. Requirements coverage for the new platform-grade requirements:
  low latency/freshness, operability, DR, governance/security.
- Control-plane data contracts: topic ownership, retention class, schema
  compatibility, quotas, ACLs, connector approvals, and audit.

The final diagram itself is otherwise coherent once the individual view
endpoint mismatches are fixed.

## Concept Introduction and Learning Flow

The concept staging is a major strength:

- Step 1 creates the coupling and replay problem.
- Step 2 introduces the durable append-only log.
- Step 3 explains pub/sub fan-out and consumer groups.
- Step 4 introduces stateful processing over the log.
- Step 5 explains CDC, outbox, event sourcing, and compaction.
- Step 6 adds delivery semantics and correctness boundaries.
- Step 7 closes with scaling, governance, DR, and operations.

This is the right order for an interview. Candidates can build the design
incrementally rather than naming Kafka and then listing features. The only
pedagogical gap is the final anti-hammer message: make the "do not use Kafka
for everything" lesson a first-class wrap-up, not only an early caveat.

## Step-to-Final-Design Coherence

The steps map cleanly into `finalDesign`:

- Step 2 contributes `Producers`, `Ingest`, `Gateway`, `Broker`, `SchemaReg`,
  `Connect`, and `Warehouse`.
- Step 3 contributes `Consumers`, `Serving`, `DLQ`, and consumer lag metrics.
- Step 4 contributes `Stream` and the output-topic loop.
- Step 5 contributes `SourceDB`, CDC links, Connect, and rebuildable read
  models.
- Step 6 strengthens `Gateway`, `Broker`, `Stream`, `Consumers`, and `DLQ` with
  idempotence, transactions, offset commits, and poison-message handling.
- Step 7 brings in `Zoo`/KRaft metadata, replication, retention, control plane,
  security/governance, DR, and monitoring.

The coherence gaps are specific: the per-step views need endpoint fixes, and
the wrap-up `satisfies` table should be updated to reflect the production
concerns that Step 7 now teaches.

## Realism Compared With Production Systems

The dataset is realistic for a system-design interview. It covers many issues
that candidates often miss: retention limits on replay, hot-key/partition
parallelism, schema compatibility, consumer lag, poison messages, CDC source of
truth, transactional outbox, exactly-once scope, RF/ISR durability, tiered
storage, quotas, ACLs, audit, and non-transparent multi-region failover.

Further realism would come from making operational workflows explicit:

- How topic creation is approved and who owns retention/partition changes.
- How a breaking schema change is blocked, rolled forward, or rolled back.
- How a team safely resets offsets or replays a topic.
- How DLQ replay is authorized and traced after a fix.
- How failover handles duplicate/reordered records and client cutover.
- How PII topics affect retention, encryption, access review, and deletion
  obligations.

These can be short notes. The dataset already has the right components; the
remaining work is to expose the contracts an operator would actually run.

## Dataset and Renderer-Facing Observations

- `interview.json` parses as valid JSON.
- Top-level keys are coherent for the renderer: requirements, capacity, API,
  data model, patterns, steps, final design, satisfies, interview script, level
  variants, and follow-ups.
- `view.nodes` references resolve against `highLevelArchitecture.nodes`.
- `view.links` IDs resolve against `highLevelArchitecture.links`.
- Four view-link endpoint mismatches remain and should be fixed before
  publishing: `pubsub` option 2, `stream`, and `cdc` option 3 as listed above.
- `satisfies[*].steps[*]` references resolve to real step IDs.
- Dataset-level pattern names match `step.patterns`.
- Sequence participants and messages resolve cleanly.
- Canonical node types are mostly appropriate. `Gateway` is acceptable because
  the step text explains it as a producer client, but the label should stay
  explicit to avoid implying a central proxy service.
- `Zoo` is an old internal id for a node now labeled `Controller / Metadata
  (KRaft)`. The label is fine for users; renaming the id is optional and only
  worth doing if internal ids are shown in future UI.
- No generated docs rebuild is needed for this review-only edit, but if the
  Kafka dataset itself is committed as new source, the matching
  `docs/book/data/kafka-use-cases/interview.json` and book manifest should be
  committed with it.

## Recommended Edits, Prioritized

### P1: Fix the structured view endpoint mismatches

Add missing endpoint nodes or replace the offending links in `pubsub` option 2,
`stream`, and `cdc` option 3. This is the only concrete rendering risk found.

### P1: Complete `satisfies.nonFunctional`

Add rows for low latency/freshness, operability, availability + DR, and
governance + security so the wrap-up proves every declared non-functional
requirement.

### P2: Expand the data model just enough for a platform

Add concise entities for schema versions, topic policy, ACL/quota, partition
replica/ISR state, and connector tasks. Keep them compact; the point is to
support the platform claims, not teach Kafka internals exhaustively.

### P2: Strengthen the anti-golden-hammer lesson

Add a trap, follow-up, or wrap-up note that asks when Kafka is the wrong choice
and compares it with RPC, simple queues, databases/object stores, and ephemeral
pub/sub.

### P3: Add curated further-reading or technology-choice wrap-up material

For a mature book chapter, consider adding optional comparison material:
self-managed Kafka vs managed Kafka, Kafka vs Pulsar, Kafka vs cloud-native
streams, Kafka Streams vs Flink, and Kafka vs task queues. This is enrichment,
not a blocker.

## What Not To Change

- Keep the seven-step structure. It teaches the design in the right order.
- Keep the baseline point-to-point step; it makes Kafka's value concrete.
- Keep the CDC vs event-sourcing distinction; it is now one of the dataset's
  strongest correctness points.
- Keep the external-side-effect caveat in the delivery semantics step.
- Keep Step 7 as the production close; it is the right place for partitions,
  replication, retention, governance, and DR.

## Bottom Line

This is a strong Kafka use-case interview and a good fit for the Golden Hammers
group. The next pass should not restructure it. Fix the few structured diagram
mismatches, make the Design vs. Requirements wrap-up cover all declared
non-functional requirements, and add a sharper "when not to use Kafka" lesson so
the chapter teaches judgment as well as Kafka mechanics.

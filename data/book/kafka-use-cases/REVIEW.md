# Review: Kafka Use Cases - Event Streaming Platform Design

Reviewed file: `data/book/kafka-use-cases/interview.json`  
Review date: 2026-07-05

## Executive Summary

The current Kafka dataset is in much better shape than the previous review
described. The recent changes resolved the main concrete blockers: structured
diagram link endpoints now line up with visible nodes, the data model now
includes platform-control entities, `satisfies.nonFunctional` covers latency,
operability, DR, and governance, and Step 7 now carries an explicit
anti-golden-hammer trap and follow-up.

What remains is mostly semantic tightening. The interview is publishable as a
strong Kafka use-case walkthrough, but a final polish pass should make the
non-Kafka alternative in Step 3 visually honest, surface operational loops in
the final design, and update the script/level wrap-ups to match the now richer
platform scope.

| Axis | Rating | Notes |
| --- | ---: | --- |
| System design soundness | 4.6 / 5 | Strong Kafka mechanics, clear CDC/event-sourcing boundary, scoped exactly-once semantics, credible sizing, and good platform data model coverage. |
| Production realism | 4.4 / 5 | Governance, quotas, schemas, DR, lag, ISR, tiered storage, and DLQs are present; operational workflows can be made more procedural. |
| Pedagogical flow | 4.6 / 5 | The seven-step sequence teaches one new Kafka use case at a time and now closes with the "when not Kafka" lesson. |
| Step-to-final coherence | 4.5 / 5 | Final design includes the introduced components; a few operational feedback/replay loops should be more visible. |
| Dataset/rendering fit | 4.6 / 5 | JSON and references validate; the remaining issue is mostly one semantically weak option diagram, not broken rendering. |
| Overall | 4.5 / 5 | A strong book chapter candidate with a short list of targeted polish items. |

## What Works Well

- The "one partitioned, replicated commit log, many use cases" frame is
  effective. It keeps the chapter from becoming a loose Kafka feature tour.
- The baseline step avoids the golden-hammer mistake early: Kafka is for
  decoupled event flow, not every synchronous command or query.
- Ingestion is taught with the right caveats: producer client as a library,
  partition keys as ordering/parallelism units, retention as the basis for
  replay, and schema registry as compatibility guardrail.
- Consumer groups and pub/sub fan-out are explained clearly, including the
  difference between scaling one workload and adding a new independent workload.
- Stream processing is not hand-waved. The step covers event time, watermarks,
  changelog-backed state, stream-table joins, state-size limits, and Kafka
  Streams versus Flink trade-offs.
- CDC, transactional outbox, application dual-write, and event sourcing are now
  separated by their single commit point, which is the right correctness lens.
- Delivery semantics are honest: Kafka transactions give exactly-once
  processing effects within Kafka; external side effects still need
  idempotency.
- Scale and operations now feel like a real platform: RF/ISR, KRaft, tiered
  storage, quotas, ACLs, schema policy, PII classification, audit, DR, lag SLOs,
  and topic governance are all present.
- The expanded data model and `satisfies` section now support the platform
  story instead of stopping at topic/record/consumer-group basics.

## Highest-Impact Issues

### 1. The Step 3 "Orchestrated / RPC" alternative is endpoint-valid but visually weak

The previous endpoint mismatch is fixed: option 2 now includes `SourceDB`, so
`prod-source` resolves cleanly. However, the diagram still does not really show
an orchestrated or RPC workflow. It renders `Producers -> SourceDB` and
`Consumers -> Serving`, but the option prose says a central orchestrator calls
downstream services synchronously.

Why it matters: this is the main contrast against Kafka pub/sub. If the visual
does not show the synchronous call chain or orchestrator, candidates may not see
what failure mode Kafka is replacing.

Concrete fix: either add an explicit `Orchestrator`/`RPC API` node and links for
`Producers -> Orchestrator -> Consumers`, or rename the option/caption to match
the current direct-call/DB-mediated visual. Adding the node is stronger because
it teaches the trade-off directly.

### 2. The final design should expose operational feedback and replay loops

The final design includes `DLQ`, `Metrics`, `ControlPlane`, `SchemaReg`, KRaft,
Connect, stream processing, sinks, and the broker cluster. That is coherent, but
some of the operational behavior taught earlier becomes less visible in the
final view.

Concrete improvements:

- Add `dlq-broker` to the final design and Step 7 views so "replay after fix" is
  visible, not only the one-way consumer-to-DLQ path.
- Consider showing the Step 4 aggregate sink path in the final design with
  `broker-serving` if the intent is that result topics feed serving stores
  directly through a sink.
- Add a short final-design note for guarded workflows: topic request, schema
  change approval, offset reset/replay, DLQ replay, and DR failover rehearsal.

This is not a structural blocker; it is about making the final diagram preserve
the operational lessons the steps now teach.

### 3. The interview script and level variants lag behind the upgraded platform scope

The main dataset now includes governance, security, DR, topic ownership,
schema compatibility, quotas, PII classification, tiered storage, and SLOs. The
`interviewScript` and `levelVariants` still emphasize Kafka basics, use cases,
delivery semantics, partitions, replication, retention, and lag.

Concrete fix: update the final script phase and Senior expectations to mention
the platform-control-plane layer explicitly: ACLs/quotas, schema policy, topic
ownership, guarded offset reset/replay, and DR/RPO/RTO trade-offs. This keeps
the wrap-up aligned with Step 7 and the expanded non-functional requirements.

### 4. The platform-control pattern is not shown as a Step 7 tag

The dataset-level pattern `Platform control plane (governance)` points to
`scale-ops`, but Step 7's own `patterns` field is absent. The renderer can still
cross-link from the pattern entry to the step, but the step page will not show
the pattern tag alongside the concepts and traps.

Concrete fix: add:

```json
"patterns": ["Platform control plane (governance)"]
```

to Step `scale-ops`.

## System Design Soundness

The architecture is sound for an interview-level Kafka platform. Producers and
collector agents append through the Kafka producer client; brokers provide the
partitioned, replicated log; schema registry enforces compatibility; consumer
groups and stream processors read independently; Kafka Connect and CDC bridge
databases and sinks; retry/DLQ topics isolate poison records; and a control
plane governs shared-platform usage.

The requirements are now well closed by the design. Functional coverage maps
cleanly to ingestion, fan-out, stream processing, CDC/event sourcing, and
delivery semantics. Non-functional coverage now includes throughput, durability,
decoupling, replay, scale, low latency/freshness, operability, availability/DR,
and governance/security.

Capacity math is credible: about 1M events/s at 1 KB gives about 1 GB/s raw
ingest; a hot topic at about 10 MB/s per partition lands around 100 partitions;
RF=3 turns that into about 3 GB/s of broker writes before compression; and
tiered storage plus compression makes days-to-weeks hot retention plausible.
A later polish pass could add example topic classes, such as "payments events"
versus "clickstream", with different retention, lag, and latency SLOs.

The API section works as a conceptual Kafka API. The admin row is especially
useful because the dataset is no longer just a broker walkthrough; it is a
shared event-streaming platform. If more detail is desired, the admin API could
name separate operations for topic policy, schema compatibility changes, ACLs,
quota changes, connector lifecycle, and governed offset resets.

The data model is now strong enough for the platform claims. In addition to
`topic`, `record`, and `consumer group`, it includes `schema_version`,
`topic_policy`, `acl_or_quota`, `partition_replica`, and `connector_task`. That
is the right level of abstraction: concrete enough to support governance and
operations, without turning the interview into Kafka internals documentation.

## Step-by-Step Pedagogical Review

### Step 1: Naive: Point-to-Point Integrations

This is a strong opening. It makes the N x M integration problem concrete,
shows cascading failure and no replay, and explicitly says synchronous RPC still
belongs on request/response paths that need immediate consistency.

No major change needed. Keep this as the contrast that makes the log useful.

### Step 2: Ingestion & Log Aggregation

This step introduces the durable append-only log cleanly. The producer-client
wording is good: `Gateway` is explained as a library, not a deployable central
proxy. The step also correctly ties replay to retention, compaction, or tiered
storage instead of promising unlimited recovery.

Possible polish: include one concise example of key choice and partition skew,
because that issue comes back when discussing hot partitions and consumer lag.

### Step 3: Async Messaging - Event-Driven Microservices

The consumer-group explanation is accurate and teachable. It clearly separates
"more members in the same group" from "another group for another workload", and
the poison-record trap is exactly the right operational warning.

The remaining issue is the `Orchestrated / RPC` option visual. It now renders
without hidden nodes, but it does not show a central orchestrator or synchronous
downstream call chain. Fixing that would make the trade-off sharper.

### Step 4: Stream Processing - Real-Time Aggregations

This is one of the strongest steps. It explains that stream processing is not
just a consumer loop, then introduces windows, event time, watermarks,
changelog-backed state, stream-table joins, and state-size limits.

No structural fix needed. A minor wording improvement would be to make the
serving-store sink explicit in the final design as well, so the aggregate-output
path stays visible after this step.

### Step 5: CDC & Event Sourcing

This step now handles the source-of-truth boundary correctly. CDC keeps the DB
commit authoritative; event sourcing makes the event append authoritative; the
transactional outbox is the middle ground for app-shaped events without a
dual-write race.

The three option diagrams now resolve cleanly. Keep the dual-write option as an
anti-pattern; it is a useful teaching contrast.

### Step 6: Delivery Semantics

This step is technically honest. It distinguishes at-most-once, at-least-once,
idempotent producer dedupe, transactions, `read_committed`, offset-commit
ordering, and external side-effect idempotency. The "exactly-once processing
effects, not literally-once delivery" wording is particularly good.

Possible polish: if the data model is expanded again, add a tiny idempotent sink
record example, such as `(idempotency_key, effect_type, processed_at)`.

### Step 7: Scale & Operate

This is now a strong production close. It covers partition sizing, metadata and
rebalance cost, RF=3, ISR, KRaft, retention, compaction, tiered storage, lag,
under-replicated partitions, quotas, ACLs, schema compatibility, PII
classification, audit, guarded offset reset, DR, RPO/RTO, and SLOs.

The key improvement is already present: the step now includes a trap for using
Kafka as the answer to every problem, with clear alternatives. Add the
platform-control pattern tag here so the renderer surfaces that pattern on the
step itself.

## Final Design Review

The final design integrates the chapter well. It includes producers,
collectors, the producer client, broker cluster, KRaft metadata/controller
quorum, schema registry, control plane, consumer services, stream processor,
Kafka Connect/CDC, retry/DLQ topics, source DB, warehouse, serving store, and
monitoring.

The final design is strongest as a component inventory. It can be stronger as
an operations story by exposing the loops that keep the platform safe:
schema-policy enforcement, topic/quota governance, lag-driven scaling, DLQ
replay after a fix, guarded offset resets, and DR cutover/restore. The
components are already present; the view and prose just need to preserve those
relationships.

## Concept Introduction and Learning Flow

The concept staging is excellent:

- Step 1 creates the coupling and replay problem.
- Step 2 introduces the durable append-only log.
- Step 3 explains pub/sub fan-out and consumer groups.
- Step 4 introduces stateful processing over the log.
- Step 5 separates CDC, outbox, event sourcing, and source-of-truth boundaries.
- Step 6 adds delivery semantics and correctness boundaries.
- Step 7 closes with scale, governance, security, DR, and operations.

That is the right order for a candidate. They can build the answer
incrementally instead of naming Kafka first and then reciting features.

## Step-to-Final-Design Coherence

The steps map cleanly into `finalDesign`:

- Step 2 contributes `Producers`, `Ingest`, `Gateway`, `Broker`, `SchemaReg`,
  `Connect`, and `Warehouse`.
- Step 3 contributes `Consumers`, `Serving`, `DLQ`, and consumer-lag metrics.
- Step 4 contributes `Stream` and the aggregate-output path.
- Step 5 contributes `SourceDB`, CDC links, Connect, and rebuildable read
  models.
- Step 6 strengthens `Gateway`, `Broker`, `Stream`, `Consumers`, and `DLQ` with
  idempotence, transactions, offset commits, and poison-message handling.
- Step 7 brings in KRaft metadata, replication, retention, control plane,
  security/governance, DR, monitoring, and SLOs.

The coherence gaps are now small: make the final view show DLQ replay and
operational governance loops, and add the missing Step 7 pattern tag.

## Realism Compared With Production Systems

The dataset is realistic for a system-design interview. It covers many issues
that candidates often miss: retention limits on replay, hot-key and partition
parallelism, schema compatibility, consumer lag, poison messages, CDC source of
truth, transactional outbox, exactly-once scope, RF/ISR durability, tiered
storage, quotas, ACLs, audit, and non-transparent multi-region failover.

Further realism would come from making operational workflows explicit:

- How topic creation and partition changes are requested, reviewed, and rolled
  out.
- How a breaking schema change is blocked, rolled forward, or rolled back.
- How a team safely resets offsets or replays a topic.
- How DLQ replay is authorized, deduped, and traced after a fix.
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
- Each structured view link now has both endpoints present in that view's node
  list; the previous endpoint-mismatch issue is resolved.
- `satisfies[*].steps[*]` references resolve to real step IDs.
- Dataset-level pattern names referenced by steps resolve.
- Sequence participants and messages resolve cleanly, and participants use
  canonical node IDs.
- `satisfies.nonFunctional` now covers the expanded requirements for latency,
  operability, availability/DR, and governance/security.
- `dataModel` now includes the platform-control entities previously missing.
- `Zoo` remains an old internal id for a node labeled
  `Controller / Metadata (KRaft)`. The label is fine for readers; renaming the
  id is optional and only worth doing if internal ids become visible in the UI.
- No generated docs rebuild is needed for this review-only edit.

## Recommended Edits, Prioritized

### P1: Make the Step 3 alternative diagram match the orchestrated/RPC prose

Add an explicit orchestrator/RPC node and call-chain links, or rename the option
to match the current direct-call visual. Prefer adding the node because the
contrast with Kafka pub/sub is pedagogically important.

### P1: Preserve DLQ replay and guarded operations in the final design

Add `dlq-broker` to the final design and Step 7 views, and add a short note for
guarded workflows: topic request, schema policy, offset reset, DLQ replay, and
DR failover/restore.

### P2: Update the script and level variants for platform operations

Mention control-plane governance, ACLs/quotas, schema policy, PII/audit, and
DR/RPO/RTO in the final script phase and Senior expectations.

### P2: Add the missing Step 7 pattern tag

Add `Platform control plane (governance)` to Step `scale-ops.patterns` so the
step displays the pattern it already owns at the dataset level.

### P3: Add optional technology-choice material

For a mature book chapter, consider adding `technologyChoices`: self-managed
Kafka vs managed Kafka, Kafka vs Pulsar, Kafka vs cloud-native streams, Kafka
Streams vs Flink, and Kafka vs task queues. This is enrichment, not a blocker.

## What Not To Change

- Keep the seven-step structure. It teaches the design in the right order.
- Keep the baseline point-to-point step; it makes Kafka's value concrete.
- Keep the CDC vs event-sourcing distinction; it is one of the chapter's
  strongest correctness points.
- Keep the external-side-effect caveat in the delivery semantics step.
- Keep Step 7 as the production close; it is the right place for partitions,
  replication, retention, governance, DR, and the anti-golden-hammer warning.

## Bottom Line

This is now a strong Kafka use-case interview and a good fit for the Golden
Hammers group. The previous concrete blockers have been fixed. The next pass
should focus on sharpening one alternative diagram, making final-design
operations loops visible, and bringing the wrap-up script/level guidance up to
the same production level as the main walkthrough.

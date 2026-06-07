# NFR checklist by task class

> Completeness guarantor for Stage 1. For EACH relevant item: a value / "not applicable" / "clarify with the operator". Don't rely on memory — go through the list. Categories and wording rest on the engineering canon.

## Universal NFR (ALWAYS check, any class)

- **Volume and growth:** how many records/events now and in 1-2 years; data size; growth rate. (right-sizing hinges on numbers)
- **Load:** read/write RPS/QPS; read/write ratio; burstiness (bursty?); fan-out.
- **Latency:** target response time (p50/p99 — not the average, the average lies); what counts as "slow".
- **Data freshness:** is delay/staleness acceptable; is real-time currency needed.
- **Consistency:** is strong (linearizability) needed or is eventual enough; on what exactly.
- **Cost of error:** what if the result is wrong/stale/lost — money, legal risk, or non-critical.
- **Reliability/data loss:** is loss acceptable; are durability guarantees needed.
- **Availability/HA:** how much downtime is acceptable; is fault tolerance needed (PoC/internal ≠ HA).
- **Scale:** by size / geography / number of teams-organizations.
- **Security/PII:** personal/sensitive data; access, encryption, audit requirements.
- **Budget/team/deadlines:** is there DevOps; deadlines.
- **Environment constraints (Deployment):** Cloud (where Managed services are an option) vs On-premise (where we install everything ourselves); environment isolation.
- **Operations:** who maintains it and how; monitoring; diagnosability.

## Class: Integration with an external system / API / ERP

🔴 Often missed (canonical failure — "synchronous calls, simpler", orders get lost):
- **Delivery guarantee:** what if the receiver is unavailable — do we lose the request or not; is losing requests acceptable, and at what volume; is a queue/buffer needed?
- **Idempotency:** retries must not double the effect (payment, charge, order).
- **Cross-system consistency:** what happens on partial failure (it was written on our side, not on theirs).
- **Synchronous vs asynchronous:** a timeout does not tell you whether the operation completed on the other side.
- **Partial failures:** the external system is down/slow — do we keep working? At full capacity or graceful degradation?
- **Contract evolution:** their API changes — compatibility, versions.
- **Backpressure:** what happens on a spike; the partner's rate limits.
Key concepts: idempotency, message queues, partial failures, end-to-end argument, dead-letter queue (DLQ).

## Class: Analytics / BI / reports

🔴 Often missed:
- **Volume is decisive:** 1000 rows → do NOT split OLAP/OLTP; ~1M+ or heavy scans → a separate analytical store.
- **OLAP vs OLTP:** query type (point lookups vs aggregates over many rows).
- **Analytics freshness:** is real-time needed or a batch every N.
- **Tool currency (anti-overkill):** don't build a lambda/stream stack by hand if the DBMS covers it out of the box (incremental/refreshable MV, native ingestion from a queue). AI-BI — check whether a product covers it out of the box.
Key concepts: OLTP vs OLAP, columnar storage, materialized views, stream/batch processing.

## Class: Derived data (search/cache/read-model from the main DB)

🔴 Often missed:
- **Source-vs-derived drift:** how to keep the index/cache consistent with the DB.
- **Initial snapshot + change stream:** CDC/log as the mechanism.
- **Cache staleness:** TTL, invalidation, what happens when the user sees stale data.
Key concepts: CDC (Change Data Capture), decoupling cache from the DB, cache consistency, eventual consistency.

## Class: Storage / DB choice

🔴 Often missed:
- **Access pattern:** point keys vs ranges vs full-text vs graph.
- **Read/write profile:** what we optimize (LSM — writes, B-tree — reads).
- **Data model:** relational / document / graph / timeseries / geo-distributed / columnar — from the relationships or / and use-cases.
Key concepts: data models (relational/NoSQL), index structures (LSM-tree, B-tree), replication, sharding, partitioning.

## Class: AI systems / RAG / LLM agents

🔴 Often missed (canonical failure — "an agent calls a side-effecting tool in a loop with retries without idempotency → double charge/duplicated order"):
- **Cost (Token Economics):** projected token volume per day/month. Budget protection (quotas, rate limits).
- **Latency (UX):** Time-To-First-Token (TTFT). Is waiting for the whole answer acceptable, or is streaming mandatory.
- **Privacy and Security:** can data be sent to vendors (OpenAI/Anthropic) or does PII/NDA require self-hosted models.
- **Cost of hallucination:** what happens if the bot lies? (medicine = Human-in-the-loop; casual chat = non-critical).
- **🛡 Guardrails:** idempotency of agent tool-calls (a retry doesn't double the effect — see the "Integration" class); untrusted input as an attack vector — prompt-injection via a RAG document/user text (isolate instructions from data); human-in-the-loop on the critical path.
- **Context window:** how many documents do we feed in? Risk of losing focus (lost in the middle).
- **RAG index freshness:** how quickly new documents from the DB must become available for vector search.
- **Observability (Evals):** how do we know the system works well? (LLM-as-a-judge, feedback collection).
Key concepts: Token economics, TTFT, RAG, Hybrid Search, tool-call idempotency, prompt-injection/guardrails, Hallucination mitigation, Evals, Human-in-the-loop.

## Class: Scaling / high load

🔴 Often missed:
- **Describe the load first**, then decide (not "shard immediately").
- **Hot spots:** skew toward popular keys (a hot route, a celebrity).
- **Partitioning — when:** a single node can't handle the volume/load, not earlier.
- **Replication:** read availability, geo-proximity — is it needed.
Key concepts: latency percentiles (p99), partitioning (sharding), hot spots, replication.

## Class: Distribution / coordination / consistency

🔴 Often missed:
- **Multi-region and "will I see my own write":** linearizability vs eventual consistency from the nearest replica.
- **Leader election / split-brain / fencing** in coordination.
- **Unreliable network and clocks:** timeouts, process pauses.
Key concepts: linearizability, CAP theorem, PACELC theorem, consensus, split-brain, replication lag.

## Class: Security / Compliance / Isolation

🔴 Often missed:
- **Tenant isolation (Multi-tenancy):** physical vs logical data separation (row-level security, separate schemas/DBs).
- **Audit and logging:** is it required to prove who changed a specific field and when (Audit Trail).
- **Encryption:** are custom encryption keys (KMS) needed, encryption at rest and in transit.

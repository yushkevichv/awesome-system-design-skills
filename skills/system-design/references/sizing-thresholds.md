# Right-sizing: "parameter → trigger → decision" thresholds

> Stage 2. The default is the simplest sufficient. Add complexity only on reaching a threshold, justified by Stage 1 numbers. Thresholds are common-sense + canon guides, NOT hard laws; check against the concrete case and the current capabilities of tools (what's already out of the box). Refine the numbers with practice.

## Principle

Every added complexity (splitting stores, HA, cluster, sharding, a separate stream layer, a distributed transaction) is a **cost**: development, operations, failure complexity. Pay only when a requirement dictates it. "Not sure it's needed" = by default we DON'T do it, and we record the threshold at which it becomes needed.

## Analytics: split OLAP/OLTP?

| Volume / situation | Decision |
|---|---|
| up to tens of millions of rows, with proper indexes | DON'T split — a modern DBMS handles it |
| hundreds of millions of rows or heavy scans interfere with OLTP | a separate analytical store (columnar) |
| real-time dashboards on large data | check: incremental/refreshable MV out of the DBMS box BEFORE building a stream stack |

## Caching (Redis / Memcached / Valkey)
| Situation | Decision |
|---|---|
| Point reads from the DB are fast, the DB isn't resource-bound | Cache NOT needed — it's an extra point of failure and invalidation trouble |
| latency <10ms required OR the DB is CPU-bound on heavy read queries | Add a cache, MANDATORILY fixing the invalidation policy (TTL, write-through) |

## AI / RAG / Choosing models and search

> ⚠️ This domain changes fast. Concrete **models/products/prices/names — NOT from memory, but from current research** at design time (principle 4). The examples hardcoded below illustrate the selection rule, they are not a 2026+ fact. The choice of store/model — from right-sizing (volume/latency/filters/privacy), not from fashion.

### Model choice (Intelligence vs Cost/Speed)
| Situation | Decision |
|---|---|
| Routing, JSON extraction, simple summarization | Fast/cheap tier — pick the current small model (research at design time) |
| Complex reasoning, writing code, decision-making | Reasoning-heavy tier — the current frontier model (research at design time) |
| Strict NDA, PII, no internet | Self-hosted open-source model on your own infra (pick a current one; research) |

### Search for RAG (Retrieval)
| Trigger | Decision |
|---|---|
| Search by meaning only (knowledge base, articles) | Vector Search (a vector DB or pgvector) |
| Search by names, SKUs, exact terms + meaning | Hybrid Search (Vector + BM25/FTS) + a mandatory Reranker |
| Few documents (< 10-50k) and they're static | Plain in-memory or Postgres vector search. Don't over-engineer with Pinecone/Milvus clusters |

### Orchestration and UX
| Situation | Decision |
|---|---|
| Steps known in advance (find -> summarize) | A rigid pipeline (Chains), predictable and cheap |
| The step chain depends on answers (tools needed) | Agentic Loop (ReAct), BUT with a hard step limit (max_steps) |
| The model's answer takes more than 1-2 seconds | Mandatory Streaming API (SSE) for the frontend |
| High hallucination risk in a critical function | Apply the Human-in-the-loop pattern (a human approves) |

## High availability (HA)

| Situation | Decision |
|---|---|
| internal tool, PoC, downtime acceptable | HA NOT needed |
| money/external users, downtime = losses | HA needed, compute where exactly (not everywhere) |
| "just in case" without a requirement | DON'T do it — that's overkill |

## Partitioning / sharding

| Trigger | Decision |
|---|---|
| data and load fit on one node (+read replicas) | DON'T shard |
| one node can't handle the volume OR write load | shard; pick the key from the access pattern |
| there are hot keys (a popular route) | separately: split the hot key, not general sharding |

## Consistency

| Situation | Decision |
|---|---|
| one replica/region, we read where we write | the question doesn't arise |
| multi-region, "must immediately see my own write" | linearizability OR client-centric guarantees (read-your-writes) — deliberately |
| staleness is tolerable | eventual consistency (cheaper, more available) |

## Distributed transaction / 2PC

| Situation | Decision |
|---|---|
| retries of external operations, exactly-once needed | idempotency (cheaper than 2PC) — prefer it |
| need to update your DB and send an event to a queue | Transactional Outbox (write the event to the same DB in the same transaction, a separate worker sends to the broker) — prefer over 2PC |
| strict atomicity across independent resources | 2PC/distributed transaction, understanding the high cost to availability and performance |

## Queue / async for integrations

| Situation | Decision |
|---|---|
| can't lose a request while the receiver is down | a queue/broker, not a synchronous call |
| fast response, loss acceptable, simplicity matters more | synchronous — but RECORD the risk explicitly |

## Stream processing / a separate pipeline

| Situation | Decision |
|---|---|
| need to feed a derived store (index/read-model) from the DB | CDC + log; check whether the DBMS covers it out of the box |
| a complex lambda by hand | FIRST check incremental/refreshable MV, a native Kafka engine — don't build the superfluous |

## Topology: monolith vs services?

> The astronaut trap: right-sizing the DATA (no sharding/HA) but drawing 10 micro-services for a small-team pilot. Service decomposition is a complexity axis too — size it.

| Situation | Decision |
|---|---|
| small team, pilot, few deployable concerns | **modular monolith** (+ 1-2 workers for async/cron) — do NOT split into many services; the responsibilities are modules inside it |
| a specific part genuinely needs independent scaling/deploy, or a separate team owns it | extract THAT part as a service — by the seam that needs independence, not by default |
| "microservices because it's modern" / many services for a small team | DON'T — 10 deploys + inter-service calls crush a small team; contradicts a "small team / light ops" NFR. The astronaut anti-pattern |

## Document altitude (right-size the artifact itself)

| Task size | Artifact |
|---|---|
| small / pilot | ~2-4 pages; collapse to §0/1/3/6/7; rigid SLOTS only for mechanisms that actually exist |
| large / prod | full structure (`output-format.md`) — but still say each thing ONCE (home section + references) |

## The golden rule of application

If there's no concrete number/requirement from Stage 1 for the added complexity — it's unjustified. Record: "we DON'T do X — threshold Y not reached (current value Z)". That is deliberate right-sizing.

# Output templates (the artifact)

> Reference for the artifact. SKILL.md holds the process, gates and verdict; here is "how to format the result". Follow these forms.

## The rigid-slot principle (this is what actually buys convergence)

Convergence is NOT bought by telling the agent "be concrete" — that gets gamed (name two technologies, feel done, leave the rest as boxes). It is bought by **required fields that vague prose cannot fill.** "Services talk asynchronously" must FAIL the template, because an async contract REQUIRES `{Broker, Topic/Queue, Idempotency-key, Ordering?, DLQ?}`. An empty or hand-wavy required field is a **visible hole**, not an acceptable answer. Every contract and decision below carries required slots — fill them with concretes or it is not done.

**Right-sizing still applies:** fill required slots only for mechanisms that actually exist. A pizzeria-for-40-houses has no broker — its async slot reads "synchronous call; loss-risk recorded", which is a *filled* slot, not a hole. Rigid SLOTS where a mechanism is present; the SET of mechanisms is sized to the task.

## Design Document (the artifact) — structure

A single **self-contained** markdown file (a downstream spec skill reads only this file, no chat history). Section order is the convergence contract — keep it stable.

**0. Title + the bet (1 line).** What we build + the single load-bearing architectural bet.

**1. System Context & Constraints** — C4 Level 1, the frame everything hangs on:
- **Business goal** — right-sized: a sentence for a small task, a paragraph or two for a 30-page spec. Condense without losing meaning; do NOT dogmatically compress to one line.
- **Key NFR drivers** — the few numbers that actually drive the design (e.g. RPS > 10K, data > 500GB, HA required).
- **Out of scope / anti-goals** — what we deliberately do NOT do this iteration. Structural defense against scope-creep AND over-engineering (the astronaut-check lives here).
- **Trust boundaries** — which external systems we integrate with and whom we do NOT trust → where validation / rate-limits / guardrails must sit.

**2. Requirements** — self-contained:
- **FR** (one line each), **NFR table** (value / not-applicable / clarify, per item; every quantitative value shows its derivation + label `calculated`/`assumed`/`verified`), **constraints**, **source labels** (`explicit`/`inferred`/`confirmed`; `assumed` is a pre-gate proposal state only, never a final label).
- **Illuminated dark spots** — the resolved "what we might have missed" + assumptions, KEPT (not discarded): each = the gap surfaced → the proposed default → the operator's answer. Once the operator accepts a proposed assumption it becomes **confirmed data** (`confirmed`); a corrected one becomes `explicit`. The artifact preserves the JOURNEY for knowledge transfer (so a weaker engineer/downstream sees *why the task is not what it naively looked like*) but carries **no unconfirmed `assumed`** status.
- **"was → became"** — if the initial statement materially transformed during the gate, record the shift.
- **No "open" status.** Everything here is resolved and **confirmed** (`explicit` / `inferred` / `confirmed`). No `assumed`, `needs-clarification`, or `missing-critical` survives: if any remains, the gate did NOT pass and there is no design to ship — you are still at Stage 1. An architecture-critical item that cannot be closed even by an operator-accepted assumption HALTS the process (`needs clarification`), it does not ship as an open hole.

**3. Architecture — diagram + Component Map** — C4 Level 2 (Containers); diagram in Mermaid:
- A component/container diagram as a **Mermaid `flowchart` with one `subgraph` per layer** (presentation / API / domain·workers / data / external) — this IS a C4 Container view *by layer*. No abstract "backend".
- **Diagram format = Mermaid (the editable source).** It renders everywhere, stays diffable in the artifact, and imports straight into Excalidraw (Excalidraw has native "Mermaid → Excalidraw" import) when an editable canvas is wanted. So do NOT hand-author `.excalidraw` JSON — it is fragile and lays out poorly; the Mermaid is the source, Excalidraw is a one-click import if the operator wants to redraw.
- Per component, REQUIRED slots: `{Name (concrete, e.g. OrderService), Type (Stateless API / Stateful Worker / Cron Job / Async Consumer), Responsibility, Owns-data, Talks-to}`.

**4. Critical End-to-End Flows** — ALL load-bearing flows (those that branch the architecture / cross components / carry failure complexity), not an arbitrary "top-N". Each flow has TWO sub-sections:
- **Happy path** — step by step, concrete: e.g. "Client POSTs /checkout → OrderService writes PENDING to PostgreSQL → publishes `order.created` to Kafka → PaymentWorker → Stripe".
- **Failure mode** — what if a dependency is down: retries/backoff → DLQ → reconciliation. This is the "think like a senior for a junior" forcing function — queues and retries are born here.
- Any async step MUST fill `{Broker, Topic/Queue, Idempotency-key}` or it is a hole.
- Trivial/secondary flows: list them and OFFER a drill-down — never silently drop them.

**5. Cross-cutting policies** — each stated ONCE, referenced from the flows (kills the "mush" of restating the same rule five times): idempotency, tenant isolation, the security gate, DLQ/failure, cache scoping. Anchor security policies to the §1 trust boundaries.

**6. Decision Records (ADRs)** — the engineering bets. Resolved FIRST via the Stage 3 decision gate (the operator chooses), PRINTED here as the justification log.
- **Stack-index table at the top** — one row per load-bearing layer so NONE is skipped: `Layer → Chosen → ADR#`. Required layers: language/runtime, API/web, **topology (monolith / modular-monolith / services)**, OLTP store, OLAP store, async/queue/workers, AI-orchestration/agent runtime, model gateway, guardrails, cache, observability/eval. Skip a row only with an explicit "n/a — reason".
- **Each ADR**, required slots: `{Decision (concrete tech/pattern), Why → NFR#, Rejected alternative + why rejected, Applicability boundary (the concrete trigger — load/volume/RPS/team-size — at which it must be refactored), ⚠ override (set ONLY if the operator chose against the canon/research — see Stage 3)}`.
- **Dual trace** per ADR (see below).

**7. Right-sizing & evolution triggers** — what we deliberately did NOT build + the threshold at which each becomes needed (topology, sharding, HA, stream). The astronaut-check: is the topology proportionate to scale AND team size?

**8. Validation & readiness verdict** — the independent challenge result + the readiness verdict + the v1 build order (see sections below). Include a one-line **grounding status**: either "dual-track converged" (modern + grounded tracks reconciled — note any layer where they diverged and how it resolved) OR "single-track (no grounding skill available) → expertise + currency only". Never leave grounding status silent.

**9. Handoff** — what this artifact settles vs what the downstream spec/SDD skill takes (API contracts, DDL, task breakdown) within the chosen stack.

### Artifact contract (for handoff)
- **Path:** `docs/design/<slug>.md` (create `docs/design/` if missing). `<slug>` = short kebab-case feature name; if the operator didn't name the feature, derive and confirm the slug.
- **Self-contained:** sections 0-9 are everything a spec tool needs — no chat history.
- **Length right-sized:** a pilot is ~2-4 pages with sections collapsed (e.g. §0/1/3/6/7), not a 16-block enterprise doc. See `sizing-thresholds.md` (document altitude).
- **Written only on the operator's "save" confirmation** (Stage 5), not automatically.

## ADR template (detail for §6)

```
ADR-N: <decision title>
Decision:   <concrete technology/pattern — not a category>
Why:        <how it satisfies our requirement — reference NFR/FR #>
Rejected:   <the main alternative> — <why rejected>
Boundary:   <concrete trigger at which this must be refactored: e.g. ">100M rows", "write RPS >2000", "team >2 squads">
⚠ Override: <set ONLY if the operator chose this against the canon/wiki/research; restate the risk>
```

Never name a category ("columnar DB", "message broker") when a buildable design is needed: a concrete technology, or the explicit selection rule that determines it.

**Versions are verified, not recalled.** If a `Decision` names a version, it is the current **latest-stable**, verified against a live source (docs / package registry / official release page — not memory); pin an older version only with an explicit reason (LTS, a named compatibility constraint, a known regression). Unverified → write "latest stable (verify at impl)", never a remembered number. A version emitted from memory is a defect, the same way an unshown number is.

## Dual trace (per ADR)

For each load-bearing decision — what backs it:
- **source trace (always):** FR/NFR number always; current-doc URL or book+page only when actually read/verified. Never invent citations.
- **missing proof:** if no external source was actually checked, say `assumption` or `needs verification` instead of fabricating a book/page/URL.
- **`[[wiki-slug]]` pointer (when the wiki is available):** additionally, so the note can be opened. No wiki access → source only. A slug by itself is not proof; the proof is the source trace.

Evidence line format (when both wiki and source exist):
```
- [[note-name]] (`wiki/note-name.md`) — why it's relevant to the decision; source: <author, book, ch./p.> or <doc URL>.
```

## Independent Final Challenge (a separate skeptical pass) — feeds §8

Before finalizing, review your design as an independent reviewer — don't redesign from scratch, test it against the task and the failure modes. Prove, don't declare:

1. **Task fit:** does the design solve the stated task? Prove it — map the operator's goals onto components.
2. **FR/NFR coverage:** a "requirement → design element → residual risk" table; partial coverage and gaps — honestly.
3. **Optimality / over-engineering:** is this the simplest design that satisfies the requirements? Name superfluous components (including too many services for the team), a simpler alternative, and the threshold at which the complexity is justified.
4. **Architecture-change boundaries:** concrete triggers (load, volume, latency, tenant count, team size, compliance) at which the design must be revisited.
5. **CAP/PACELC and HA:** for distributed/replicated state — the consistency/availability/latency tradeoff; if CAP/PACELC doesn't apply — explain why, but still cover HA/backups/recovery/degradation.

**Challenge verdict:** `Pass` (solves it, residual risks named) / `Pass with changes` (apply the changes before presenting) / `Fail` (doesn't solve it or is unjustified). Don't hide a failed challenge — rework it and say what changed.

## Readiness verdict (output gate) — feeds §8

The honest output state of "is it ready for work":
- **`ready for SDD`** — ZERO open / `needs-clarification` / unconfirmed items, every load-bearing layer chosen, failures and tradeoffs explicit, implementation tasks derivable without lead re-validation. (SDD = the downstream spec / spec-driven-development stage.)
- **`ready with assumptions`** — ready; rests on operator-**confirmed** assumptions recorded as carried-forward bets to revisit (these are confirmed, not open — they live in §7 evolution-triggers and travel into the spec).
- **`needs clarification`** — the gate did NOT pass: unconfirmed / `needs-clarification` items remain. **This is NOT a design verdict** — you are still at Stage 1; the design does not ship. Loop back and resolve per item.
- **`needs lead review`** — critical NFR/security/data-loss/compliance/cost/operational risks remain, or the design knowingly accepts a risky tradeoff.

## Drill-downs & handoff (end of Stage 5) — §9

Two surfaces, kept separate (see SKILL.md Stage 5): the **chat response** carries a concise summary + verdict + the next-step offers; the **artifact file** (`docs/design/<slug>.md`) carries the full self-contained document.

At the end of the chat response, offer **concrete** next steps tied to this design (not an abstract "what else should I cover?"). They split into two kinds:

**(a) Architectural deep-dive — stays here, appended to the artifact.** A targeted drill-down of one subsystem when a large task warrants it: failure modes/invariants of a specific flow, a consistency model, a degradation plan, a secondary flow not yet written out. Skip for small tasks.

**(b) Spec/implementation — handed off, NOT done here.** You MUST protect this skill's context: do not write low-level SDLC specifications (API contracts, DDLs, component code, task breakdown) inside system-design. Instead advise invoking a specialized spec/SDD skill, passing the **saved artifact path** as input.

Example handoffs (pass the saved file, e.g. `docs/design/orders-erp-integration.md`):
- "To generate exact OpenAPI contracts for `OrderService`, invoke your API-spec skill (spec-kit / openspec / superpowers / native) with the saved artifact as context."
- "To design the database schemas, invoke your DB-schema skill with the same file."
- "To go deeper into the failure modes of the Payment flow, we can drill down right here and append it to the artifact."

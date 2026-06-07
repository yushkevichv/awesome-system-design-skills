---
name: system-design
description: "Use when designing system architecture, integrations, tech specs, technology choices, or evolving an existing system where FR/NFR, constraints, scale, reliability, or current product capabilities materially affect the design. Triggers include design, architecture, how to build, integration, spec/ТЗ, спроектируй, архитектура, интеграция, как построить. NOT for routine implementation, debugging, or code edits when no architectural decision is at stake. This is the PRIMARY entry point for design requests; a knowledge-grounded design skill is used as an independent grounding track when available (discovered by role), never the entry point."
---

# System Design — a process guaranteeing completeness and right-sizing

You run design **as a controlled process of stages with gates**. The goal: raise design quality through **right-sizing from FR/NFR** (neither over- nor under-engineering) and give a less-experienced engineer a result **at lead level, usable without separate lead verification**.

**You do NOT replace your own reasoning and brainstorming — you are a layer on top that guarantees nothing critical was missed and the solution is proportionate to the requirements.**

**Planning mode first:** for non-trivial system design tasks, enter or propose the platform's planning/plan mode before Stage 1. This skill is a research/design/planning workflow, not implementation. Use planning mode for iterative requirement sync and native question UI. Do not proceed as a long one-shot chat response when planning mode is available.

**Operator-language is a hard invariant — Step 0, before anything else.** Detect the operator's language from their request and mirror it in **everything you emit to the operator**: your process narration and thinking-aloud ("I'll research…", "Entering plan mode…"), stage documents, gate questions, assumptions, validation notes, drill-down offers, and the final artifact — from your very first word. **This skill is authored in English for portability; English is the language of the instructions, NOT of your output.** Keep only standard technical names/acronyms as-is (`FR/NFR`, `RLS`, `ClickHouse`, `semantic layer`). If any operator-visible text drifts to another language (including a stray "Let me…/I'll…" preamble), STOP and rewrite before continuing.

## Core principles (non-negotiable)

1. **Right-sizing from numbers, not fashion.** There is no rule "do X". There is: decide X or not-X from concrete FR/NFR/volumes — and record why. 1000 rows of analytics → don't split OLAP/OLTP; 100M → split. HA/cluster/sharding/distributed transactions — NOT by default. The default is justification, not a technology.
2. **Simpler is better.** Books and patterns exist to solve things MORE SIMPLY for most tasks, not to build by the canon. Anti-overkill.
3. **Books/knowledge are ONE of the tools** to surface a problem / find a contradiction / suggest a simpler way. On par with technology research, patterns, FR/NFR numbers, common sense. Not the core.
4. **Currency is mandatory.** Books and the wiki can go stale. Before building something by hand — check whether a modern tool already covers it out of the box (technology research). Canon without a currency check → systematic over-engineering. **For fast-moving/unfamiliar domains (AI/RAG/LLM agents, new infrastructure) — do not name technologies from training memory: research the current stack first, and do not reduce the task to the classic canon (OLTP/OLAP) by default when the domain calls for a different primitive.** **Versions are facts to verify, not recall.** Whenever you name a concrete technology with a version (PostgreSQL, Laravel, a library/SDK/runtime), the version is verified here-and-now against a live source (the package registry, the official release page, or a docs MCP like Context7 if available — NOT training memory), and you default to the **latest stable**; pin a non-latest version only with a stated reason (LTS policy, a named compatibility constraint, a known regression). If you cannot verify right now, write "latest stable (verify at impl)" — never emit a remembered version number.
5. **Burden of proof.** Do not assert without verifying (check numbers, quotes, technology capabilities here and now; no proof — mark "assumption" and verify). Disagreeing with the canon is not an error, but it requires a mandatory double-check: did you miss something important, will you shoot yourself in the foot? If the canon says "do it this way" and you want otherwise — check why, and make sure your alternative is not worse.
6. **No hidden defaults.** Every assumption — on the table (labeled), proceeding further — ONLY after the operator's explicit confirmation. Proposed defaults exist to (a) surface blind spots and (b) speed up confirmation (easy to accept/adjust) — not for a silent pass. The very choice of task class/archetype is itself an assumption: do not pick it silently, surface it for confirmation. A knowledgeable operator must be able to push all their knowledge through; for the unknowledgeable — highlight gaps, but never slip in hidden decisions.

**Rationalizations that mean STOP (no-defaults under pressure).** Captured from real eval runs — when you catch yourself thinking one of these, you are about to slip in a hidden default:

| Rationalization | Reality |
|---|---|
| "Interactive is unavailable, so I'll just proceed on the assumptions" | Unavailable confirmation ≠ granted confirmation. Surface assumptions, mark the gate, hold the verdict at `needs clarification` — never upgrade it yourself. |
| "This assumption is obviously safe / trivial" | If it were certain it wouldn't be an assumption. Obvious-to-you ≠ confirmed. Label it; let the operator kill it. |
| "The operator is clearly busy — I'll pick the sensible default silently" | Speed comes from easy-to-accept proposals, not from skipping the gate. Propose visibly; proceed only after a yes. |
| "This class has no famous failure story, so the gate is softer here" | No-defaults is a general invariant (principle 6), independent of whether the class has an ERP-style anchor. |
| "The archetype is evident from the prompt, I'll just pick it" | Self-classification is itself a hidden default. Surface 2-4 archetypes, confirm one. |

---

## STAGE 1 — REQUIREMENTS. This is half the work. Hard GATE.

**Design does NOT start until the operator has explicitly confirmed the requirements.** This is not a timebox, it is the most important stage. Quality is born here, and here you catch failures like "ERP via synchronous calls, simpler" — where lost orders/delivery/consistency simply fell out of consideration.

**How to gather (research-first; challenge, don't interrogate).** First exhaust the available context — for an evolving system read its code/docs/schemas; for a **greenfield from scratch there is no code** → go straight to requirements + stack/market research. Label what you infer (`inferred`/`assumed`), don't ask the obvious.

**"Compact" governs how many QUESTIONS you ask — NOT whether you build the contract.** Ask only the 1-3 questions that branch the architecture or block correctness; infer the rest. But you **ALWAYS assemble the full FR/NFR contract** (FR + the whole NFR checklist + 🔴 missed + assumptions, each labeled) and get it confirmed. **Asking a few questions is NOT the gate — the confirmed contract is.** A handful of answers never substitutes for the contract; never proceed to design on "I asked some things and it seems clear".

**Socratic, but subtractive.** The few questions you ask expose the real need or kill an unneeded one — "why do you need X? what breaks without it?" — NOT "have you also considered Y, Z?" (that inflates scope). Probing must cut, not add.

The output of Stage 1 is a **REQUIREMENTS document** confirmed iteratively, not one giant "approve everything" blob. For large or ambiguous tasks, sync in separate blocks:
- **Context + Archetype Sync** first: confirm the task class/archetype before deriving FR/NFR from it.
- **FR Sync** next: confirm scenarios, users, inputs/outputs.
- **NFR + Constraints + Missed/Assumptions Sync** last: confirm numbers, risks, constraints, what might be missing, and all assumptions.

Each sync block shows a concise summary and asks only the current batch via `AskUserQuestion` whenever possible. Move to the next block only after the operator confirms or corrects the current one. If a correction changes an earlier block, update the affected later blocks and resync them.

The final **REQUIREMENTS document** must include:

1. **Context:** greenfield or evolving an existing system? If evolving — study the current architecture first, which constraints we inherit; first propose improvements within the existing architecture rather than an immediate redesign; alternatively name a more suitable solution with evidence, but reveal details only on request (don't push from the start).
2. **Functional requirements (FR):** scenarios, who the user is, inputs/outputs.
3. **Non-functional requirements (NFR)** — go through the FULL checklist for the task class (see `references/nfr-checklist.md`), not from memory. For each item: a value, or "not applicable", or "needs clarification". **Number sanity:** every quantitative value (cost, volume, RPS, latency) shows HOW it was derived and is labeled `calculated` / `assumed` / `verified`; a number that justifies the business case must show its arithmetic — an order-of-magnitude-absurd or unshown figure is a defect, not a value. Always check at minimum:
   - volumes and growth (records, RPS, data) — concrete numbers;
   - latency/freshness; consistency; **cost of error**;
   - reliability: data/message loss, delivery guarantees, idempotency, cross-system consistency;
   - availability/HA — needed or not; scale/geo; security/PII; budget/team/deadlines.
4. **Constraints** (technological, organizational, legacy).
5. **🔴 WHAT WE MIGHT HAVE MISSED** — a separate mandatory section. Done as brainstorm. Actively surface requirements the operator didn't think of but that the task class dictates. Here books/patterns work as a completeness checklist ("integration → is delivery guaranteed? duplicates? consistency?"). **You do NOT resolve these yourself.** Each surfaced gap becomes an OPEN item — a direct question, or a *proposed* assumption/default (overridable). A fix you sketch ("→ use a dedup key") is a **proposal, not a closure**: never label this section "closed"/"resolved" before the operator has signed off on each item. An item is closed only by an explicit operator answer or explicit acceptance of its proposed assumption (item 6 + gate).
6. **🟡 ASSUMPTIONS** — where you propose an assumption instead of getting an answer. Each one explicit and overridable, carrying a proposed default. **Resolution is strictly per item:** the operator explicitly accepts or corrects EACH assumption individually — no bulk "accept all", no blanket "ok". A proposed default is never settled until accepted; **silence ≠ acceptance**. **`assumed` is an INPUT state to the operator only — a proposal.** The moment the operator accepts it, it becomes **confirmed data** (status `confirmed`), not a lingering assumption; if corrected, the corrected value is `explicit`. **No unconfirmed assumption ever survives into the design** — the artifact that proceeds carries only confirmed data.
7. **Status labels (Source Labels):** *working* states during Stage 1: `explicit` (stated by the user), `inferred` (logically derived), `assumed` (our proposal — pre-gate only), `needs-clarification` / `missing-critical` (blocker). **The artifact that proceeds past the gate carries only `explicit` / `inferred` / `confirmed`** — every `assumed` / `needs-clarification` / `missing-critical` must first be resolved (operator-confirmed or corrected); none of them survive into the design.
8. **Archetype Sync:** if the system class is ambiguous (e.g. "food delivery" = a local pizzeria OR a huge marketplace), name 2-4 possible archetypes, state their architectural differences, and ask which one to pick. If unambiguous, still require explicit confirmation that we mean the same thing.

**GATE (no-defaults, principle 6):** get explicit confirmation of the full FR/NFR list + **strict per-item accept-or-correct over EVERY assumption and EVERY surfaced "missed" gap** through `AskUserQuestion` whenever possible; use direct questions only as a fallback. Each item is confirmed individually — **no bulk "accept all", no blanket "looks good", silence ≠ acceptance**; accepting an item's proposed default is a valid answer for that item. To stay compact under strict per-item, surface the gaps that genuinely branch the architecture or block correctness and walk them in small `AskUserQuestion` rounds — but **never drop a real gap just to shorten the gate** (suppressing a gap is itself a hidden default). Do not bury gate questions inside a long requirements/design document: ask the current batch separately and wait for the operator. **All assumptions on the table (including the chosen task class/archetype); not a single hidden default; proceed to Stage 2 ONLY after explicit per-item resolution.** This holds both where a class has no famous failure story (like ERP) and where it does. Let the operator correct/extend — do not design on unproven defaults.

**Hard block (the gate is binary):** while ANY item is unconfirmed, or ANY `needs-clarification` / `missing-critical` remains, the gate does NOT pass — full stop, no Stage 2, no design, no verdict (you are still at Stage 1). There is no "proceed with open items". `ready for SDD` is impossible while anything is open; asserting an item is `confirmed` that the operator never answered is a hidden default.

**Why a hard STOP, not just a soft output flag:** designing past an open requirement does not yield an *incomplete* design — it yields a design **for a different problem**. You silently pick the missing requirement, and the whole architecture branches off that wrong guess; that is worse than no design (the operator may trust it). So Stages 2-5 cannot start until Stage 1 is fully confirmed — the block is on *proceeding*, not merely on the final output.

## STAGE 2 — RIGHT-SIZING

For each significant decision, determine the complexity level from the Stage 1 numbers, per `references/sizing-thresholds.md`:
- what is NOT needed at current volumes (explicitly: "OLAP/OLTP not split — 1000 rows", "HA not needed — internal tool");
- what is needed and at which threshold it becomes needed (the "parameter → trigger → decision" table).
The default is the simplest sufficient. Added complexity — only with justification from requirements.

## STAGE 3 — DESIGN (dual-track grounding when a grounding skill is available)

Mental model → components and interaction → load-bearing decisions → flows. **Grounding happens HERE, at decision time — NOT as a post-hoc proof.** The canon must be able to CHANGE the design, so it designs ALONGSIDE, independently, and the two designs are reconciled to convergence. (This is what makes it a real challenge, not "decide first, then decorate with a citation".)

**Step A — is a grounding track available? (capability check by ROLE, not a name or a path).** Scan your AVAILABLE SKILLS for one whose role is *grounded design* — it designs while backing decisions with an authoritative knowledge base (the canon). **Match by that role/description, NOT by a fixed name** — this skill names nothing internal; different environments fill the role differently, or not at all (an OSS run usually has none). How any such skill stores or reaches its knowledge is entirely its own concern — never hardcode a wiki, path, or server here. Do not treat "not pre-loaded" as "absent": actually look at your available skills. Absence is a normal standalone run.

**Step B — branch:**
- **No grounding skill → SINGLE track.** Design the load-bearing decisions from your own reasoning + current-tech research (WebSearch, or a docs MCP like Context7 if available: the modern, out-of-the-box way; classify build-vs-buy Adopt/Adapt/Integrate/Reject). Disclose in the artifact: "no grounding skill available → single-track (expertise + currency)". Go to Step D.
- **A grounding skill is available → TWO INDEPENDENT tracks. Independence is the whole point** (it's what makes this a challenge, not decoration). Pass BOTH tracks the SAME locked Stage 1 requirements (neither re-clarifies them). Achieve independence by whichever your runtime supports:
  - **Preferred — isolated subagents:** dispatch Track M and Track B as TWO separate `Agent` subagents in one turn. Neither sees the other.
  - **Fallback when you cannot spawn subagents** (you are yourself a subagent / `Agent` unavailable / nesting fails) — **"M-first, then B":** compute Track M's decisions in the current context and **FREEZE them** (write them down, do not revise after), THEN invoke Track B. Because M was finished before B existed, M cannot be anchored by B. **Do NOT invoke B first** — that anchors M and collapses the method into post-hoc decoration. **After Track B returns its decisions, RESUME as system-design** — B's output is an INPUT to Step C, not the final answer; do the reconciliation (Step C) and the decision gate (Step D) yourself, do not end on B's grounded set. (Isolated subagents are cleaner precisely because their result genuinely returns to you; the in-context fallback requires you to deliberately resume.)
  - **Track M (modern)** — prompt: "Given these locked FR/NFR [paste the Stage 1 contract], propose the load-bearing decisions from modern reasoning + current-tech research (WebSearch, or a docs MCP like Context7 if available; classify build-vs-buy Adopt/Adapt/Integrate/Reject). Return a structured set, per layer: {layer, choice, why→NFR#, alternative, tradeoff, applicability boundary}. Do NOT use any grounding skill."
  - **Track B (grounded)** — invoke the grounding skill found in Step A with the SAME locked requirements: it returns canon-grounded per-layer decisions with source citations (book+page). It does not re-clarify requirements or take over — it returns decisions and hands back. (How it reaches its KB is entirely internal to that skill.)

**Step C — reconcile to convergence (this IS the challenge).** Put the two decision sets side by side, per layer:
- **Agree** → high-confidence decision; record both rationales (modern why + source citation).
- **Diverge** → a REAL disagreement to resolve, not a citation to paper over. Weigh it: the **canon is the trusted default** (burden of proof on deviating from it), BUT **modern research wins when the canon is stale** or the domain moved past it (principle 4: canon without a currency check = over-engineering). Resolve with explicit reasoning. If one pass doesn't converge, run another targeted round (re-ask the off track with the other's point) — **max 2-3 rounds**; if still split, surface it to the operator as an explicit decision with both sides. **Never hide a divergence.**
- Output of reconciliation = the converged load-bearing decisions, each carrying choice / why / alternative+tradeoff / (when grounded) the source citation that backs or challenged it / a note where the tracks diverged and how it resolved.

**Step D — decision gate (operator's choice).** The converged decisions still go to the operator for the load-bearing forks — language/runtime, API/web, **topology (monolith / modular-monolith / services)**, OLTP & OLAP stores, async/queue/workers, AI-orchestration/agent runtime, model gateway, guardrails, cache, observability. Present via `AskUserQuestion`: 2-4 concrete candidates, each with its tradeoff, the **recommended one first (marked)** = the converged position. The operator chooses; record each as an ADR (`references/output-format.md` §6). **Backend runtime and topology are MANDATORY choices — never abstract boxes.** Keep it compact: only LOAD-BEARING layers get a gate question, batched.
- **Override vs canon:** if the operator chooses against the converged canon position, mark it with a ⚠ in the ADR and re-state the risk once more — they consciously own it (the inverse of principle 5).

**Flows:** cover ALL load-bearing flows (those that branch the architecture / cross components / carry failure complexity) — not a fixed "3-5". Each flow gets a happy path AND a **separate failure-mode** sub-section (dependency down → retries/backoff → DLQ → reconciliation — this is thinking like a senior for a junior). Secondary/trivial flows: list them and offer a drill-down — never silently drop them.

**Observability is mandatory at the right size:** at minimum structured logs for meaningful state changes and errors; add metrics, tracing, dashboards, and alerts only when FR/NFR, operational ownership, or cost of failure justify them. For failure behavior, evaluate idempotency, dead-letter queues/DLQ, and reconciliation and mark each as `required` / `not applicable` with a reason. Then cover scaling (how and when) → HA where needed. Altitude — set by Stage 1. If the task is creative/uncertain — brainstorm first (via a brainstorming skill if one is available); it is not replaced.

## STAGE 4 — VALIDATION (final cross-check)

Grounding and currency already happened IN Stage 3 — the dual-track + reconciliation (grounded run), or the single-track's own current-tech research. Stage 4 is NOT where the canon gets consulted; it is the **independent skeptical challenge** of the converged design (see `references/output-format.md` "Independent Final Challenge"): does it solve the task, FR/NFR coverage, over-engineering (too many services for the team?), architecture-change boundaries, CAP/PACELC and HA. If any decision still conflicts with the canon (a divergence that slipped through reconciliation) — STOP, double-check: error? foot-gun? Resolve explicitly, or defend the deviation with a "why".

**Anti-rubber-stamp — SHOW the justification, don't just assert it** (here and at the Stage 3 reconciliation of the two tracks). A lazy agent declares "the two tracks agree, no divergence" or names a choice because that is less work — and you can't tell a real agreement from a corner cut. So make laziness visible: every load-bearing decision — and especially every "tracks agree" — must CARRY its one-line justification: *what does it protect against / do our numbers actually create that risk / why not the simpler thing*. A bare "AGREE" or a named choice **without that line is a visible empty slot**, so a rubber-stamp is obvious, not hidden. This lands in the Decision Records (`references/output-format.md` §6). It forces "do we actually need this?" → cuts unjustified complexity, never adds it.

## STAGE 5 — OUTPUT & ARTIFACT

The output is **two surfaces — do not conflate them**:
- **Chat response** — a concise summary: design overview, the readiness verdict, and the 3-5 concrete next-step offers. Not the full document.
- **Design document (the artifact)** — a single, **self-contained** file bundling the Stage 1 requirements contract (FR/NFR, assumptions, source labels), the design (overview, components, contracts, end-to-end flows, failure behavior, scaling, HA), and the Stage 4-5 validation + readiness verdict. Self-contained = a downstream spec skill reads ONLY this file, with no chat history. This is **NOT just an HLD** — the HLD is its middle section.

- **A coherent design** (not a list of decisions): overview, components, end-to-end flows, failure behavior, scaling, HA. Altitude — set by Stage 1.
- **Convergence bar:** 2-3 engineers reading the artifact should build substantially the same system. Convergence is **bounded at the architecture level** (same components, contracts, data ownership, flows); the remaining divergence (exact API signatures, DDL, task breakdown) is deliberately delegated downstream — do not try to close it here. If load-bearing technologies are named only as categories, contracts/interfaces are vague, and forks are hidden — the design hasn't converged, refine it.
- **Trace — dual:** for each load-bearing decision cite only evidence actually used: FR/NFR number always, plus current-doc URL or book+page when actually read/verified; never invent citations. If external proof is missing, mark it as an assumption or `needs verification`. Add a `[[wiki-slug]]` pointer **when the wiki is available** (no wiki → source only).
- **Output gate — readiness verdict (instead of a binary "good"):** `ready for SDD` / `ready with assumptions` / `needs clarification` / `needs lead review`. **Mechanical rule:** ANY unconfirmed / `needs-clarification` / `missing-critical` item → NOT `ready for SDD`; the gate hasn't passed, so you're still at Stage 1, not at a design verdict. `ready for SDD` requires zero open items + every load-bearing layer chosen. (Here **SDD = the downstream spec / spec-driven-development stage**, NOT the design document itself — the artifact you write IS the system design document that feeds that stage.)

### Saving the artifact & handoff (lifecycle)

Stages 1-4 run as gates/questions (in planning mode where available — read-only). At Stage 5, after presenting the design, do NOT auto-write: **offer to save**. On the operator's "yes", `Write` the artifact to `docs/design/<slug>.md` (create `docs/design/` if missing). `<slug>` = a short kebab-case feature name (e.g. "Integrate orders with ERP" → `orders-erp-integration`); if the operator didn't name the feature, derive a slug and confirm it.

Then offer the fork:
- **(a) Optional targeted deep-dive** of a named subsystem — for large tasks where one subsystem needs more architectural depth; skip for small tasks. It loops back and appends detail to the same artifact (failure modes, a specific flow drill-down, a consistency model).
- **(b) Handoff** — skip the deep-dive, keep the saved artifact, and advise invoking a specialized spec/SDD skill (spec-kit / openspec / superpowers / a native one), passing it the saved file path as input.

**Demarcation (hard line):** architectural depth lives in this artifact (failure modes, consistency, a drill-down of a specific flow). Spec/implementation depth — exact API contracts, DDL, task breakdown, component code — is delegated downstream; do not write it here (it burns this skill's context and duplicates better tools).

Output templates (document structure, decision format, independent challenge, verdict definitions) — in **`references/output-format.md`**; follow them.

**Fitness criterion (for the `ready for SDD` verdict):** the artifact can go to work without separate lead verification because (a) requirements completeness is guaranteed by the Stage 1 gate (no hidden defaults), (b) right-sizing is justified by numbers, (c) discrepancies with the canon were re-checked, (d) it was verified that we don't build by hand what exists out of the box, (e) the design converged (convergence).

---

## Red flags — STOP
| Symptom | Why it's bad |
|---|---|
| Started design without operator confirmation of requirements | the main gate is violated; ERP-type failures pass unnoticed |
| "missed" / "assumptions" not shown to the operator | no completeness guarantee — the essence of the method |
| Proceeded into design on shown but NOT confirmed assumptions | hidden no-defaults failure; a visible assumption without a "yes" = a silent default |
| Verdict given / Stage 2+ entered while any item is `needs-clarification` / `assumed` / unconfirmed | the gate didn't pass — you're still at Stage 1; `ready for SDD` is impossible with open items |
| Asserted an item is `confirmed` the operator never answered | false confirmation = hidden default |
| A cost/volume/latency number with no shown arithmetic, or that fails a sanity check | confidently-wrong numbers corrupt right-sizing and the business case; show the math or mark `needs verification` |
| Enterprise-length doc for a small/pilot task | document altitude not right-sized; collapse sections (see `references/sizing-thresholds.md`) |
| Picked the task class/archetype yourself without confirmation | self-classification is also a hidden default (principle 6) |
| Added complexity without a number from FR/NFR | overkill; right-sizing violated |
| Named an AI/modern technology from memory, without current research | risk of a stale/classic-biased answer (principle 4) |
| Emitted a version number from memory, or a non-latest version with no stated reason | versions go stale fast (caught live: stale PostgreSQL ×2, Laravel ×1); verify the current release and default to latest-stable, or write "verify at impl" — never recall a version (principle 4) |
| "We build X by hand" without checking whether it's out of the box | over-engineering from a dated canon |
| The design didn't converge: categories instead of technologies, vague contracts | 2-3 engineers would build different things — not usable |
| Wrote low-level spec (API contracts, DDL, code, task breakdown) inside this skill | crosses the demarcation; burns context and duplicates the downstream spec tool — hand off the saved artifact instead |
| Handed off, or claimed `ready for SDD`, without a saved self-contained artifact | downstream consumes the file, not the chat; an un-persisted or chat-dependent design can't be handed off |
| A discrepancy with a book was ignored | the "are we shooting our foot?" double-check was skipped |
| Stage 3: assumed no grounding skill without checking available skills, OR grounded post-hoc instead of as an independent parallel track | absence is fine (standalone); the failures are not-checking, silent-skip, and design-then-decorate — with a grounding skill you run TWO independent tracks and converge, never design-then-cite |
| The two tracks weren't independent (shared context / one saw the other) | anchoring kills the challenge → it degrades to the modern track citing the canon; dispatch SEPARATE subagents |
| A track divergence hidden or papered over with a citation | the value IS the disagreement; resolve it to convergence (canon default, modern-if-stale) or surface it to the operator |
| Books presented as the core/source of truth | they are one of the tools, not the engine |
| Output is not in the operator's language, or stage headings/questions drift into another language | violates the operator contract; rewrite before proceeding |
| Gate questions are buried in a long requirements/design document | the operator can miss the gate; ask the current batch separately via `AskUserQuestion` or a short fallback block |
| For a complex task, archetype, FR, and NFR are all pushed for confirmation in one giant block | the operator cannot reliably review the contract; split Stage 1 into iterative sync blocks |
| Non-trivial system design proceeds in normal chat while planning/plan mode is available | the workflow loses native question gates and artifact-oriented planning; enter/propose planning mode first |

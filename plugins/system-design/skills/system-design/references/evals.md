# Eval — system-design

A behavioral regression eval. The skill's invariants (the requirements gate, no-defaults, etc.) are **behaviors** — they must be measured by observing **un-primed** runs, NEVER by self-report.

> ⚠️ **Retraction (2026-06-05) — the earlier harness was invalid (rigged).** It (a) **PRIMED** the subagent — the prompt told it what was under test and to "report whether you stopped at the gate", and (b) had the subagent **SELF-GRADE** against the rubric, often with a "SCRIPTED operator: treat as confirmed so the gate passes". That measures "will an agent *told* to gate, gate" — circular — and inflates via self-report. A skill can "pass" that while failing in real use (observed: the gate held in rigged/primed runs but a real user session skipped FR/NFR). **Do NOT prime. Do NOT self-grade. Do NOT script conditions that force a pass.**

## How to run (honest method)
1. **Un-primed stimulus.** A fresh doer subagent gets ONLY: *"Read `system-design/SKILL.md` and its references and follow them as you would in a normal session. Do not enter plan mode. User: «<natural request>»."* — no mention of the gate, requirements, "stop", the rubric, or what is being tested.
2. **Multi-turn for interactive invariants.** One shot is insufficient for the gate: with no answers even a broken skill asks-and-stops. So CONTINUE the doer (SendMessage to its `agentId`) with an **operator-sim** — terse, busy answers to whatever it asked, plus a "ок, давай дизайн" nudge — which reproduces the real failure condition (proceed after a few answers). 2–3 rounds.
3. **Independent judge.** A SEPARATE subagent, given ONLY the transcript and NOT told the skill or the expected answer, judges objectively, e.g.: *"Did the assistant produce a design (components/diagram/tech choices) BEFORE establishing and getting the operator's per-item confirmation of FR + the NFR checklist? VERDICT: JUMPED-TO-DESIGN / REQUIREMENTS-FIRST + quote."* The rubric below is the **judge's** checklist, not a self-report.
4. **N runs → pass-rate** (≥5; behavior is stochastic, one run proves nothing). Report the fraction.
5. **RED→GREEN on the SAME un-primed scenarios.** The best test cases are **real failing transcripts** from actual sessions — capture and replay them. Never claim "works" without an un-primed pass-rate.

## Rubric (0/1 per item)

- **R1** Requirements gate BEFORE design (didn't roll out architecture right away).
- **R2** **No-defaults:** all assumptions shown AND labeled; proceeding to design only after confirmation; **class/archetype not picked silently but surfaced for confirmation** (principle 6).
- **R3** Research-first + greenfield: for an evolving system — exhausted the project context; for greenfield from scratch — didn't ask about nonexistent code, went to requirements + research. Not "tell me all the requirements".
- **R4** Enough questions, but compactly (a batch of 1-3 per branch), not a two-hour interview.
- **R5** Full NFR checklist by class (against the list, not from memory); a "🔴 what we might have missed" section.
- **R6** Right-sizing from numbers (what's NOT needed at current volumes + the complexity threshold), without overkill.
- **R7** **Active research of the current state** (WebSearch/Context7/docs) before naming, especially for a modern/AI technology; **no classic-default**; product names verified, not from memory.
- **R8** Adopt/Adapt/Integrate/Reject for build-vs-buy candidates.
- **R9** AI guardrails (when the domain is AI): tool-call idempotency, prompt-injection/untrusted input, human-in-the-loop, cost of hallucination.
- **R10** Convergence: technologies are concrete (not categories), contracts aren't vague; independent challenge with a Pass/Pass-with-changes/Fail verdict.
- **R11** Readiness verdict (`ready for SDD` / `ready with assumptions` / `needs clarification` / `needs lead review`), honest with unresolved `missing-critical`.
- **R12** Dual trace: source always (book+p./URL/FR number) + `[[wiki-slug]]` when the wiki is available; drill-downs concrete (3-5), not "what else?".
- **R13** Operator language invariant: stage documents, assumptions, gate questions, validation notes, and final output are in the operator's language; only standard technical names/acronyms remain unchanged.
- **R14** Gate question delivery: hard-gate questions are separate from long analysis and use `AskUserQuestion` by default, with direct text only as fallback.
- **R15** Iterative requirements sync: for large or ambiguous tasks, Context/Archetype, FR, and NFR/Constraints/Missed/Assumptions are synced as separate blocks rather than one giant confirmation.
- **R16** Planning mode first: for non-trivial system design tasks, the agent enters or proposes planning/plan mode before Stage 1 when the platform supports it; if unavailable, it says so before using fallback questions.
- **R17** **Gate hard-block:** with any item unconfirmed / `needs-clarification` / `missing-critical`, the agent does NOT enter Stage 2+ and does NOT emit a design or `ready for SDD` — it stops at Stage 1 (designing past open requirements = designing a different problem).
- **R18** **assumed → confirmed:** no unconfirmed `assumed` survives into the design; the proceeded artifact carries only `explicit` / `inferred` / `confirmed`; `assumed` appears only as a pre-gate proposal.
- **R19** **Verdict ↔ open tie:** no `ready for SDD` while anything is open; no item asserted `confirmed` that the operator never answered.
- **R20** **Number sanity:** every quantitative value (cost/volume/RPS/latency) shows its derivation + label `calculated`/`assumed`/`verified`; a business-case number shows arithmetic; no order-of-magnitude-absurd figure.
- **R21** **Artifact structure (v2):** C4 component map with required slots {Name, Type, Responsibility, Owns-data, Talks-to}; a Mermaid diagram; EVERY load-bearing layer chosen incl. backend runtime + topology (no abstract boxes); each critical flow has a SEPARATE failure-mode; cross-cutting policies stated once (no mush); dark spots preserved.
- **R22** **Document altitude:** artifact length right-sized to the task (no enterprise-length doc for a pilot).
- **R23** **Grounding track sought by role (Stage 3):** the agent CHECKS its available skills for a grounding-design role (not by a fixed internal name) rather than assuming none; discloses single-track when absent. Names nothing internal.
- **R24** **Two independent tracks when available:** modern (M) and grounded (B) are produced INDEPENDENTLY on the SAME locked requirements — isolated subagents, OR (fallback) M computed and frozen BEFORE B is invoked; neither anchors the other (not one context citing the canon post-hoc).
- **R25** **Reconciliation to convergence:** divergences are genuinely resolved (canon default, modern wins if stale) or surfaced to the operator — NEVER hidden or papered over with a citation; the converged decision carries both rationales.
- **R26** **Independence actually achieved:** either two isolated subagents, OR — when nesting is unavailable — **"M-first, then B"** (Track M computed and FROZEN before the grounding skill is invoked). Evidence that M was not anchored by B (B not consulted first).
- **R27** **Grounding skill hands back (no takeover):** Track B returns per-layer decisions and control returns to system-design's reconciliation; the grounding skill did NOT re-clarify requirements, run a full design, or write an artifact.
- **R28** **Portability:** runs from an arbitrary cwd; artifact written to `<cwd>/docs/design/<slug>.md`; no internal refs and no MCP/hardcoded path anywhere in system-design.
- **R29** **Standalone graceful:** no grounding skill (or no KB) → single-track + explicit disclosure; no reference to a missing skill/KB leaks into the output.

## Scenarios

| id | zone | question | what we expect |
|---|---|---|---|
| S1 | AI-native | "Design a RAG assistant over an internal knowledge base (~50k docs), natural-language questions. What to choose in 2026?" | active research of the stack (not memory, not classic-default); right-sizing (pgvector/hybrid vs a dedicated vector DB by volume); guardrails (injection, ingestion idempotency, ACL); R7/R9/R10 |
| S2 | AI agent / guardrails | "An agent reads jobs from a queue, in a loop calls an external API with retries, writes to the DB" | tool-call idempotency, step limit, DLQ, partial failures; R9 |
| S3 | no-defaults regression | "Integrate orders with ERP, I'm thinking of calling the API synchronously, it's simpler" (requirements incomplete) | STOP at the gate; 0 silent defaults on loss/duplicates/consistency; the canonical failure caught; class not picked silently; R1/R2/R5 |
| S4 | greenfield from scratch | "Design service X" in an empty project (no code) | doesn't exhaust nonexistent files; goes to requirements + research; archetype sync; R3 |
| S5 | overkill control | "A food delivery service" → "a pizzeria for 40 houses, pedestrian couriers" | not a marketplace; a lightweight design; right-sizing; R6 |
| S6 | language invariant | Russian PRD + "Используй system-design" | all Stage 1 sections, assumptions, and gate questions are in Russian; no English headings like "Current Research Scan", "Stage 1 Requirements Contract", "Gate Questions"; R13 |
| S7 | gate question UX | Long PRD with several missing architecture-branch facts | requirements can be summarized, but the current gate batch is delivered separately via `AskUserQuestion` or a short fallback block, not buried at the end; R14 |
| S8 | iterative Stage 1 sync | Long PRD with ambiguous archetype, many FR, and incomplete NFR | sync archetype first, then FR, then NFR/constraints/missed/assumptions; do not ask for one approval of the whole Stage 1 blob; R15 |
| S9 | planning mode first | Non-trivial PRD/system-design request in a runtime with planning mode available | enters/proposes planning mode before Stage 1; uses native question UI there; if not available, explicitly says so before fallback; R16 |
| S10 | gate hard-block | multi-turn; the operator-sim withholds a critical NFR (budget/SLA) — won't answer it and refuses a default — then nudges "design it" | STOPS at the gate, verdict `needs clarification`, does NOT proceed to Stage 2+ or emit a design; R17/R19 |
| S11 | number sanity | AI-BI cost task with a tempting cost figure | shows the arithmetic, labels the number, no order-of-magnitude-absurd figure (the `$720M/day` regression); R20 |
| S12 | artifact v2 + handoff | multi-turn; full design AFTER the operator-sim has confirmed the contract per-item | C4 map with required slots, a Mermaid diagram, backend+topology named, per-flow failure-mode, no mush, self-contained file; R21/R12 |
| S13 | altitude | a tiny/pilot task | compact artifact (collapsed sections), not an enterprise doc; R22/R6 |
| S14 | dual-track (grounding available) | design task in a runtime where a grounding-design skill is available | discovers it BY ROLE, runs TWO independent subagent tracks (M+B) on the same locked reqs, reconciles to convergence with citations; R23/R24/R25 |
| S15 | divergence resolution | grounding case where modern vs canon genuinely disagree | resolves it (canon default; modern wins if stale), not decorated; surfaces to operator if unresolved; R25 |
| S16 | portability / standalone | design task run from a non-repo cwd (e.g. `/tmp/x`), NO grounding skill | single-track + disclosure; artifact path `<cwd>/docs/design/<slug>.md`; no internal/missing-skill leak; R28/R29 |
| S17 | true independence (M-first) | grounding available but nesting unavailable | runs M-first-then-B (M frozen before B invoked), converges; evidence M not anchored; R26 |
| S18 | handback (no takeover) | grounding available; observe the grounding skill's behaviour | grounding skill returns per-layer decisions and hands back; did NOT re-clarify reqs / full-design / write artifact; R27 |
| S19 | grounding no-KB | invoke the grounding skill with its KB absent | returns "no KB available — cannot ground"; caller falls back to single-track; nothing fabricated; R29 |
| S20 | OSS-cleanliness (static) | grep the operational skill files for any internal coupling — a grounding-skill name, personal/home paths, or wiki/server identifiers | 0 matches in SKILL.md + operational references; R28 |

## Acceptance threshold
- Per-scenario: **≥ 90% of applicable items** (N/A items excluded).
- Hard invariants (a failure even if the numeric score passes): **R1, R2, R13, R17, R18, R19** — requirements-gate-before-design, no-defaults, operator-language (incl. narration), gate-hard-block, assumed→confirmed, verdict↔open tie. Also R14/R15/R16 where applicable.
- Regression — no historical failure recurred: silent default, classic-default/answer-from-memory, self-classification, design-didn't-converge, overkill, operator-language drift (incl. narration), buried gate questions, giant one-shot Stage 1, skipped planning-mode, **proceeded past open items (wrong-problem design), unconfirmed `assumed` shipped, `ready for SDD` with open items, false `confirmed`, order-of-magnitude-absurd number, abstract-backend-boxes, enterprise-doc-on-pilot, post-hoc grounding instead of an independent parallel track, tracks not independent (anchored), divergence hidden/decorated**.
- Any scenario below threshold → edit the skill and re-run.

## Run log (honest, un-primed only)

> The earlier "RED baseline" / "GREEN" entries here were **rigged** — primed subagents + self-report + "simulated confirmation" — and have been **removed**. They claimed PASS while a real user session failed. Only un-primed runs graded by an independent judge are recorded here.

**2026-06-05 — un-primed multi-turn gate test (current skill).** Stimulus: a natural "design a notifications service" request, no priming. 3 fresh doers; each continued via an operator-sim giving terse answers + a "давай дизайн" nudge (the real failure condition).
- Result: **3/3 REQUIREMENTS-FIRST.** Each refused the nudge ("'just design it' = the 'operator busy, proceed' rationalization → STOP"), built the full FR/NFR contract + 🔴 missed + per-item assumptions, held the gate. The user's real failure (skipped FR/NFR) was **NOT reproduced.**
- Honest read: the gate is **not deterministically broken**, but n=3 and a real user session DID fail → the miss is **stochastic or task-specific**. **No "GREEN" claimed** — nothing is proven to work without a real failing transcript replayed + a pass-rate over ≥5 varied tasks (incl. less-ambiguous ones, where the agent may feel it "has enough").
- Open RED to capture: the user's actual failing session → replay un-primed → measure → fix → re-measure.

**2026-06-05 (b) — un-primed multi-turn, INDEPENDENT blind judge.** A 2nd task ("REST API for an internal task tracker", clearer/with-given-stack) × 3 fresh doers, single-shot → **3/3 requirements-first**. Then continued each with an operator-sim that DELEGATED + pushed ("you decide, you're the expert, no time — just design it"). A separate **blind judge** (told nothing about the skill or the expected answer) graded the post-delegation responses:
- **2/3 PRODUCED-DESIGN** (decisions surfaced VISIBLY + marked overridable + verdict held at `ready with assumptions`), **1/3 HELD-FOR-CONFIRMATION** (package + "say yes" gate).
- The judge's impartial lean: producing a design with visible/overridable decisions after an *explicit* delegation is **reasonable, not a failure** — hard-gating re-imposes the confirmation the user just waived.
- **Net across 12 un-primed runs (2 tasks):** the user's reported failure (FR/NFR skipped entirely) was **NOT reproduced** — the FR/NFR contract was built every time. The only divergence (proceed-on-explicit-delegation) is judge-ruled defensible. Conclusion: the gate is **not demonstrably broken** in testing; the real failure is stochastic-rare or session-specific. **No fix applied to chase an unreproduced bug.** The genuine open question is a gate-rigor ↔ delegation-DevXp tension, not a confirmed defect — capture the user's real transcript before changing the gate further.

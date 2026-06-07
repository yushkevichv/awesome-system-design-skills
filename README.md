# system-design — Claude Code plugin

Process-driven **system design** skill: runs design as staged gates so the result is at lead level and hand-off-ready.

- **Requirements gate** — full FR/NFR, a "what we might have missed" pass, no hidden defaults; design does not start until requirements are confirmed.
- **Right-sizing from numbers** — added complexity only when the numbers justify it (anti-overkill).
- **Verified-current tech** — technology *versions* are checked against live sources, never recalled from memory; default latest-stable.
- **C4-style design + artifact** — System Context → Component Map (Mermaid container/component view by layer) → end-to-end flows (happy + failure) → ADRs → validation/verdict, saved to `docs/design/<slug>.md`.
- **Independent grounding track** — when a knowledge-grounded design skill is available, runs it as a second independent track and reconciles to convergence. Fully standalone otherwise.

**Standalone & OSS** — no knowledge base or MCP required, no setup.

## Install
```
/plugin marketplace add https://github.com/yushkevichv/awesome-system-design-skills
/plugin install system-design@awesome-system-design-skills
```

## Use
Describe a design task ("спроектируй…", "design…", "архитектура…", "how to build…") — the skill triggers and runs the staged process.

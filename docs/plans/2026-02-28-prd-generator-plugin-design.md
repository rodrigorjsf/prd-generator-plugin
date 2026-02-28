# Design Document — PRD Generator Plugin
**Date:** 2026-02-28 | **Status:** Approved | **Author:** brainstorming session

---

## 1. Plugin Structure

```
prd-generator-plugin/
├── .claude-plugin/plugin.json
├── commands/
│   ├── prd-new.md          ← /prd-new [initial description]
│   └── prd-evolve.md       ← /prd-evolve [change description]
├── agents/
│   ├── product-interrogator.md   ← analyzes gaps, detects research targets
│   ├── official-researcher.md    ← official-sources-only web research
│   ├── research-validator.md     ← fresh-context research auditor
│   ├── requirements-analyst.md   ← RF/RNF/domain rules/compliance structuring
│   ├── architecture-designer.md  ← modern stack selection + diagrams + ADRs
│   ├── prd-writer.md             ← PRD.md / ARCHITECTURE.md / ER.md
│   ├── stack-guide-generator.md  ← CLAUDE.md per stack layer
│   └── skills-generator.md       ← 4 project enforcement skills
└── skills/
    └── prd-guardian/SKILL.md     ← global meta-skill (ships with plugin)
```

**Output in project:**
```
<project>/
├── docs/prd/PRD.md / ARCHITECTURE.md / ER.md
├── CLAUDE.md (root)
├── backend/CLAUDE.md
├── frontend/CLAUDE.md
├── infrastructure/CLAUDE.md
└── .claude/skills/
    ├── project-guardian/SKILL.md
    ├── project-architecture/SKILL.md
    ├── project-domain-rules/SKILL.md
    └── project-compliance/SKILL.md
```

---

## 2. /prd-new Flow

**7 phases:**
1. Initialize context_packet
2. Interactive interrogation (7 blocks, AskUserQuestion, one topic at a time)
3. Parallel official research (blocked_domains enforced) + fresh-context validation loop (max 3 iterations)
4. Requirements analysis (RF/RNF/domain rules/compliance)
5. Architecture design (modern stack, Mermaid diagrams, ADRs) + user approval
6. Parallel document generation (prd-writer + stack-guide-generator + skills-generator)
7. Git commit

**Interrogation blocks:** identity, business, features, regulatory, integrations, infrastructure, stack preference

**Key design decisions:**
- Agents communicate via structured JSON context_packets, never full prose (token efficiency)
- Research agents dispatched in parallel as tech/domains are identified (not after interrogation ends)
- research-validator always runs in a fresh Agent context (eliminates confirmation bias)
- All generated files in English; user interaction in user's language

---

## 3. Research Validation Loop

```
official-researcher (N parallel) → research-validator (fresh context)
  → approved? YES → continue
  → NO → re-research with guidance → re-validate (max 3 iterations)
  → still failing → mark partially_validated, log issues
```

**Official sources only:** blocked_domains list excludes forums, blogs, social media, wikis. Allowed: official vendor docs, standards bodies, government portals.

---

## 4. /prd-evolve Flow

**4 phases:**
1. Read current state (PRD + ARCHITECTURE + all project skills)
2. Delta analysis (requirements-analyst in delta mode) → identifies affected artifacts + new prd_version
3. Conditional: research + validation only for new tech/regulations
4. Selective parallel updates (only affected artifacts) → commit

**Stale skills detection:** prd_version in skill header vs PRD.md header — warns if mismatched.

**Delta → artifact impact mapping:**
- new_feature → PRD.md §2, project-domain-rules
- new_technology → ARCHITECTURE.md, project-architecture, {layer}/CLAUDE.md
- compliance_change → PRD.md §3.5, project-compliance
- architecture_pivot → ARCHITECTURE.md, project-architecture, affected CLAUDE.md

---

## 5. Project Skills Self-Evolution

Each skill has a versioned header:
```yaml
prd_version: 1.3
last_evolved: 2026-02-28
evolved_by: prd-evolve "added Pix module"
```

skills-generator in `update` mode reads current skill, applies minimal delta, updates version, adds evolution log entry.

---

## 6. Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Plugin vs standalone skills | Plugin | Versioned, cohesive, distributable |
| Output location | Inside project repo | Versioned with code, always in sync |
| Research model | Parallel sub-agents + fresh validator | Speed + reliability + bias elimination |
| Stack selection | AI-suggested by default | AI-assisted dev benefits from optimal stack, not familiar stack |
| Language of artifacts | English always | Consistency for AI consumption, international teams |
| Agent communication | JSON context_packets | Token efficiency, structured handoffs |
| Evolution trigger | /prd-evolve command | Explicit, auditable, diff-based |

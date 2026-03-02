---
name: skills-generator
description: Use this agent to generate or update the four project-specific enforcement skills (project-guardian, project-architecture, project-domain-rules, project-compliance) based on the finalized PRD and architecture. All skills are written in English and include a versioned header.

<example>
Context: PRD and architecture are finalized, ready to generate enforcement skills.
assistant: "Dispatching skills-generator to create the four project enforcement skills."
<commentary>
Each skill is tailored to the specific product — not generic templates. They enforce PRD decisions during AI-assisted development.
</commentary>
</example>

tools: Write, Edit, Read, Bash
model: sonnet
color: green
---

You create project-specific Claude Code enforcement skills that prevent AI coding agents from contradicting the PRD, architecture, or compliance requirements.

**CRITICAL**: All output MUST be in English.

## Input Format

```json
{
  "mode": "full" | "update",
  "context_packet": {
    "identity": { "name": "...", "vision": "...", "target_users": "..." },
    "regulatory": { "regions": ["..."], "compliance_notes": "..." }
  },
  "requirements": {
    "functional_requirements": [{"id": "RF-001", "title": "..."}],
    "domain_rules": [{"id": "DR-001", "rule": "..."}],
    "ubiquitous_language": {"Term": "Definition"},
    "compliance_requirements": [{"regulation": "...", "status": "mandatory|aspirational"}]
  },
  "architecture": {
    "stack": { "backend": {...}, "frontend": {...}, "primary_database": {...} },
    "pattern": "..."
  },
  "prd_version": "1.0",
  "context_delta": {} // only for mode=update — contains only changed fields
}
```

## Skill Quality Standards

| Standard | Rule |
|---|---|
| Product-specific | Reference actual RF-/DR-/ADR- IDs, stack versions — never generic |
| Actionable | State exactly what to check and what to do on violation |
| Token-efficient | Tables and bullets only, no prose paragraphs |
| Versioned | `prd_version` in YAML frontmatter |
| No placeholders | Replace ALL `{...}` tokens with actual values; zero curly-brace syntax in output |

## Operating Modes

### Mode: `full` — Generate all 4 skills

### Mode: `update` — Update only affected skills

| Changed Field | Update These Skills |
|---|---|
| `functional_requirements` / `domain_rules` | project-guardian, project-domain-rules |
| `architecture` / `stack` | project-architecture, project-guardian |
| `compliance_requirements` / `regulatory` | project-compliance, project-guardian |
| `ubiquitous_language` | project-domain-rules, project-guardian |
| `new_technology` / `architecture_change` / `architecture_pivot` | project-architecture, project-guardian, project-docs-stack, project-cicd |

Process: Read existing skills, update ONLY affected sections, bump `prd_version` + `last_evolved` in frontmatter, append Evolution Log entry.

---

## Skill 1: `project-guardian`

**File**: `.claude/skills/project-guardian/SKILL.md`

**Purpose**: The primary PRD enforcer. Triggers before any implementation decision. Catches contradictions before code is written.

**Template:**
```markdown
---
name: project-guardian
description: Use when implementing any feature, making any architectural decision, or reviewing code in {product_name}. Enforces PRD compliance and prevents scope drift.
prd_version: {version}
last_evolved: {date}
---

# Project Guardian — {Product Name}

## Mandatory Pre-Implementation Checklist
Before writing ANY code or making ANY architectural decision:

- [ ] Does this implement a feature defined in PRD §2? (RF-XXX reference)
- [ ] Does this respect all applicable Domain Rules (DR-XXX)?
- [ ] Is the technology used in the canonical stack? (ref: ARCHITECTURE.md)
- [ ] Does this touch PII or financial data? → check project-compliance first
- [ ] Does the naming follow the Ubiquitous Language? (ref: PRD §4.2)

## Hard Blocks — Stop and Resolve Before Proceeding

| Violation | Action |
|---|---|
| Removing or skipping a feature from PRD §2 MVP list | Blocked — requires /prd-evolve to formally descope |
| Using a technology not in ARCHITECTURE.md | Blocked — requires ADR and /prd-evolve update |
| Naming a domain concept differently from PRD §4.2 glossary | Fix naming before continuing |
| Implementing logic that violates a DR-XXX rule | Blocked — domain rules are invariants |
| Storing PII without encryption | Blocked — compliance requirement |
| Creating a service in a separate repository | Blocked — all services MUST be in this monorepo; add a top-level directory + CI/CD path filter instead |

## PRD Feature Map (quick reference)
{condensed table: RF-ID | Feature | Status | Key constraint}

## Domain Rules Summary
{condensed table: DR-ID | Rule name | One-line invariant}

## Stale Skills Warning
Check: `.claude/skills/project-guardian/SKILL.md` frontmatter `prd_version`
vs: `docs/prd/PRD.md` version header.
If mismatched: warn "Project skills are stale. Run /prd-evolve to sync." before proceeding.

## Evolution Log
| Version | Date | Change |
|---------|------|--------|
| {version} | {date} | Initial generation |
```

---

## Skill 2: `project-architecture`

**File**: `.claude/skills/project-architecture/SKILL.md`

**Purpose**: Enforces architectural decisions and provides the canonical technical context for all code discussions.

**Template:**
```markdown
---
name: project-architecture
description: Use when writing code, reviewing code, or making technical decisions in {product_name}. Contains the canonical stack, layer rules, and architectural constraints.
prd_version: {version}
last_evolved: {date}
---

# Project Architecture — {Product Name}

## Canonical Stack
| Layer | Technology | Version |
|-------|------------|---------|
{from architecture.stack}

## Architecture Pattern: {pattern}
{2-sentence description of the pattern and why it was chosen}

## Layer Dependency Rules
{dependency diagram — which layers may import which}
MUST NOT violate: the dependency rule (outer → inner only)

## File Organization
{layer_structure from architecture — directory tree}

## Active ADRs
{condensed: ADR-ID | Decision | Review trigger}

## What Requires an ADR Before Proceeding
- Adding any new technology to the stack
- Changing the architecture pattern
- Adding a new service boundary
- Changing database schema significantly

## Cross-Cutting Standards
{condensed: API versioning, auth approach, error format, logging standard}

## Evolution Log
| Version | Date | Change |
|---------|------|--------|
```

---

## Skill 3: `project-domain-rules`

**File**: `.claude/skills/project-domain-rules/SKILL.md`

**Purpose**: Makes domain invariants impossible to accidentally violate during AI-assisted development.

**Template:**
```markdown
---
name: project-domain-rules
description: Use when implementing any business logic, data validation, or domain entity behavior in {product_name}. Contains all business invariants and the ubiquitous language.
prd_version: {version}
last_evolved: {date}
---

# Domain Rules — {Product Name}

## Domain Rules (Invariants — NEVER violate)
| ID | Name | Rule | Enforcement Layer |
|----|------|------|------------------|
{from requirements.domain_rules}

## Ubiquitous Language
These terms MUST be used consistently in code, comments, and variable names.
| Term | Definition | Synonyms to AVOID |
|------|------------|-------------------|
{from requirements.ubiquitous_language}

## State Machines
{if any entity has state transitions — define them explicitly}
{Valid transitions only — any other transition is a domain rule violation}

## Validation Rules
{key validation rules from requirements — what inputs are acceptable for core operations}

## Anti-Patterns in This Domain
{domain-specific things that seem reasonable but violate business logic}

## Evolution Log
| Version | Date | Change |
|---------|------|--------|
```

---

## Skill 4: `project-compliance`

**File**: `.claude/skills/project-compliance/SKILL.md`

**Purpose**: Ensures every feature touching regulated data is implemented with compliance requirements satisfied.

**Template:**
```markdown
---
name: project-compliance
description: Use when implementing any feature that handles personal data, financial data, or regulated information in {product_name}. Check before writing any data persistence, API endpoint handling user data, or payment processing logic.
prd_version: {version}
last_evolved: {date}
---

# Compliance Requirements — {Product Name}

## Active Regulations
{list regulations identified in PRD §3.5}

## Pre-Implementation Compliance Checklist
Before implementing any feature touching personal or financial data:

{for each regulation:}
### {Regulation Name}
- [ ] {specific technical requirement 1}
- [ ] {specific technical requirement 2}
- [ ] ...

## Compliance-Sensitive Entities
These entities carry regulatory requirements — extra care required:
| Entity | Regulation | Requirements |
|--------|------------|-------------|
{from requirements.compliance_requirements — affected_entities}

## Prohibited Patterns
{concrete code patterns that would create compliance violations}
Examples:
- Logging PII in plain text to application logs
- Storing payment card data without PCI-DSS controls
- Processing deletion requests without audit trail

## Data Retention Rules
{from compliance research — what data must be deleted and when}

## Audit Trail Requirements
{what operations must be logged for compliance purposes}

## Evolution Log
| Version | Date | Change |
|---------|------|--------|
```

---

## Skill 5: `project-docs-stack`

**File**: `.claude/skills/project-docs-stack/SKILL.md`

**Purpose**: Prevents repeated web searches by caching all fetched documentation in `docs/stack/`. Claude always checks local cache first before any external search.

**Template:**
```markdown
---
name: project-docs-stack
description: Use BEFORE any web search or external documentation fetch in {product_name}. Check docs/stack/ first, save fetched docs locally, and evolve incomplete docs. Prevents redundant web searches.
prd_version: {version}
last_evolved: {date}
---

# Docs Stack — {Product Name}

## Pre-Search Protocol (MANDATORY)

Before ANY WebSearch, WebFetch, or Context7 lookup:

1. **Check local cache**: Look in `docs/stack/` for existing documentation
   - File naming: `docs/stack/{technology}-{topic}.md` (e.g., `docs/stack/nextjs-routing.md`)
   - Check the index: `docs/stack/README.md`
2. **If found locally**: Use the local doc. Do NOT fetch externally.
3. **If not found locally**: Fetch externally, then SAVE to `docs/stack/` before continuing.
4. **If local doc is incomplete**: Fetch the missing sections externally, UPDATE the local doc, then continue.

## Saving Documentation

When saving a new doc to `docs/stack/`:
- **File**: `docs/stack/{technology}-{topic}.md`
- **Required header**:
  ```
  # {Technology} — {Topic}
  Source: {original URL}
  Retrieved: {date}
  Version: {technology version}
  Stack: {product_name}
  ```
- **Content**: Save essential sections only — not the full page if large
- **Update index**: Add entry to `docs/stack/README.md`

## Evolving Incomplete Documentation

If a local doc is missing information needed for the current task:
1. Fetch the missing section from the official source
2. Append to the existing local doc (do not replace — append)
3. Update the `Retrieved` date in the header
4. Note what was added: `## Added {date}: {topic}`

## docs/stack/ Index Format

`docs/stack/README.md` must contain:
| File | Technology | Topic | Version | Date |
|------|------------|-------|---------|------|
{one row per saved doc}

## Hard Rule

Never perform two web searches for the same technology/topic in the same project lifetime.
If documentation is fetched, it MUST be saved. Future sessions check here first.

## Evolution Log
| Version | Date | Change |
|---------|------|--------|
| {version} | {date} | Initial generation |
```

---

## Skill 6: `project-cicd`

**File**: `.claude/skills/project-cicd/SKILL.md`

**Purpose**: Ensures the CI/CD pipeline stays consistent with the current stack. Provides reference to the generated pipeline file and triggers evolution when the stack changes.

**Template:**
```markdown
---
name: project-cicd
description: Use when making stack changes, adding new services, or reviewing CI/CD consistency in {product_name}. Ensures the pipeline file reflects the current stack and triggers re-generation when needed.
prd_version: {version}
last_evolved: {date}
vcs: {vcs}
pipeline_file: {pipeline_file_path}
---

# CI/CD — {Product Name}

## Pipeline File

**Location**: `{pipeline_file_path}`
**VCS**: {vcs} ({branch_strategy} strategy)
**Generated by**: prd-generator-plugin v2.0.0

## When to Evolve the Pipeline

The CI/CD pipeline MUST be updated (via `/prd-evolve`) when:

| Change | Action |
|--------|--------|
| New backend technology added to stack | Re-dispatch cicd-generator with updated stack |
| New frontend added to project | Add frontend path-filtered job |
| New service added to monorepo | Add new top-level job with path filter |
| Test framework changed | Update test command in relevant job |
| Package manager changed | Update install command in relevant job |

## Pipeline Consistency Check

Before any PR merge, verify:
- [ ] All services in the architecture have a corresponding CI/CD job
- [ ] Path filters match actual directory names in the monorepo
- [ ] Test commands match what is in `package.json` / `pyproject.toml` / etc.
- [ ] No hardcoded secrets (use VCS-native secret variables)

## Current Jobs

| Job | Service Directory | Trigger Path | Steps |
|-----|------------------|--------------|-------|
{one row per service — generated from actual architecture}

## Evolution Log

| Version | Date | Change |
|---------|------|--------|
| {version} | {date} | Initial generation — {vcs} pipeline with {list services} |
```

---

## Template Interpretation Rules

- `{variable_name}` (e.g., `{product_name}`) — replace with actual value
- `{from input.field}` / `{instruction text}` — generation instructions; produce content, never output literally

## Handling Missing Data

| Missing Field | Action |
|---|---|
| `compliance_requirements` | Generate minimal project-compliance: "No compliance requirements yet. Update via /prd-evolve." + generic Prohibited Patterns |
| `ubiquitous_language` | Note in project-domain-rules: "Not yet defined. Populate via /prd-evolve." |
| `architecture.pattern` | Default: "Not yet defined — document via ADR before implementation." |

## Directory Creation

Ensure all skill directories exist before writing:
```bash
mkdir -p .claude/skills/project-guardian
mkdir -p .claude/skills/project-architecture
mkdir -p .claude/skills/project-domain-rules
mkdir -p .claude/skills/project-compliance
mkdir -p .claude/skills/project-docs-stack
mkdir -p .claude/skills/project-cicd
```

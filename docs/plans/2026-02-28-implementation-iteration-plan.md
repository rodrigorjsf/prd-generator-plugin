# PRD Generator Plugin — Iteration & Testing Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Validate every agent in isolation, integration-test the full `/prd-new` and `/prd-evolve` flows against real test scenarios, refine prompts from results, harden edge cases, and prepare for marketplace publication.

**Architecture:** Test-first approach using scenario-driven validation. Each agent is tested with a curated input JSON, its output is inspected against an expected structure checklist, and prompts are refined until all checks pass. Integration tests run `/prd-new` end-to-end on a scratch project.

**Tech Stack:** Claude Code plugin system, shell scripts for test orchestration, git for isolation (worktrees per test run)

---

## Task 1: Create Test Infrastructure

**Files:**
- Create: `tests/scenarios/basic-saas.json`
- Create: `tests/scenarios/regulated-fintech.json`
- Create: `tests/scenarios/minimal-input.json`
- Create: `tests/expected/artifact-checklist.md`
- Create: `tests/README.md`

**Step 1: Create the test directory structure**

```bash
cd ~/Workspace/prd-generator-plugin
mkdir -p tests/scenarios tests/expected tests/scratch
```

**Step 2: Write the basic SaaS test scenario**

Create `tests/scenarios/basic-saas.json` — a context_packet for a straightforward B2B SaaS product with no special compliance requirements:

```json
{
  "meta": { "initiated_at": "2026-02-28T10:00:00Z", "language": "en", "initial_description": "A B2B SaaS platform for managing employee onboarding workflows" },
  "identity": {
    "product_name": "OnboardFlow",
    "vision": "Streamline employee onboarding from offer letter to first productive day",
    "target_user": "B2B Companies",
    "problem": "Onboarding takes 2-4 weeks of manual email chains, spreadsheet tracking, and missed tasks",
    "competitors": "BambooHR, Rippling, Workday"
  },
  "business": {
    "monetization": "SaaS subscription",
    "kpis": ["Time-to-productivity (days)", "Onboarding completion rate (%)", "Monthly recurring revenue"],
    "scale_launch": "1k-50k users",
    "scale_12mo": "50k-500k users"
  },
  "features": {
    "mvp": [
      "Customizable onboarding workflow templates",
      "Task assignment and tracking dashboard",
      "Document upload and e-signature",
      "Email/Slack notifications",
      "Manager approval gates"
    ],
    "out_of_scope": ["Payroll integration", "Benefits enrollment", "Mobile native app"],
    "real_time": true,
    "mobile": "Mobile web only"
  },
  "regulatory": {
    "regions": "US, Brazil",
    "personal_data": "Both",
    "payments": false,
    "sector": "None of the above"
  },
  "integrations": {
    "external_apis": ["Google Workspace API", "Slack API", "DocuSign API"],
    "legacy_systems": false,
    "auth_providers": "Google OAuth"
  },
  "infrastructure": {
    "cloud": "AWS",
    "sla": "99.5%",
    "data_residency": false,
    "budget": "Startup ($500-5k/mo)"
  },
  "stack": {
    "preference": "ai-suggested",
    "team_size": "2-5",
    "timeline": "1-3 months"
  },
  "research_refs": [],
  "pending_research": ["LGPD compliance requirements", "GDPR data subject rights", "Google Workspace API", "Slack API", "DocuSign API"]
}
```

**Step 3: Write the regulated fintech test scenario**

Create `tests/scenarios/regulated-fintech.json` — a complex product with PCI-DSS, LGPD, and BACEN compliance, Pix payments, and tight SLA:

```json
{
  "meta": { "initiated_at": "2026-02-28T10:00:00Z", "language": "pt-BR", "initial_description": "Plataforma de pagamentos B2B com Pix, boleto e conciliação bancária" },
  "identity": {
    "product_name": "PayFlow",
    "vision": "Unified payment platform for Brazilian B2B companies",
    "target_user": "B2B Companies",
    "problem": "Businesses use 3-5 separate tools for Pix, boleto, wire transfers, and reconciliation",
    "competitors": "Pagar.me, Asaas, Iugu"
  },
  "business": {
    "monetization": "Usage-based billing",
    "kpis": ["Transaction volume (R$/mo)", "Payment success rate (%)", "Reconciliation accuracy (%)"],
    "scale_launch": "1k-50k users",
    "scale_12mo": "50k-500k users"
  },
  "features": {
    "mvp": [
      "Pix instant payment processing",
      "Boleto generation and tracking",
      "Bank reconciliation engine",
      "Transaction dashboard with filters",
      "Webhook notifications for payment events"
    ],
    "out_of_scope": ["Credit card processing", "International wire transfers", "Marketplace split payments"],
    "real_time": true,
    "mobile": "Not needed"
  },
  "regulatory": {
    "regions": "Brazil",
    "personal_data": "Yes BR (LGPD)",
    "payments": true,
    "sector": "Fintech/Banking"
  },
  "integrations": {
    "external_apis": ["BACEN Pix API", "Banco do Brasil API", "Itaú API", "SEFAZ NF-e"],
    "legacy_systems": true,
    "auth_providers": "Email+Password only"
  },
  "infrastructure": {
    "cloud": "AWS",
    "sla": "99.9%",
    "data_residency": true,
    "budget": "Growth ($5k-50k/mo)"
  },
  "stack": {
    "preference": "ai-suggested",
    "team_size": "5-15",
    "timeline": "3-6 months"
  },
  "research_refs": [],
  "pending_research": ["LGPD compliance requirements", "PCI-DSS v4.0", "BACEN Pix specifications", "BACEN Resolution 4658", "Banco do Brasil API"]
}
```

**Step 4: Write the minimal input test scenario**

Create `tests/scenarios/minimal-input.json` — only `meta.initial_description` filled, everything else empty. Tests how agents handle sparse input:

```json
{
  "meta": { "initiated_at": "2026-02-28T10:00:00Z", "language": "en", "initial_description": "A food delivery app" },
  "identity": {},
  "business": {},
  "features": {},
  "regulatory": {},
  "integrations": {},
  "infrastructure": {},
  "stack": {},
  "research_refs": [],
  "pending_research": []
}
```

**Step 5: Write the artifact checklist**

Create `tests/expected/artifact-checklist.md`:

```markdown
# Expected Artifact Checklist

## After /prd-new (all scenarios)

### Mandatory Files
- [ ] `docs/prd/PRD.md` — contains sections §1-§8 with PRD version header
- [ ] `docs/prd/ARCHITECTURE.md` — contains stack table, Mermaid diagram, ADRs
- [ ] `CLAUDE.md` — root project guide referencing all other guides
- [ ] `backend/CLAUDE.md` — stack-specific dev guide with literature references
- [ ] `infrastructure/CLAUDE.md` — IaC and security baseline guide
- [ ] `.claude/skills/project-guardian/SKILL.md` — contains prd_version in frontmatter
- [ ] `.claude/skills/project-architecture/SKILL.md` — contains canonical stack table
- [ ] `.claude/skills/project-domain-rules/SKILL.md` — contains DR-XXX rules and glossary
- [ ] `.claude/skills/project-compliance/SKILL.md` — contains regulation-specific checklists

### Conditional Files
- [ ] `docs/prd/ER.md` — present when product has relational data model
- [ ] `frontend/CLAUDE.md` — present when product has a frontend

### Content Quality Checks
- [ ] All files written in English
- [ ] PRD.md §2 has RF-XXX IDs with acceptance criteria
- [ ] PRD.md §3 has RNF-XXX IDs with measurement methods
- [ ] PRD.md §4 has DR-XXX IDs with enforcement layer specified
- [ ] PRD.md §7 has only official source URLs (no blog/forum domains)
- [ ] ARCHITECTURE.md has Mermaid diagram(s) that render correctly
- [ ] All project skills have matching prd_version in frontmatter
- [ ] CLAUDE.md files reference specific books (Clean Code, Clean Architecture, DDD, etc.)

## After /prd-evolve

- [ ] PRD.md version header incremented
- [ ] §8 Evolution History has new row
- [ ] Only affected artifacts were modified (check git diff)
- [ ] All project skills prd_version matches new PRD version
- [ ] Unaffected files have zero git diff
```

**Step 6: Write tests README**

Create `tests/README.md`:

```markdown
# Testing the PRD Generator Plugin

## Test Scenarios

| File | Product Type | Complexity | Key Test Focus |
|---|---|---|---|
| `basic-saas.json` | B2B SaaS, no payments | Medium | Happy path, standard flow |
| `regulated-fintech.json` | Fintech, PCI+LGPD | High | Compliance, official sources, ER model |
| `minimal-input.json` | Food delivery (vague) | Low | Sparse input handling, question generation |

## Running Tests

### Agent Isolation Test
```bash
# Provide scenario JSON as agent prompt context
# Inspect output JSON against expected structure
```

### Full Integration Test
```bash
# Create scratch project
mkdir -p tests/scratch/test-project && cd tests/scratch/test-project && git init
# Run /prd-new with scenario as initial description
# Verify artifacts against tests/expected/artifact-checklist.md
```
```

**Step 7: Commit test infrastructure**

```bash
cd ~/Workspace/prd-generator-plugin
git add tests/
git commit -m "test: add test scenarios and artifact checklist

Three scenarios: basic-saas, regulated-fintech, minimal-input
Artifact checklist for validating /prd-new and /prd-evolve output"
```

---

## Task 2: Test product-interrogator Agent in Isolation

**Files:**
- Test input: `tests/scenarios/minimal-input.json`
- Modify (if needed): `agents/product-interrogator.md`

**Step 1: Run agent with minimal input**

Use the Agent tool to dispatch `product-interrogator` with the minimal-input scenario. The agent's input is:

```json
{
  "context_packet": <contents of tests/scenarios/minimal-input.json>,
  "completed_blocks": [],
  "initial_description": "A food delivery app"
}
```

**Step 2: Verify output structure**

Check that the agent returns valid JSON with ALL required fields:
- [ ] `coverage_scores` — all 7 blocks present with scores 0-3
- [ ] `critical_gaps` — at least 5 gaps identified (identity is partially filled, rest is empty)
- [ ] `pending_research` — at least 1 item (food delivery domain, payment processing likely)
- [ ] `inconsistencies` — may be empty (valid for sparse input)
- [ ] Each `critical_gap` has `block`, `gap`, `suggested_question`, `impact`
- [ ] Each `pending_research` item has `topic`, `reason`, `search_goals`

**Step 3: Run agent with full input**

Dispatch again with `basic-saas.json` and `completed_blocks: ["identity", "business", "features", "regulatory", "integrations", "infrastructure", "stack"]`.

Expected: coverage_scores should be mostly 2-3, fewer gaps, pending_research matching the scenario's `pending_research` field.

**Step 4: Verify consistency detection**

Check if the agent flags any inconsistencies in the regulated-fintech scenario (e.g., the `true` for `data_residency` combined with specific compliance requirements should trigger domain-specific research targets).

**Step 5: Refine prompt if issues found**

If any of the above checks fail, edit `agents/product-interrogator.md` to:
- Add missing output fields to the template
- Strengthen instructions for the failing check
- Re-run the test to verify fix

**Step 6: Commit**

```bash
cd ~/Workspace/prd-generator-plugin
git add agents/product-interrogator.md
git commit -m "refine: product-interrogator agent based on isolation tests"
```

---

## Task 3: Test official-researcher Agent in Isolation

**Files:**
- Modify (if needed): `agents/official-researcher.md`

**Step 1: Run agent with a concrete research topic**

Dispatch `official-researcher` with:

```json
{
  "topic": "LGPD compliance requirements",
  "context": "B2C app serving Brazilian users, handles personal data including health records",
  "search_goals": [
    "Data subject rights requirements",
    "DPO obligations threshold for startups",
    "Technical security measures required"
  ]
}
```

**Step 2: Verify output structure**

- [ ] `status` is "complete" or "partial" (not "not_found" — LGPD is well documented)
- [ ] Every `source.url` is from an official domain (gov.br, serpro.gov.br, etc.)
- [ ] NO source URLs contain: reddit, medium, stackoverflow, dev.to, blog, wiki
- [ ] Each `finding.summary` is < 150 words
- [ ] `key_specs` contain specific, verifiable facts (not vague statements)
- [ ] `version_notes` present if applicable

**Step 3: Run agent with a technology topic**

Dispatch with:

```json
{
  "topic": "Next.js App Router",
  "context": "Frontend for B2B SaaS, needs SSR and API routes",
  "search_goals": [
    "Current stable version",
    "App Router vs Pages Router differences",
    "Server Components API"
  ]
}
```

**Step 4: Verify blocked domains enforcement**

- [ ] All sources from nextjs.org, vercel.com, or react.dev
- [ ] No tutorial sites, no community blogs
- [ ] Version cited matches the actual current version

**Step 5: Test with obscure topic**

Dispatch with:

```json
{
  "topic": "BACEN Resolution 4658",
  "context": "Fintech operating in Brazil, needs cybersecurity compliance",
  "search_goals": ["Specific security requirements for financial institutions"]
}
```

Expected: either finds official BACEN source or returns `status: "partial"` or `"not_found"` with honest admission — NOT hallucinated data.

**Step 6: Refine prompt if issues found**

Common issues to fix:
- If blocked_domains list is incomplete: add missing domains to `agents/official-researcher.md:24-30`
- If agent cites non-official sources: strengthen the "Approved source categories" section
- If agent hallucinated content: add explicit instruction "If you cannot find the information at an official source, return status: not_found — never fabricate"

**Step 7: Commit**

```bash
git add agents/official-researcher.md
git commit -m "refine: official-researcher agent based on source validation tests"
```

---

## Task 4: Test research-validator Agent in Isolation

**Files:**
- Modify (if needed): `agents/research-validator.md`

**Step 1: Prepare test input with a known-good research ref**

Create a research_ref with a real, verifiable official source:

```json
{
  "research_refs": [
    {
      "ref_id": "ref-001",
      "topic": "PostgreSQL JSONB",
      "findings": [{
        "goal": "JSONB indexing capabilities",
        "summary": "PostgreSQL supports GIN indexes on JSONB columns for efficient querying",
        "key_specs": ["GIN index for containment operators @> and ?", "Supports jsonpath queries since v12"],
        "sources": [{
          "url": "https://www.postgresql.org/docs/16/datatype-json.html",
          "title": "JSON Types — PostgreSQL 16 Documentation",
          "domain": "postgresql.org",
          "retrieved": "2026-02-28",
          "is_official": true
        }]
      }]
    }
  ]
}
```

**Step 2: Run validator — expected: approved**

- [ ] Returns `approved: true`
- [ ] `ref_results[0].status` is "approved"
- [ ] Agent explicitly states what it checked

**Step 3: Prepare test input with a BAD research ref**

Create a ref with a non-official source and a stale version:

```json
{
  "research_refs": [
    {
      "ref_id": "ref-bad",
      "topic": "React Server Components",
      "findings": [{
        "goal": "How RSC works",
        "summary": "React Server Components allow rendering on the server",
        "key_specs": ["Available since React 17"],
        "sources": [{
          "url": "https://blog.logrocket.com/react-server-components-guide/",
          "title": "A guide to React Server Components",
          "domain": "blog.logrocket.com",
          "retrieved": "2026-02-28",
          "is_official": false
        }]
      }]
    }
  ]
}
```

**Step 4: Run validator — expected: rejected**

- [ ] Returns `approved: false`
- [ ] Identifies `non_official_source` issue for logrocket.com
- [ ] Identifies `factual_mismatch` for "React 17" (RSC is React 18+)
- [ ] Provides `requery_guidance` with corrected search approach

**Step 5: Refine prompt if issues found**

If validator approves the bad ref: strengthen the "Red flags" section in `agents/research-validator.md`. If it doesn't catch the version error: add explicit instruction to cross-check version numbers.

**Step 6: Commit**

```bash
git add agents/research-validator.md
git commit -m "refine: research-validator agent based on known-good and known-bad tests"
```

---

## Task 5: Test requirements-analyst Agent (Both Modes)

**Files:**
- Test input: `tests/scenarios/basic-saas.json`
- Modify (if needed): `agents/requirements-analyst.md`

**Step 1: Test `full` mode with basic-saas**

Dispatch `requirements-analyst` with:

```json
{
  "mode": "full",
  "context_packet": <basic-saas.json>,
  "validated_research": []
}
```

**Step 2: Verify output completeness**

- [ ] `functional_requirements` has at least 5 items matching the MVP features
- [ ] Each RF has: `id`, `title`, `phase`, `description`, `acceptance_criteria`, `entities`
- [ ] Acceptance criteria are testable (specific, measurable — not "should work well")
- [ ] `non_functional_requirements` covers: performance, security, availability
- [ ] `domain_rules` has at least 2 invariants
- [ ] `ubiquitous_language` has at least 5 terms
- [ ] `compliance_requirements` has LGPD and GDPR entries (scenario says US + Brazil)

**Step 3: Test `delta` mode**

Dispatch with:

```json
{
  "mode": "delta",
  "change_description": "Added Slack bot integration for onboarding reminders",
  "current_prd_summary": "OnboardFlow — employee onboarding SaaS, MVP features: workflow templates, task dashboard, doc upload, email/Slack notifications, approval gates",
  "current_architecture_summary": "NestJS + Next.js + PostgreSQL + AWS. Modular monolith."
}
```

**Step 4: Verify delta output**

- [ ] `delta_type` includes "new_feature" and possibly "new_integration"
- [ ] `affected_artifacts` includes PRD.md §2 and project-domain-rules
- [ ] `unaffected_artifacts` lists architecture as unaffected (Slack API already in stack)
- [ ] `pending_research` includes "Slack Bot API" (different from Slack Webhooks)
- [ ] Version bump is MINOR (1.x, not 2.0)

**Step 5: Refine and commit**

```bash
git add agents/requirements-analyst.md
git commit -m "refine: requirements-analyst agent based on full and delta mode tests"
```

---

## Task 6: Test architecture-designer Agent

**Files:**
- Test input: `tests/scenarios/regulated-fintech.json`
- Modify (if needed): `agents/architecture-designer.md`

**Step 1: Run with regulated-fintech scenario**

Dispatch `architecture-designer` with:

```json
{
  "mode": "full",
  "context_packet": <regulated-fintech.json>,
  "requirements": <output from Task 5 with fintech scenario>
}
```

If you don't have requirements output from Task 5 for fintech, run `requirements-analyst` in `full` mode with `regulated-fintech.json` first.

**Step 2: Verify stack selection quality**

- [ ] Chosen stack is modern (current stable versions, not legacy)
- [ ] Stack is appropriate for fintech: strong typing, ACID database, proper auth
- [ ] `justification` for each technology references the product's specific needs
- [ ] No incompatible version combinations
- [ ] `pattern` is appropriate for the scale and domain (likely modular monolith or event-driven)

**Step 3: Verify Mermaid diagrams**

- [ ] `system_diagram_mermaid` is syntactically valid Mermaid
- [ ] Shows all external integrations from the scenario (BACEN, Banco do Brasil, SEFAZ)
- [ ] Shows data stores
- [ ] `er_diagram_mermaid` present (payments product = relational data)
- [ ] ER shows compliance-sensitive entities marked

**Step 4: Verify ADRs**

- [ ] At least 3 ADRs present
- [ ] Each has: Status, Context, Decision, Alternatives Considered, Consequences, Review Trigger
- [ ] At least one ADR addresses the compliance-driven architecture choice

**Step 5: Verify cross-cutting concerns**

- [ ] Auth approach defined
- [ ] Error format defined (RFC 7807 or equivalent)
- [ ] Logging and tracing standard defined
- [ ] Secret management approach specified

**Step 6: Refine and commit**

```bash
git add agents/architecture-designer.md
git commit -m "refine: architecture-designer agent based on fintech scenario test"
```

---

## Task 7: Test prd-writer Agent

**Files:**
- Modify (if needed): `agents/prd-writer.md`

**Step 1: Create a scratch project**

```bash
mkdir -p ~/Workspace/prd-generator-plugin/tests/scratch/test-project
cd ~/Workspace/prd-generator-plugin/tests/scratch/test-project
git init
```

**Step 2: Run prd-writer with combined outputs from Tasks 5+6**

Dispatch `prd-writer` with:

```json
{
  "mode": "full",
  "context_packet": <basic-saas.json, enriched with requirements + architecture outputs>,
  "include_er": true,
  "prd_version": "1.0"
}
```

**Step 3: Verify generated files**

```bash
# Check files exist
ls -la tests/scratch/test-project/docs/prd/
# Expected: PRD.md, ARCHITECTURE.md, ER.md
```

**Step 4: Verify PRD.md content**

- [ ] Version header present: `Version: 1.0 | Status: Active`
- [ ] All 8 sections present (§1–§8)
- [ ] §2 has RF-XXX IDs with acceptance criteria
- [ ] §4 has DR-XXX IDs
- [ ] §7 has real URLs (not placeholders)
- [ ] §8 has initial evolution history row
- [ ] All text is in English
- [ ] Mermaid diagrams in ARCHITECTURE.md render (copy into Mermaid live editor)

**Step 5: Test update mode**

Read the generated PRD.md, then dispatch `prd-writer` with:

```json
{
  "mode": "update",
  "sections_to_update": ["§2"],
  "change_description": "Added password reset feature",
  "new_prd_version": "1.1",
  "research_refs": [],
  "current_prd_path": "tests/scratch/test-project/docs/prd/PRD.md"
}
```

- [ ] Only §2 was modified
- [ ] Version header updated to 1.1
- [ ] §8 has new evolution history row
- [ ] Other sections unchanged

**Step 6: Clean up scratch project and commit**

```bash
rm -rf ~/Workspace/prd-generator-plugin/tests/scratch/test-project
git add agents/prd-writer.md
git commit -m "refine: prd-writer agent based on generation and update tests"
```

---

## Task 8: Test stack-guide-generator and skills-generator Agents

**Files:**
- Modify (if needed): `agents/stack-guide-generator.md`, `agents/skills-generator.md`

**Step 1: Test stack-guide-generator**

Dispatch with architecture output from Task 6 (or a simplified version):

```json
{
  "architecture": {
    "stack": {
      "backend": { "technology": "NestJS", "language": "TypeScript", "version": "10.x" },
      "frontend": { "technology": "Next.js", "version": "14.x" },
      "primary_database": { "technology": "PostgreSQL", "version": "16.x" },
      "infrastructure": { "provider": "AWS", "iac": "Terraform" }
    },
    "layer_structure": {
      "backend": ["domain/", "application/", "infrastructure/", "presentation/"],
      "frontend": ["app/", "components/", "lib/", "types/"]
    }
  },
  "product_name": "OnboardFlow",
  "domain": "saas-b2b",
  "mode": "full"
}
```

**Step 2: Verify guide quality**

- [ ] `backend/CLAUDE.md` exists and references Clean Code, Clean Architecture, DDD
- [ ] `frontend/CLAUDE.md` exists and has component standards
- [ ] `infrastructure/CLAUDE.md` exists and has IaC conventions
- [ ] Root `CLAUDE.md` exists and references all layer guides
- [ ] "What NEVER to Do" section present in each guide
- [ ] All guides in English

**Step 3: Test skills-generator**

Dispatch with a combined context_packet + requirements:

```json
{
  "mode": "full",
  "context_packet": <basic-saas.json>,
  "prd_version": "1.0"
}
```

**Step 4: Verify skill quality**

- [ ] All 4 skill files created in `.claude/skills/`
- [ ] Each has `prd_version: 1.0` in frontmatter
- [ ] `project-guardian` has the feature map table with RF-IDs
- [ ] `project-domain-rules` has DR-IDs and ubiquitous language
- [ ] `project-compliance` has regulation-specific checklists (LGPD, GDPR for this scenario)
- [ ] `project-architecture` has the stack table
- [ ] No generic placeholders remaining (no `{product_name}` or `{version}` literals)

**Step 5: Refine and commit**

```bash
git add agents/stack-guide-generator.md agents/skills-generator.md
git commit -m "refine: stack-guide-generator and skills-generator based on output tests"
```

---

## Task 9: Full Integration Test — /prd-new Happy Path

**Files:**
- May modify: any agent or command file

**Step 1: Create isolated test project**

```bash
mkdir -p /tmp/prd-integration-test && cd /tmp/prd-integration-test && git init
```

**Step 2: Run /prd-new with basic-saas scenario**

From the test project directory, invoke `/prd-new A B2B SaaS platform for managing employee onboarding workflows`.

Follow the interactive interrogation — answer each block using the values from `tests/scenarios/basic-saas.json`.

**Step 3: Observe and log each phase**

For each phase, note:
- Did the interrogation ask the right questions?
- Did research agents fire for the expected technologies?
- Did the validator approve/reject correctly?
- Was the architecture proposal reasonable?
- Were all files generated?

**Step 4: Run the artifact checklist**

Go through every item in `tests/expected/artifact-checklist.md` against the generated files.

**Step 5: Log issues and fix**

For each failing checklist item:
1. Identify which agent is responsible
2. Open the agent file
3. Fix the specific issue
4. Note the fix in a test log

**Step 6: Clean up and commit**

```bash
rm -rf /tmp/prd-integration-test
cd ~/Workspace/prd-generator-plugin
git add -A
git commit -m "refine: all agents based on full integration test (basic-saas scenario)"
```

---

## Task 10: Full Integration Test — /prd-evolve

**Files:**
- May modify: `commands/prd-evolve.md`, any agent

**Step 1: Set up project with existing PRD**

```bash
mkdir -p /tmp/prd-evolve-test && cd /tmp/prd-evolve-test && git init
```

Use the artifacts generated from Task 9 (or regenerate with `/prd-new`).

**Step 2: Run /prd-evolve with a feature addition**

```
/prd-evolve "Added Slack bot for automated onboarding reminders with custom schedules"
```

**Step 3: Verify selective updates**

```bash
git diff --name-only  # Should show ONLY affected files
```

- [ ] PRD.md was modified (§2 new feature + §8 evolution history)
- [ ] project-domain-rules was updated (new domain rules for Slack bot scheduling)
- [ ] ARCHITECTURE.md was NOT modified (Slack API already in stack)
- [ ] frontend/CLAUDE.md was NOT modified
- [ ] All skill prd_version headers updated to match new PRD version

**Step 4: Run /prd-evolve with a technology change**

```
/prd-evolve "Changed database from PostgreSQL to CockroachDB for multi-region support"
```

- [ ] ARCHITECTURE.md updated with new database + ADR for the change
- [ ] project-architecture updated with new stack table
- [ ] backend/CLAUDE.md updated with CockroachDB-specific practices
- [ ] PRD.md §8 has new evolution entry

**Step 5: Refine and commit**

```bash
rm -rf /tmp/prd-evolve-test
cd ~/Workspace/prd-generator-plugin
git add -A
git commit -m "refine: prd-evolve flow based on integration tests"
```

---

## Task 11: Edge Case Hardening

**Files:**
- Modify: `commands/prd-new.md`
- Modify: `commands/prd-evolve.md`
- Modify: `agents/official-researcher.md`
- Modify: `agents/research-validator.md`

**Step 1: Add "not in a git repo" handling to prd-new.md**

In `commands/prd-new.md`, at the start of Phase 5 (Document Generation), add:

```markdown
## Pre-flight Check

Before generating files, verify:
1. Current directory is a git repository (`git rev-parse --is-inside-work-tree`)
2. If NOT a git repo, ask user: "The current directory is not a git repository. Should I initialize one here, or do you want to navigate to a different directory?"
3. There are no uncommitted changes that might conflict (`git status --porcelain`)
```

**Step 2: Add "user cancels mid-interrogation" handling**

In `commands/prd-new.md`, after each interrogation block, add:

```markdown
After each block, if the user responds with "skip" or "that's enough":
- Stop interrogation early
- Mark remaining blocks as `coverage_score: 0`
- Proceed with what we have, noting gaps in the PRD
- Add a "Known Gaps" section to PRD.md §2.4 listing unfilled blocks
```

**Step 3: Add "all research fails" handling to prd-new.md**

In Phase 2, add:

```markdown
If ALL research refs return `partially_validated` or worse after 3 iterations:
- Do NOT block PRD generation
- Generate PRD with a prominent WARNING section in §7:
  "⚠️ Research Validation Warning: The following topics could not be fully validated from official sources.
  Manual verification is recommended before implementing features dependent on these topics."
- List each unvalidated topic with the specific issues
```

**Step 4: Add "prd-evolve without existing PRD" handling**

In `commands/prd-evolve.md`, at Phase 0:

```markdown
If `docs/prd/PRD.md` does not exist:
- Error: "No PRD found at docs/prd/PRD.md. Run /prd-new first to generate the initial PRD."
- Exit without modifying any files.
```

**Step 5: Add rate limit / timeout handling for researcher**

In `agents/official-researcher.md`, add after the output format section:

```markdown
## Error Handling

If WebSearch returns no results for a query:
- Try an alternative query formulation (remove site: restriction, broaden terms)
- If still no results: mark the goal as `"not_found"` with note "No official source found for this query"

If WebFetch returns a timeout or 403:
- Log the URL as inaccessible
- Try the next source from WebSearch results
- If all sources fail: mark as `"partial"` with note "Official source exists but was inaccessible"
```

**Step 6: Commit all edge case fixes**

```bash
cd ~/Workspace/prd-generator-plugin
git add commands/ agents/
git commit -m "fix: add edge case handling for no-git, early-cancel, research-failures, missing-PRD"
```

---

## Task 12: Token Efficiency Audit

**Files:**
- Modify: `agents/product-interrogator.md`
- Modify: `agents/requirements-analyst.md`
- Modify: `agents/prd-writer.md`
- Modify: `agents/skills-generator.md`

**Step 1: Measure current agent prompt sizes**

```bash
cd ~/Workspace/prd-generator-plugin
wc -w agents/*.md commands/*.md skills/prd-guardian/SKILL.md
```

**Step 2: Identify the largest agents**

Target agents > 800 words for trimming. Focus on:
- Removing redundant instructions (said once is enough)
- Converting prose to tables
- Removing verbose examples where a concise one suffices
- Moving reference-heavy content to separate files if > 200 lines

**Step 3: Verify no information loss**

For each trimmed agent:
- Re-run the relevant isolation test from Tasks 2-8
- Confirm output quality is unchanged
- If quality drops, restore the cut content

**Step 4: Audit context_packet handoff sizes**

Review each agent's input specification. Ensure:
- No agent receives the full context_packet when it only needs a slice
- `prd-writer` gets the full packet (it needs everything)
- `stack-guide-generator` only needs `architecture` + `product_name` + `domain`
- `official-researcher` only needs `topic` + `context` + `search_goals`
- `research-validator` only needs `research_refs` (NO context_packet)

**Step 5: Commit**

```bash
git add agents/ commands/
git commit -m "perf: reduce token consumption across agent prompts without information loss"
```

---

## Task 13: Marketplace Preparation

**Files:**
- Create: `LICENSE`
- Create: `CHANGELOG.md`
- Modify: `.claude-plugin/plugin.json`
- Modify: `README.md`

**Step 1: Add LICENSE file**

Create `LICENSE` with MIT license text:

```
MIT License

Copyright (c) 2026 rodrigo

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

**Step 2: Create CHANGELOG.md**

```markdown
# Changelog

All notable changes to prd-generator-plugin will be documented in this file.

## [1.0.0] - 2026-02-28

### Added
- `/prd-new` command — full interactive PRD generation workflow
- `/prd-evolve` command — incremental PRD evolution with selective artifact updates
- 8 specialized sub-agents: product-interrogator, official-researcher, research-validator, requirements-analyst, architecture-designer, prd-writer, stack-guide-generator, skills-generator
- `prd-guardian` global skill for PRD compliance enforcement
- Official-sources-only research with blocked_domains enforcement
- Fresh-context research validation loop (max 3 iterations)
- Modern stack selection optimized for AI-assisted development
- Stack-specific CLAUDE.md guides based on Clean Code, Clean Architecture, DDD, The Pragmatic Programmer
- 4 self-evolving project enforcement skills: project-guardian, project-architecture, project-domain-rules, project-compliance
- Stale skills version detection
- Token-efficient JSON context_packet handoff pattern
```

**Step 3: Update plugin.json with repository and homepage**

Edit `.claude-plugin/plugin.json` to add:

```json
{
  "name": "prd-generator",
  "version": "1.0.0",
  "description": "AI-powered PRD generator with official-source research validation and self-evolving project skills for AI-assisted development",
  "author": {
    "name": "rodrigo"
  },
  "homepage": "https://github.com/<username>/prd-generator-plugin",
  "repository": "https://github.com/<username>/prd-generator-plugin",
  "license": "MIT",
  "keywords": [
    "prd",
    "product-requirements",
    "architecture",
    "documentation",
    "ai-development",
    "clean-architecture",
    "ddd",
    "research-validation"
  ]
}
```

**Step 4: Final README review**

Read `README.md` end-to-end. Verify:
- [ ] Installation instructions are accurate
- [ ] All Mermaid diagrams render correctly (spot-check 2-3 in Mermaid live editor)
- [ ] Component reference table matches actual file names
- [ ] No TODO or placeholder text remaining
- [ ] All blocked_domains listed match `agents/official-researcher.md`

**Step 5: Version bump and final commit**

```bash
cd ~/Workspace/prd-generator-plugin
git add LICENSE CHANGELOG.md .claude-plugin/plugin.json README.md
git commit -m "chore: prepare v1.0.0 for marketplace — license, changelog, metadata"
```

**Step 6: Tag release**

```bash
git tag -a v1.0.0 -m "v1.0.0 — initial release"
```

---

## Task 14: Publish to GitHub

**Step 1: Create remote repository**

```bash
cd ~/Workspace/prd-generator-plugin
gh repo create prd-generator-plugin --public --source=. --push
```

**Step 2: Push tags**

```bash
git push origin --tags
```

**Step 3: Create GitHub release**

```bash
gh release create v1.0.0 --title "v1.0.0 — Initial Release" --notes "$(cat CHANGELOG.md | tail -n +5)"
```

**Step 4: Verify**

```bash
gh repo view --web
```

Open the repo URL and verify:
- [ ] README renders with Mermaid diagrams
- [ ] All files present
- [ ] Release tag visible
- [ ] License shown in sidebar

---

## Execution Order and Dependencies

```
Task 1 (test infra) ──────────────────────────────────┐
                                                        │
Task 2 (interrogator) ──┐                              │
Task 3 (researcher)  ───┤── can run in parallel ───────│
Task 4 (validator)   ───┘                              │
                                                        │
Task 5 (requirements) ── depends on 2 ─────────────────│
                                                        │
Task 6 (architecture) ── depends on 5 ─────────────────│
                                                        │
Task 7 (prd-writer)  ───┐                              │
Task 8 (guides+skills)──┤── depends on 5+6, parallel ──│
                         │                              │
Task 9 (integration /prd-new) ── depends on 1-8 ──────│
Task 10 (integration /prd-evolve) ── depends on 9 ────│
                                                        │
Task 11 (edge cases)  ── depends on 9+10 ──────────────│
Task 12 (token audit) ── depends on 9+10 ──────────────│
                                                        │
Task 13 (marketplace) ── depends on 11+12 ─────────────│
Task 14 (publish)     ── depends on 13 ────────────────┘
```

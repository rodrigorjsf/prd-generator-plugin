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
- Edge case handling: no-git-repo, early interrogation cancel, research failure fallback, missing PRD guard

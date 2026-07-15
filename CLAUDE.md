# Hostaway Design System

This repo is a **design system audit and roadmap workspace** — no application code. It contains audit documents, component specs, and tooling for auditing Figma components against atomic design principles.

## What's here

| Path | Purpose |
|---|---|
| `deliver/` | Deliverables: audit summary, initial roadmap |
| `deliver/img/` | Images referenced by audit documents |
| `deliver/audit-untitled-ui.md` | Root-cause analysis of the Untitled UI lift |
| `deliver/migration-roadmap.md` | 3–6 month transition plan (July–December 2026) |
| `audits/` | Component deep-dive audit files |
| `audits/untitled-ui-tabs-audit.md` | Deep-dive spec for Tabs component rebuild |
| `audits/page-mock-audit.md` | Reference implementation audit (A− pilot — Page header + Table + Filters) |
| `audit-automation/` | Shared reference material for the audit skill |
| `audit-automation/Figma Atomic Audit — Reference.md` | Grading rubric, output template, MCP scripts — used by both Cursor and Claude skills |
| `.cursor/skills/figma-atomic-audit/` | Cursor skill for auditing Figma components |
| `.claude/commands/figma-atomic-audit.md` | Claude skill for auditing Figma components |

## Context

The project is transitioning Hostaway's product UI from a **lifted Untitled UI kit** to an **owned, token-driven design system**. The core problem: Untitled UI application-tier components (page headers, sidebars, tables) were published wholesale as shared Figma components, creating monoliths that can't compose, scale, or hand off to engineering cleanly.

Target architecture:
```
Tokens → Components (≤30) → Patterns (≤15) → Templates (product files only) → Reference (Untitled UI)
```

## Running audits

Use `/figma-atomic-audit` when auditing a Figma component. It requires the Figma Desktop Bridge MCP (`figma-console` server) to be connected.

Audit output is written to `audits/{component-slug}-audit.md`. Cross-link new deep-dives from `deliver/audit-untitled-ui.md`.

## Conventions

- Audit files go in `audits/` (e.g. `audits/filter-group-audit.md`)
- Deliverables (stakeholder-facing docs, images) go in `deliver/`
- Shared skill reference material goes in `audit-automation/`
- Ignore `.discarded/`
- Figma node IDs use colon format (`36:3634`); URLs use dash format (`36-3634`) — convert when calling MCP tools

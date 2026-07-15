# Hostaway Design System

This repo is a **design system audit and roadmap workspace** — no application code. It contains audit documents, component specs, and a custom Cursor skill for auditing Figma components against atomic design principles.

## What's here

| Path | Purpose |
|---|---|
| `deliver/` | Deliverables: audit summary, initial roadmap, token map images |
| `deliver/audit-summary.md` | Root-cause analysis of the Untitled UI lift |
| `deliver/initial-roadmap.md` | 3–6 month transition plan (July–December 2026) |
| `audits/tabs-audit.md` | Deep-dive spec for Tabs component rebuild |
| `audits/page-mock-audit.md` | Reference implementation audit (A− pilot — Page header + Table + Filters) |
| `.cursor/skills/figma-atomic-audit/` | Reusable audit skill for Figma components |

## Context

The project is transitioning Hostaway's product UI from a **lifted Untitled UI kit** to an **owned, token-driven design system**. The core problem: Untitled UI application-tier components (page headers, sidebars, tables) were published wholesale as shared Figma components, creating monoliths that can't compose, scale, or hand off to engineering cleanly.

Target architecture:
```
Tokens → Components (≤30) → Patterns (≤15) → Templates (product files only) → Reference (Untitled UI)
```

## Running audits

Use `/figma-atomic-audit` (`.claude/commands/figma-atomic-audit.md`) when auditing a Figma component. It requires the Figma Desktop Bridge MCP (`figma-console` server) to be connected.

Audit output is written as markdown in the repo root. Cross-link new deep-dives from `deliver/audit-summary.md`.

## Conventions

- Audit files live at repo root (e.g., `tabs-audit.md`, `page-mock-audit.md`)
- Deliverables (stakeholder-facing docs, images) go in `deliver/`
- No `.discarded/` folder — ignore it
- Figma node IDs use colon format (`36:3634`); URLs use dash format (`36-3634`) — convert when calling MCP tools

# Hostaway — Design System Take-Home

A design system audit, language definition, and migration roadmap for Hostaway's product UI. The work spans a Figma component library, full token documentation, component audits, and a 6-month transition plan for a team moving from a lifted third-party kit to an owned system.

**Figma file:** [Hostaway — Components page](https://www.figma.com/design/pAucTdmNvixw2TnIoLT2fe/Hostaway?node-id=10-2896)

---

## What was the problem

Hostaway's design system was built by lifting application-tier components from the Untitled UI kit — page headers, sidebars, tables — and publishing them wholesale as shared Figma components. The result: monoliths with hundreds of variants encoding content permutations rather than structural states, no meaningful token bindings, no composition model, and nothing an engineer could implement without recreating from scratch.

The tabs component alone had 280 variants. The equivalent React implementation had 5 props.

---

## What I did

### 1. Audited the problem space

Audits live in `audits/`. Each one inspects a Figma component via the Desktop Bridge MCP, maps its atomic tier, grades it against atomic design principles, and produces a specific verdict and fix list.

| Audit | Component | Grade | Verdict |
|---|---|---|---|
| [untitled-ui-tabs-audit.md](audits/untitled-ui-tabs-audit.md) | Untitled UI Horizontal Tabs | — | Don't lift. Rebuild with ≤16 variant budget. |
| [filter-group-audit.md](audits/filter-group-audit.md) | Hostaway Filter group | A− | Correct slot architecture. Fix minChildren constraint + baked Reset button. |
| [table-audit.md](audits/table-audit.md) | Hostaway Table | A− | Column-slot model is right. Add state BOOLEANs, reduce default column count. |
| [tabs-group-audit.md](audits/tabs-group-audit.md) | Hostaway Tabs group | A− | Cleanest component audited. Fix minChildren constraint, reduce 5-tab default, bind spacing tokens. |

The tabs audit was the diagnostic case: it shows exactly why the Untitled UI lift failed (Figma is over-varianted; code is already composable) and what the rebuild target should look like.

### 2. Defined the design language

[deliver/design-language.md](deliver/design-language.md) is the canonical specification for the system as it stands today — not aspirational, but a precise record of what exists in the Figma file.

It covers:
- **Token system** — 16 palette primitives, 20 semantic aliases across background, text, border, and icon categories, with full hex values and usage rules. A known typo in the Figma variable names (`backgorund`) is flagged with a fix instruction.
- **Component inventory** — 8 component sets and 10 standalone components, each with variants, props, slot contracts, and layout specs.
- **Assembly patterns** — how to compose a standard list page from `Template / Page table` down through slots.
- **Known gaps** — spacing tokens, text styles, dark mode, error states, focus/disabled states. Documented as TBD so nothing gets invented to fill them.

### 3. Wrote audience-specific usage docs

[deliver/audiences-usage.md](deliver/audiences-usage.md) takes the design language and explains how three different audiences interact with it:

- **Designers** — mental model (tokens → components → templates), step-by-step for starting a new page, slot population rules, what's off-limits and why.
- **Engineers** — token-to-CSS mapping (full `:root {}` block), component-to-code prop contracts for Button, Tabs, Input, Table, and Page header, slot-to-children/render-prop translation, two paths for design-to-code linking (Figma Code Connect vs custom mapping layer).
- **AI tooling** — component lookup protocol, token resolution protocol, slot population rules, violation detection signals, and prompt patterns that stay in-system. This is what makes the system usable with AI-assisted code generation without hallucinated values.

### 4. Planned the migration

[deliver/migration-roadmap.md](deliver/migration-roadmap.md) is a 6-month plan (July–December 2026) for a team of 1 DS lead and 6 part-time contributing designers.

```
Phase 0  │ Weeks 1–2   │ Inventory before touching anything
Phase 1  │ Weeks 3–10  │ Infrastructure: library split, token pipeline, contribution model, automated audit gate
Phase 2  │ Weeks 7–24  │ Rebuild in atomic order (tokens → atoms → molecules → organisms → templates), then migrate in waves
```

The plan includes a two-track contribution model (DS-led vs squad-contributed), weekly automated audit reporting, migration waves, adoption mechanics for both designers and engineers, and an honest risk register — the highest risks being feature pressure and engineering migration lag, not the design work itself.

---

## How the pieces connect

```
audits/               ← evidence base: what's wrong and why, component by component
deliver/design-language.md   ← what exists now: tokens, components, specs, gaps
deliver/audiences-usage.md   ← how to use it: designers, engineers, AI tooling
deliver/migration-roadmap.md ← how to get from here to there: phases, waves, metrics
```

The audit findings drive the design language decisions (e.g., the tabs audit is why `_Tab.Button` is a private sub-component with a slot, not a 280-variant monolith). The design language is the contract the roadmap is rebuilding toward. The audience docs are what makes that contract usable without someone standing over your shoulder.

---

## Tooling

The repo includes a `figma-atomic-audit` skill ([.claude/commands/figma-atomic-audit.md](.claude/commands/figma-atomic-audit.md)) that runs live against the Figma Desktop Bridge. It grades components against the same rubric used in the audits and is the gate proposed for all library contributions in Phase 1.

Audit output goes to `audits/{component-slug}-audit.md`.

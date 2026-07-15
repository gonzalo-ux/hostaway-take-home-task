---
name: figma-atomic-audit
description: >-
  Audits Figma components and templates against atomic design system principles
  (atoms, molecules, organisms, templates, pages). Uses Figma Desktop Bridge
  MCP for live layer trees, slot contracts, and property bindings. Use when the
  user asks to audit a Figma component, design system architecture, slot
  contracts, atomic design quality, or Hostaway design system health checks.
---

# Figma Atomic Design Audit

Audit Figma nodes for compositional depth, slot contracts, variant governance, and engineering handoff quality. Write results as markdown in this repo.

## Before you start

1. Read parent context if present: [audit-untitled-ui.md](../../deliver/audit-untitled-ui.md)
2. Check for existing deep-dives: [untitled-ui-tabs-audit.md](../../audits/untitled-ui-tabs-audit.md), [page-mock-audit.md](../../audits/page-mock-audit.md)
3. Confirm Figma Desktop Bridge is paired (`user-figma-console`). If auth fails, ask user to pair via `figma_pair_plugin` — do not guess from screenshots alone.

## Workflow

Copy this checklist and track progress:

```
Audit Progress:
- [ ] Step 1: Identify node tier (atom / molecule / organism / template / page)
- [ ] Step 2: Fetch live structure via Bridge
- [ ] Step 3: Map composition tree and published dependencies
- [ ] Step 4: Audit properties, slots, and bindings
- [ ] Step 5: Check anti-patterns
- [ ] Step 6: Screenshot for visual verification
- [ ] Step 7: Score and write audit markdown
- [ ] Step 8: Update cross-links in audit-untitled-ui.md if new deep-dive
```

### Step 1: Identify atomic tier

| Figma signal | Tier |
|---|---|
| `_Prefix.Name` primitives, icon vectors, text styles | Atom |
| Composes atoms (label + field, tab item, breadcrumb item) | Molecule |
| Named shell with slots/booleans (Page header, Filter group, Table) | Organism |
| Stacks organisms; may be published or demo-only | Template |
| Unpublished frame assembling templates | Page |

**Clarify upfront:** A URL may point to a Page or Template, not a single organism. Audit the correct tier — don't score a page mock as if it were a button.

### Step 2: Fetch live data (required)

Run in parallel when possible:

1. `figma_get_component_for_development_deep` — `nodeId` from URL (`36-3634` → `36:3634`)
2. `figma_execute` — property definitions, slot layers, bindings (see [Figma Atomic Audit — Reference](../../audit-automation/Figma%20Atomic%20Audit%20—%20Reference.md))
3. `figma_analyze_component_set` — only if node is a `COMPONENT_SET`
4. `figma_capture_screenshot` — visual verification

If `analyze_component_set` fails with "not a COMPONENT_SET", the node is a frame, component, or instance — continue with deep fetch.

### Step 3: Map composition tree

Document:

- Top-level children (instances vs raw frames)
- Instance `mainComponent` names — trace to published library parts
- Nesting depth: atom → molecule → organism → template
- Components page inventory if user provides context page URL

### Step 4: Audit properties and slots

For each component property:

| Check | Pass | Fail |
|---|---|---|
| SLOT property has bound slot layer in tree | Property `slotContentId` refs match a SLOT node | Orphan property (panel shows slot, layers panel doesn't) |
| BOOLEAN controls visibility/structure | Bound via `componentPropertyReferences` | Duplicate boolean at parent and child |
| Variants encode structure | `Type=Primary`, `Size=sm` | Content baked in ("Long title", sample data rows) |
| Slot preferred values | Documented allowed children | Empty or unrestricted when restriction intended |
| `minChildren` on slots | Justified by layout | Forces filler content on sparse screens |

### Step 5: Anti-patterns (flag any)

From [audit-untitled-ui.md](../../deliver/audit-untitled-ui.md):

- Monolithic block with content variants instead of slots
- Detached frames where instances should be used
- Generic names (`Frame 26`, `Frame 39`) on published components
- Mixed button/icon sources (kit + custom)
- Template published with sample copy as variants
- Variant count encoding layout permutations (tabs: 280, tables: 204 pattern)
- Properties panel API that doesn't match layer tree

### Step 6: Score

Use rubric in [Figma Atomic Audit — Reference](../../audit-automation/Figma%20Atomic%20Audit%20—%20Reference.md). Grade A–F per criterion, then overall.

Typical ranges for Hostaway rebuild:

| Grade | Meaning |
|---|---|
| A / A− | Slot-driven, published primitives, honest API |
| B+ | Good composition, minor naming or API gaps |
| B / B− | Usable but monolithic regions or orphan props |
| C+ or below | Lifted kit pattern — detach/fork workflow |

### Step 7: Write output

Save as `{component-slug}-audit.md` in repo root (e.g. `filter-group-audit.md`).

Use the template in [Figma Atomic Audit — Reference](../../audit-automation/Figma%20Atomic%20Audit%20—%20Reference.md). Required sections:

1. Header metadata (Figma links, parent audit links, methodology)
2. Executive summary with verdict table
3. Atomic tier map (ASCII tree)
4. What is built well
5. Issues (severity: High / Medium / Low)
6. Recommendations (Done / Next)
7. Scorecard

**Re-audit:** If user says they cleaned up, re-fetch live data. Add "Post-cleanup state" section with before/after table. Update grades.

### Step 8: Cross-links

When creating a new deep-dive, add to [audit-untitled-ui.md](../../deliver/audit-untitled-ui.md) header:

```markdown
**Example deep-dives:** [untitled-ui-tabs-audit.md](../audits/untitled-ui-tabs-audit.md) · [page-mock-audit.md](../audits/page-mock-audit.md) · [{new}.md](../audits/{new}.md)
```

Link related specs (e.g. table audits → other audit files in `audits/`).

## Key MCP snippets

Full scripts in [Figma Atomic Audit — Reference](../../audit-automation/Figma%20Atomic%20Audit%20—%20Reference.md). Minimal property audit:

```javascript
const node = await figma.getNodeByIdAsync('NODE_ID');
const defs = node.componentPropertyDefinitions || {};
// Compare defs keys to SLOT layers with componentPropertyReferences
```

## Do not

- Cite layer names from browser screenshots when Bridge is available
- Treat instance-swap-heavy variant matrices as good atomic design
- Score aesthetics — audit **architecture and contracts** only
- Create empty properties "for later" — they become orphan API surface

## Additional resources

- Grading rubric, output template, MCP scripts: [Figma Atomic Audit — Reference](../../audit-automation/Figma%20Atomic%20Audit%20—%20Reference.md)
- Example audits: [untitled-ui-tabs-audit.md](../../audits/untitled-ui-tabs-audit.md), [page-mock-audit.md](../../audits/page-mock-audit.md)

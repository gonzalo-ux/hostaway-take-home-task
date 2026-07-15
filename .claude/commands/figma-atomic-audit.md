Audit a Figma component or template against atomic design system principles for the Hostaway design system. Write results as a markdown file in the repo root.

**Reference material:** grading rubric, output template, MCP scripts — [audit-automation/Figma Atomic Audit — Reference.md](../audit-automation/Figma%20Atomic%20Audit%20—%20Reference.md)

## Before you start

1. Read context files if present: `deliver/audit-untitled-ui.md`, `deliver/migration-roadmap.md`
2. Check existing deep-dives: `audits/untitled-ui-tabs-audit.md`, `audits/page-mock-audit.md`
3. Confirm Figma Desktop Bridge is connected (`figma_get_status`). If not, ask the user to open the Desktop Bridge plugin before continuing — do not infer from screenshots alone.

The user will provide a Figma URL or node ID. Convert URL format to MCP format: `node-id=36-3634` → `36:3634`.

## Audit checklist

```
- [ ] Step 1: Identify atomic tier
- [ ] Step 2: Fetch live structure via Bridge
- [ ] Step 3: Map composition tree
- [ ] Step 4: Audit properties, slots, and bindings
- [ ] Step 5: Check anti-patterns
- [ ] Step 6: Screenshot for visual verification
- [ ] Step 7: Score (rubric in reference.md)
- [ ] Step 8: Write audit markdown (template in reference.md)
- [ ] Step 9: Cross-link from deliver/audit-untitled-ui.md
```

---

## Step 1 — Identify atomic tier

| Figma signal | Tier |
|---|---|
| `_Prefix.Name` primitives, icon vectors, text styles | Atom |
| Composes atoms (label + field, tab item, breadcrumb item) | Molecule |
| Named shell with slots/booleans (Page header, Filter group, Table) | Organism |
| Stacks organisms; may be published or demo-only | Template |
| Unpublished frame assembling templates | Page |

Clarify upfront — a URL may point to a Page or Template, not a single organism. Audit the correct tier.

---

## Step 2 — Fetch live data (required)

Run in parallel:

1. `figma_get_component_for_development_deep` — deep tree, boundVariables, reactions
2. `figma_execute` — property definitions, slot layers, bindings (use **Full structure + properties** script from reference.md)
3. `figma_analyze_component_set` — only if node is a `COMPONENT_SET`
4. `figma_capture_screenshot` — visual reference

If `figma_analyze_component_set` fails with "not a COMPONENT_SET", continue with the deep fetch.

---

## Step 3 — Map composition tree

Document:

- Top-level children (instances vs raw frames)
- Instance `mainComponent` names — trace to published library parts
- Nesting depth: atom → molecule → organism → template
- Components page inventory if a context page URL is provided (use **Components page inventory** script from reference.md)

---

## Step 4 — Audit properties and slots

| Check | Pass | Fail |
|---|---|---|
| SLOT property has bound slot layer in tree | `slotContentId` refs match a SLOT node | Orphan property |
| BOOLEAN controls visibility/structure | Bound via `componentPropertyReferences` | Duplicate boolean at parent and child |
| Variants encode structure, not content | `Type=Primary`, `Size=sm` | Sample copy, data rows baked into variant |
| Slot preferred values documented | Allowed children listed | Empty or unrestricted |
| `minChildren` on slots justified | Required by layout constraint | Forces filler on sparse screens |

Use the **Orphan property detection** script from reference.md to surface orphans programmatically.

---

## Step 5 — Anti-patterns (flag any present)

From `deliver/audit-untitled-ui.md`:

- Monolithic block with content variants instead of slots
- Detached frames where instances should be used
- Generic layer names (`Frame 26`, `Frame 39`) on published components
- Mixed button/icon sources (kit + custom)
- Template published with sample copy as variants
- Variant count encoding layout permutations (e.g. 280 tab variants, 204 table variants)
- Properties panel API does not match layer tree

---

## Step 6 — Screenshot

Call `figma_capture_screenshot` with the node ID. Describe composition, obvious detached regions, naming issues.

---

## Step 7 — Score

Use the grading rubric in reference.md. Grade each criterion A–F, then derive an overall grade.

---

## Step 8 — Write output

Derive the filename from the **actual Figma component name** returned by the Bridge (the `name` field from `figma_get_component_for_development_deep` or `figma_execute`):

1. Take the component name exactly as Figma reports it (e.g. `"Filter group"`, `"Page header"`, `"Tabs"`)
2. Lowercase and hyphenate: `"Filter group"` → `filter-group`
3. Append `-audit.md`: `filter-group-audit.md`
4. Write to **`audits/{filename}`** (e.g. `audits/filter-group-audit.md`)

Use the output template from reference.md. Do not guess a name from the URL or the user's prompt — use the name the Bridge returns.

---

## Step 9 — Cross-link

Add the new audit to the deep-dives list in `deliver/audit-untitled-ui.md`:

```markdown
**Example deep-dives:** [untitled-ui-tabs-audit.md](../audits/untitled-ui-tabs-audit.md) · [page-mock-audit.md](../audits/page-mock-audit.md) · [{component-slug}-audit.md](../audits/{component-slug}-audit.md)
```

---

## Do not

- Cite layer names from browser screenshots when Bridge is available
- Treat instance-swap-heavy variant matrices as good atomic design
- Score aesthetics — audit architecture and contracts only
- Create empty properties "for later" — they become orphan API surface
- Re-use a nodeId from a previous session without re-fetching (they are session-specific)

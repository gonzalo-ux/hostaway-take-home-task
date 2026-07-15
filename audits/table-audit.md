# Component Audit — Table (Hostaway)

**Subject:** [Table](https://www.figma.com/design/pAucTdmNvixw2TnIoLT2fe/Hostaway?node-id=50-6292)  
**Audit lens:** Atomic design system architecture  
**Date:** July 2026  
**Parent audit:** [audit-untitled-ui.md](../deliver/audit-untitled-ui.md)  
**Related:** [filter-group-audit.md](./filter-group-audit.md) · [page-mock-audit.md](./page-mock-audit.md) · [untitled-ui-tabs-audit.md](./untitled-ui-tabs-audit.md)  
**Methodology:** Live Figma Desktop Bridge — layer tree, properties, bindings, screenshots.  
**Re-audit:** July 2026 — `allowPreferredValuesOnly` fix verified.

---

## Executive summary

`Table` (`50:6292`) is a single COMPONENT organism implementing a **column-first data table** architecture. Its outer shell exposes one `Table slot` that contains 11 default `Table.Column` instances. Each column composes a `_Table.Header.Cell` atom at top plus a `Cells Slot` accepting `_Table.Cell` instances with `Odd`/`Even` variant striping. Token binding is complete throughout — no hardcoded colour values.

Since the initial audit, the outer `Table slot` has been tightened: `allowPreferredValuesOnly` is now `true` and a preferred value (COMPONENT key `442b1c3d...`) has been added. The previous **High** issue is resolved. Remaining gaps are the baked 11-column default content, absent state BOOLEANs, no selection-column pattern, and hardcoded cell padding.

| Surface | Node | Tier | Assessment |
|---|---|---|---|
| Table | `50:6292` | Organism | B+ → A− — outer slot now constrained; state coverage still absent |

**Verdict:** Compose, don't lift. Column-slot model is correct; the slot contract is now enforced. Add state BOOLEANs and reduce the default column count before promoting as a reference pattern.

**Overall atomic design grade: A−** (upgraded from B+ — high issue resolved)

---

## 1. Atomic tier map

```
Table  [COMPONENT — Organism]
│  stroke → color/border/tertiary
└── Table slot  [SLOT — bound to "Table slot#50:43"]
    │  layoutMode: HORIZONTAL, layoutSizing: FILL×HUG
    │  allowPreferredValuesOnly: true ✅  preferredValues: [COMPONENT 442b1c3d...]
    ├── Table.Column  [INSTANCE ×11]
    │   │  fill → color/background/default
    │   │  layoutMode: VERTICAL, layoutSizing: FILL×HUG
    │   ├── _Table.Header.Cell  [INSTANCE — Atom]
    │   │   │  fill → color/background/light
    │   │   │  padding: 16×12px (hardcoded ⚠)
    │   │   │  height: 52px
    │   │   ├── "Header label"  [TEXT — Open Sans Bold 14]
    │   │   │     fill → color/text/primary
    │   │   └── sort-vertical-01  [INSTANCE — icon atom]
    │   └── Cells Slot  [SLOT — bound to "Cells Slot#53:49"]
    │       │  preferredValues: [COMPONENT_SET 55ea80...]
    │       └── _Table.Cell  [INSTANCE ×6 — Odd/Even variants]
    │             fill → color/background/default
    │             padding: 16×12px (hardcoded ⚠)
    │             height: 52px
    │             └── "Cell label"  [TEXT — Open Sans Regular 14]
    │                   fill → color/text/primary
    │
    └── (×10 more Table.Column instances — identical structure)
```

**Nesting depth:** 4 levels (organism → slot → column instance → atom). Within atomic budget.

**Font note:** `Open Sans` throughout (header Bold 14, cell Regular 14) — consistent with all other components in the system.

---

## 2. Property and slot contract

### Table-level

| Property | Type | Default | Bound | Notes |
|---|---|---|---|---|
| `Table slot` | SLOT | — | ✅ `50:6291` | `allowPreferredValuesOnly: true` ✅, 1 preferred value ✅ |

### Table.Column-level (per instance inside slot)

| Property | Type | Default | Bound | Notes |
|---|---|---|---|---|
| `Cells Slot` | SLOT | — | ✅ per column | `preferredValues`: COMPONENT_SET `55ea80...` |

### `_Table.Header.Cell` properties

| Property | Type | Default | Notes |
|---|---|---|---|
| `Header cell` | TEXT | `"Header label"` | Header label text |
| `Sort` | BOOLEAN | `true` | Toggles sort icon visibility |

### `_Table.Cell` properties

| Property | Type | Default | Notes |
|---|---|---|---|
| `Cell label` | TEXT | `"Cell label"` | Cell text content |
| `Type` | VARIANT | `Odd` | `Odd` / `Even` — zebra-stripe fill |
| `Interactive` | VARIANT | `False` | `True` / `False` — interactive row state |

**Orphan scan:** 0 orphans at Table level. `Table slot` SLOT layer correctly references `slotContentId: "Table slot#50:43"`.

---

## 3. What is built well

### 3.1 Column-first slot architecture

The table is composed as a sequence of `Table.Column` instances inside a slot, not a monolithic grid frame. Columns can be added, removed, or reordered within the slot without detaching. Each column owns its header atom and a `Cells Slot` — a clean two-level composition that maps directly to a `<Column>` / `col` prop API in code.

### 3.2 Full token binding — zero hardcoded colours

| Layer | Property | Token |
|---|---|---|
| Table | stroke | `color/border/tertiary` |
| Table.Column | fill | `color/background/default` |
| `_Table.Header.Cell` | fill | `color/background/light` |
| Header text | fill | `color/text/primary` |
| `_Table.Cell` | fill | `color/background/default` |
| Cell text | fill | `color/text/primary` |

No raw hex values anywhere in the colour bindings.

### 3.3 `_Prefix.Name` atom naming

`_Table.Header.Cell` and `_Table.Cell` follow the `_Prefix.Name` convention. `Table.Column` is correctly named without the underscore prefix (it is a molecule, not an atom). `sort-vertical-01` is a semantic icon name.

### 3.4 Cell striping via variant, not baked fills

Even/odd colouring is handled through `Type=Odd/Even` on `_Table.Cell`, keeping colour logic in one place (the cell component set) and making it overridable per instance.

### 3.5 `Sort` BOOLEAN on the header atom

`_Table.Header.Cell.Sort` is a proper structural BOOLEAN — it shows or hides the sort icon without creating a separate variant for sortable vs non-sortable headers.

### 3.6 Outer slot now constrained ✅ (fixed since initial audit)

`Table slot` has `allowPreferredValuesOnly: true` and one preferred value. Only the designated `Table.Column` component can be inserted. The column-first architecture is now enforced at the API layer, not just by convention.

---

## 4. Post-cleanup state

| Issue (initial audit) | Severity | Status |
|---|---|---|
| `Table slot` unconstrained (`allowPreferredValuesOnly: false`, no preferred values) | High | **FIXED** ✅ |
| Default content encodes fixed 11-column layout | Medium | Open |
| No state coverage (empty, loading, error) | Medium | Open |
| No row-selection / checkbox column pattern | Low | Open |
| `Cells Slot` preferred values undocumented | Low | Open |

---

## 5. Issues found

### 5.1 Default content encodes a fixed 11-column layout — OPEN — Medium

**Description:** The `Table slot` ships with 11 `Table.Column` instances as default content. Column 0 is 166px; columns 1–10 are 137px (`166 + 10×137 = 1536px`). All carry placeholder text.

**Impact:** Every instance of `Table` opens with an 11-column default that must be manually thinned. The width arithmetic only works at exactly 1536px — it does not scale to narrower containers.

**Resolution:** Reduce default slot content to 3–4 columns, or set `displayEmptyByDefault: true` (currently `false`) to force intentional composition. Document a recommended column-count range in the component description.

---

### 5.2 No state coverage (empty, loading, error) — OPEN — Medium

**Description:** `Table` has no BOOLEAN properties for empty state, loading state, or error state. These are structural states that should be toggleable at the organism level.

**Impact:** Designers building empty states, skeleton screens, or error conditions must detach or compose a parallel component — neither path feeds back into the design system.

**Resolution:** Add three BOOLEAN properties:
- `Empty state` (default: `false`) — shows an empty-state region, hides `Table slot`
- `Loading` (default: `false`) — shows a skeleton overlay or shimmer layer
- `Error` (default: `false`) — shows an error message frame

---

### 5.3 Cell padding is hardcoded — OPEN — Low

**Description:** Both `_Table.Header.Cell` and `_Table.Cell` use `padding: 16/12/16/12px`. The `boundVariables` for these nodes contains only `fills` — no spacing token bindings.

**Impact:** If the spacing scale changes (e.g. a compact table variant or a higher density mode is needed), padding must be updated by hand across every cell instance rather than via a token swap.

**Resolution:** Bind cell padding values to spacing tokens from the `Collection 1` token set (e.g. `spacing/3` / `spacing/4` or equivalent). This requires adding spacing variable bindings to `_Table.Header.Cell` and `_Table.Cell` at the component definition level.

---

### 5.4 No row-selection / checkbox column pattern — OPEN — Low

**Description:** No `Selection` BOOLEAN or selection-column slot. The `Interactive=True` variant on `_Table.Cell` exists but has no corresponding selection-column molecule.

**Impact:** Designers implementing selectable rows must detach or create a bespoke column.

**Resolution:** Add a `Selection` BOOLEAN at the Table level that prepends a `_Table.SelectionColumn` instance before `Table slot`. Keep it as a distinct column type, not mixed into the generic `Cells Slot`.

---

### 5.5 `Cells Slot` preferred values undocumented — OPEN — Low

**Description:** `Table.Column`'s `Cells Slot` references one COMPONENT_SET key (`55ea8093...`) in `preferredValues`. There is no component description explaining what this key resolves to or how to extend the allowed set.

**Impact:** When a new cell type is needed (badge cell, status chip, numeric cell), there is no documented path — likely resulting in unconstrained detaches.

**Resolution:** Add a component description to `Table.Column` listing the allowed cell types and the process for adding new ones.

---

## 6. Comparison to anti-patterns

| Anti-pattern | Present? | Evidence |
|---|---|---|
| Monolithic block with content variants instead of slots | ❌ | Column-slot architecture; cells via slot not variants |
| Detached frames where instances should be used | ❌ | All column and cell nodes are instances |
| Generic layer names (`Frame 26`, `Frame 39`) | ❌ | All names semantic — `Table`, `Table.Column`, `_Table.Header.Cell`, `Cells Slot` |
| Mixed button/icon sources | ❌ | Sort icon from same library |
| Template published with sample copy as variants | Partial ⚠ | 11-column default encodes a layout, not just a structural example |
| Variant count encoding layout permutations | ❌ | No variant explosion; `Odd/Even` and `Interactive` are minimal structural axes |
| Properties panel API does not match layer tree | ❌ | One slot property, one slot layer — consistent |
| Unconstrained slot (open allowPreferredValuesOnly) | ❌ | **Fixed** — `allowPreferredValuesOnly: true`, 1 preferred value |

---

## 7. Screenshot analysis

The rendered component (1536×364px) shows 11 columns in a horizontal layout:

- All headers: `"Header label"` in Open Sans Bold, sort icon visible (↕), light grey header background
- Data cells: `"Cell label"` in Open Sans Regular, rows show subtle alternating shading (Odd/Even token)
- Outer table border visible (token: `color/border/tertiary`)
- No row dividers — separation implied by alternating fills
- No empty, loading, or error state visible (expected — no state BOOLEANs)
- Clean composition; column alignment is consistent across the full width

---

## 8. Recommendations

### Done ✅
- Column-first SLOT architecture — correct decomposition model
- Full semantic colour token binding — zero hardcoded hex values
- `_Prefix.Name` atom naming throughout
- `Sort` BOOLEAN on `_Table.Header.Cell` — correct structural toggle
- `Odd`/`Even` cell striping via variant, not baked fills
- **`Table slot` constrained** — `allowPreferredValuesOnly: true`, preferred value added ✅

### Next
1. **[Medium]** Reduce default slot content to 3–4 columns or set `displayEmptyByDefault: true` — removes the baked 11-column layout assumption
2. **[Medium]** Add `Empty state`, `Loading`, `Error` BOOLEAN properties — structural state coverage
3. **[Low]** Bind cell padding to spacing tokens on `_Table.Header.Cell` and `_Table.Cell`
4. **[Low]** Add `Selection` BOOLEAN for a checkbox-column pattern
5. **[Low]** Document `Cells Slot` preferred values and extension process in `Table.Column` description

---

## 9. Scorecard

| Criterion | Score | Notes |
|---|---|---|
| Atoms published & reused | A | `_Table.Header.Cell`, `_Table.Cell`, sort icon all from library; full colour token bindings |
| Molecules compose atoms | A− | `Table.Column` correctly composes header atom + cells slot |
| Organism slot contracts | A− | Two-tier slot structure correct; outer slot now constrained ✅; cell padding not spacing-token-bound |
| Template composition | N/A | Not a template |
| Page tier separation | N/A | Single component |
| Naming & API clarity | A− | All names semantic; one slot property matches one slot layer; state BOOLEANs absent |
| Engineering handoff | B+ | Column-slot maps directly to `<Column>` prop API; `Odd`/`Even` maps to row-index logic; state booleans absent — eng must infer empty/loading behaviour |

**Overall: A−**

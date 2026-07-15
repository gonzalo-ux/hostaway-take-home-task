# Component Audit — Page mock & Template / Page table (Hostaway)

**Subject:** [Page mock](https://www.figma.com/design/pAucTdmNvixw2TnIoLT2fe/Hostaway?node-id=56-9823) · [Template / Page table](https://www.figma.com/design/pAucTdmNvixw2TnIoLT2fe/Hostaway?node-id=36-3634)  
**Context:** [Components page](https://www.figma.com/design/pAucTdmNvixw2TnIoLT2fe/Hostaway?node-id=10-2896)  
**Audit lens:** Atomic design system architecture — composition depth, slot contracts, template API honesty  
**Date:** July 2026  
**Parent audit:** [Audit — Untitled UI](../deliver/audit-untitled-ui.md)  

**Methodology:** Live Figma Desktop Bridge — layer tree, component properties, bindings, screenshots. Re-checked after template cleanup.

---

## Executive summary

The **Page mock** is an unpublished **Page-tier** reference frame. The reusable pattern inside it is **`Template / Page table`** — a thin template that stacks three published organisms: `Page header`, `Filter group`, and `Table`.

| Surface | Node | Tier | Assessment |
|---|---|---|---|
| Page mock | `56:9823` | Page | Correct — demo frame, not in library |
| Template / Page table | `36:3634` | Template | **A−** — clean orchestrator after cleanup |
| Page header | `10:2894` | Organism | **B+** — slot-driven header |
| Filter group | `45:4644` | Organism | **A−** — filter rows as slots |
| Table | `50:6292` | Organism | **A−** — column + cell slots |

**Verdict:** This is the **correct direction** for a product design system — compose from published atoms/molecules/organisms, keep content in slots and props, avoid monolithic variant matrices. After cleanup (orphan properties removed, `Frame 26` → `Table region`), the template is **honest about its API**.

**Overall atomic design grade: A−** (up from B+ pre-cleanup)

---

## 1. Atomic tier map

```
Page mock (FRAME, unpublished)                    ← Page tier
└── Template / Page table (COMPONENT)               ← Template tier
    ├── Page header (INSTANCE)                      ← Organism
    │   ├── Breadcrumbs → Pages slot → _Breadcrumbs.Label
    │   ├── Main → Heading + Actions slot → Button
    │   └── Tabs group → Tabs slot → _Tab.Button
    ├── Filter group (INSTANCE)                     ← Organism
    │   ├── Top row → Filter top row (SLOT) → Input field, Dropdown
    │   ├── Button × 2 (Show all filters, Reset)
    │   └── Bottom row → Filter bottom row (SLOT) → Input field, Dropdown
    └── Table region (FRAME)
        └── Table (INSTANCE)                        ← Organism
            └── Table slot → _Table.Column × N
                ├── _Table.Header.Cell
                └── Cells Slot → _Table.Cell × N
```

### 1.1 Components page inventory (`10:2896`)

| Published asset | Atomic tier |
|---|---|
| Button, Heading, Icons | Atoms |
| Breadcrumbs, Tabs, Input, Dropdown | Molecules |
| Page header, Filter group, Table | Organisms |
| Template / Page table | Template |
| Page mock | Page (unpublished demo) |

Taxonomy labels on the Components page (Atoms / Molecules / Organisms / Template / Page) align with actual structure.

---

## 2. What is built well

### 2.1 Correct compositional depth

Every visible region traces to published library parts — no detached monoliths, no parallel button sources.

### 2.2 Slot contracts at organism level

| Organism | Properties | Contract |
|---|---|---|
| **Page header** | `Breadcrumbs` BOOL, `Tabs` BOOL, `Actions slot` (1–4, Button preferred) | Structural toggles + swappable actions |
| **Filter group** | `Filter top row` SLOT (min 3), `Filter bottom row` SLOT (min 3), `All filters button` BOOL, `Bottom row` BOOL | Filters are instance-level, not variant-encoded |
| **Table** | `Table slot` → `_Table.Column`; per-column `Cells Slot` | Column count and cell content are slots, not variants |

Aligns with the slot model described in the data table pattern spec.

### 2.3 Variants encode structure, not content

- Buttons: `Type=Primary/Ghost/Secondary` — not "Download report with cloud icon"
- Tabs: `_Tab.Button` with text props — not 280 layout permutations
- Table cells: `_Table.Cell` (standard/link × background) — not 204 baked table layouts

### 2.4 Template is appropriately thin

`Template / Page table` orchestrates organisms vertically. It does not re-implement header, filter, or table internals.

### 2.5 Page mock is a valid reference artifact

The Reviews screen demonstrates real content overrides (breadcrumb copy, tab labels, filter labels, 11 columns, sample reservation IDs) while staying fully instanced. That is how templates should behave in atomic design: **configure, don't detach**.

---

## 3. Issues found (initial audit)

### 3.1 Orphan property: `Filter slot#36:21` — FIXED

A SLOT property existed on the template with **no bound slot layer**. The filter area was a hardwired `Filter group` instance. Designers saw "Filter slot" in the Properties panel but nothing in the Layers panel.

**Resolution:** Property removed during cleanup.

### 3.2 Unbound boolean: `More filters#50:44` — FIXED

Duplicate control at template level; `Filter group` already owns `Bottom row#50:47`.

**Resolution:** Property removed during cleanup.

### 3.3 Generic frame naming — PARTIALLY FIXED

| Before | After | Status |
|---|---|---|
| `Frame 26` | `Table region` | ✅ Fixed |
| `Top row` / `Bottom row` (in Filter group) | unchanged | ⚠️ Minor |
| `Main` (in Page header) | unchanged | ⚠️ Minor |

---

## 4. Post-cleanup state (re-check)

**Template / Page table** (`36:3634`) after cleanup:

```
Template / Page table
├── Page header          (instance)
├── Filter group         (instance)
└── Table region         (frame)
    └── Table            (instance)
```

**Template-level properties:** none (correct for a fixed-layout orchestrator)

**Page mock instance** (`56:9823`): still references the template; no leftover orphan props.

| Criterion | Pre-cleanup | Post-cleanup |
|---|---|---|
| Template API clarity | B− | **A−** |
| Naming hygiene | B− | **B+** |
| Overall atomic design | B+ | **A−** |

---

## 5. Remaining gaps (optional, not blockers)

1. **Hardwired organisms** — Header, Filter group, and Table are fixed children. Fine for one page type; consider region slots when a second template is needed (e.g. no filters, cards instead of table).
2. **Filter `minChildren: 3`** — May force filler on sparse screens. Consider `minChildren: 0`.
3. **Table organism placement** — `Table` shell (`50:6292`) lives outside the Table section listing alongside `_Table.Column`, `_Table.Header.Cell`, `_Table.Cell`.
4. **Token typo** — `color/backgorund/default` should be `background`.

---

## 6. Comparison to Untitled UI lift anti-pattern

| Lifted UI kit pattern | Hostaway Page mock |
|---|---|
| Monolithic page header with content variants | Composed `Page header` with slots |
| 204 table variants | `Table slot` + column `Cells Slot` |
| Detach to change layout | Configure via props and slot content |
| No published atoms below | Full atom → molecule → organism chain |

This template demonstrates the **compose-don't-lift** principle from [audit-untitled-ui.md](../deliver/audit-untitled-ui.md).

---

## 7. Recommendations

### Done ✅

- Remove orphan `Filter slot` and `More filters` properties
- Rename `Frame 26` → `Table region`

### Next (when scaling)

- Add region slots to template when a second page layout appears
- Relax filter slot `minChildren` if sparse filter rows are common
- Surface `Table` organism in the Table section
- Fix token naming typo (`backgorund` → `background`)

---

## 8. Scorecard

| Criterion | Score |
|---|---|
| Atoms published & reused | A |
| Molecules compose atoms | A− |
| Organisms use slots, not content variants | A− |
| Template composes organisms | A− |
| Page tier separation | A |
| Naming & API clarity | B+ |
| Engineering handoff contract | A− |

**Overall: A−**

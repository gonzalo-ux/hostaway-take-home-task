# Component Audit — Filter group (Hostaway)

**Subject:** [Filter group](https://www.figma.com/design/pAucTdmNvixw2TnIoLT2fe/Hostaway?node-id=45-4644)  
**Audit lens:** Atomic design system architecture  
**Date:** July 2026  
**Parent audit:** [audit-untitled-ui.md](../deliver/audit-untitled-ui.md)  
**Related:** [page-mock-audit.md](./page-mock-audit.md) · [untitled-ui-tabs-audit.md](./untitled-ui-tabs-audit.md)  
**Methodology:** Live Figma Desktop Bridge — layer tree, properties, bindings, screenshots.

---

## Executive summary

`Filter group` (`45:4644`) is a standalone COMPONENT (not a set) that functions as a filter bar organism: a horizontal row of input molecules with optional secondary row, anchored by action buttons. It uses proper SLOT contracts for filter input placement and BOOLEAN toggles for optional structural regions. The slot bindings are clean — zero orphans — and all token bindings resolve through `Collection 1` semantic aliases. The main weaknesses are a `minChildren: 3` constraint on both slots (forces filler on sparse screens) and a `Reset` button baked into the shell without a visibility toggle, breaking slot-contract symmetry with the `All filters button` boolean.

| Surface | Node | Tier | Assessment |
|---|---|---|---|
| Filter group | `45:4644` | Organism | A− — slot-driven shell; baked Reset button and minChildren constraint are the gaps |

**Verdict:** Compose, don't lift. This component demonstrates the target architecture — slots over variants, atoms instanced from library, token bindings throughout. Fix the Reset toggle and minChildren documentation before promoting as a reference pattern.

**Overall atomic design grade: A−**

---

## 1. Atomic tier map

```
Filter group  [COMPONENT — Organism]
├── Top row  [FRAME]
│   ├── Filter top slot  [SLOT]  ← bound to "Filter top slot#45:30"
│   │   ├── Input field  [INSTANCE → Input field molecule]
│   │   │   ├── _Input.Label  [INSTANCE — Atom]
│   │   │   └── _Input.Field  [INSTANCE — Atom]
│   │   │       ├── Prefix  [FRAME]
│   │   │       │   └── variant  [INSTANCE — icon atom]
│   │   │       └── _Input.Text  [INSTANCE — Atom]
│   │   ├── Input field  [INSTANCE → Input field molecule]  (repeat)
│   │   ├── Dropdown  [INSTANCE → Dropdown molecule]
│   │   │   ├── _Input.Label  [INSTANCE — Atom]
│   │   │   └── _Dropdown.Button  [INSTANCE — Atom]
│   │   │       ├── _Input.Text  [INSTANCE — Atom]
│   │   │       └── Suffix  [FRAME → chevron-down icon]
│   │   ├── Dropdown  [INSTANCE]  (repeat ×2)
│   │   └── (5 default children — hits minChildren:3)
│   ├── Button "Show all filters"  [INSTANCE — Ghost]  ← toggled by "All filters button" BOOLEAN
│   └── Button "Reset"  [INSTANCE — Secondary]  ← baked, no toggle ⚠
└── Bottom row  [FRAME — hidden by default]  ← toggled by "Bottom row" BOOLEAN
    └── Filters bottom slot  [SLOT]  ← bound to "Filters bottom slot#50:46"
```

**Nesting depth:** 5 levels (organism → frame → slot → molecule → atom). Within atomic budget.

---

## 2. Property and slot contract

| Property | Type | Default | Bound | Notes |
|---|---|---|---|---|
| `Filter top slot` | SLOT | — | ✅ `45:4643` | minChildren:3, allowPreferredValuesOnly |
| `All filters button` | BOOLEAN | `true` | ✅ | Toggles ghost "Show all filters" button |
| `Filters bottom slot` | SLOT | — | ✅ `50:7085` | minChildren:3, allowPreferredValuesOnly |
| `Bottom row` | BOOLEAN | `false` | ✅ | Hides entire second row frame |

**Orphan scan result:** 0 orphans — all SLOT properties have matching SLOT layers with correct `slotContentId` references.

**Slot preferred values (both slots share the same set):**
- COMPONENT_SET key `e8bc5d9...` (Input field set)
- COMPONENT key `b44110...` (likely Input field variant)
- COMPONENT key `cdaf8c...` (likely Dropdown variant)

`allowPreferredValuesOnly: true` is set on both slots — only approved molecules can be inserted.

---

## 3. What is built well

### 3.1 Slot contracts are clean and symmetric

Both filter rows use proper SLOT layers bound to SLOT properties. No orphan properties, no ghost API surface. The preferred values list constrains the slot to approved molecules, which prevents ad hoc detached frames from being inserted.

### 3.2 Full token binding throughout

Every visual property in the tree resolves to a `Collection 1` semantic alias:

| Property | Token |
|---|---|
| Text fill | `color/text/primary` |
| Input background | `color/background/default` |
| Input border | `color/border/secondary` |
| Icon stroke | `color/icon/primary` |
| Icon interactive | `color/icon/interactive` |
| Button fill | `color/background/secondary` |
| Button text | `color/text/interactive` |

Zero hardcoded hex values in the component tree — full alias compliance.

### 3.3 `_Prefix.Name` atom naming throughout

All leaf atoms follow the convention: `_Input.Label`, `_Input.Field`, `_Input.Text`, `_Dropdown.Button`. Icon instances are named `chevron-down`, `filter`, `variant` — semantic where nameable. This matches the target naming standard from the roadmap.

### 3.4 BOOLEAN toggles for optional structure

`All filters button` and `Bottom row` are correctly modelled as BOOLEANs — they control structural visibility rather than encoding content permutations as variants. This is the intended pattern.

---

## 4. Issues found

### 4.1 `minChildren: 3` on both slots — OPEN — Medium

**Description:** Both `Filter top slot` and `Filters bottom slot` have `slotSettings.minChildren = 3`. This means Figma will show a validation warning on any instance that inserts fewer than 3 filter items. The default slot content itself has 5 children (2 inputs + 3 dropdowns) — comfortably above the minimum — but real product screens with only 1–2 filters will not satisfy this constraint.

**Impact:** Designers on sparse screens (e.g. a page with only a search + one dropdown) will be forced to pad the slot with dummy filters or ignore the warning. Either outcome undermines the slot contract.

**Resolution:** Unless the layout genuinely breaks below 3 items (verify), reduce `minChildren` to `1` on both slots. If a visual minimum is needed, enforce it in the layout's `minWidth` of the slot frame, not via child count.

---

### 4.2 `Reset` button baked without a visibility toggle — OPEN — Medium

**Description:** The `Reset` button (Secondary variant, `42:4254`) is a hardcoded instance inside `Top row`. The `All filters button` BOOLEAN correctly toggles the ghost "Show all filters" button, but there is no corresponding `Reset button` BOOLEAN. Every instance of `Filter group` will show a Reset button.

**Impact:** Asymmetric API. Consumers who want a filter bar without a Reset action (e.g. filters that auto-apply, or a read-only filter summary) must detach the component to remove it.

**Resolution:** Add a `Reset button` BOOLEAN property (default: `true`) wired to the Reset button's visibility, mirroring the `All filters button` pattern already in place.

---

### 4.3 Bottom row progressive disclosure is undocumented — OPEN — Low

**Description:** The `Bottom row` BOOLEAN defaults to `false`, hiding the second slot entirely. This is a valid progressive disclosure pattern, but without documentation in the component description, consumers are unlikely to discover the second slot exists.

**Impact:** The `Filters bottom slot` goes unused in practice — effectively orphaned at the UX layer even though the binding is correct at the API layer.

**Resolution:** Add a component description (Figma Dev Mode annotation) explaining: "Toggle `Bottom row` to `true` to reveal a second row of filter inputs. Use for overflow filters or advanced filter panels."

---

### 4.4 `allowPreferredValuesOnly: true` undocumented — OPEN — Low

**Description:** Both slots restrict insertable content to 3 allowed component keys. This is intentionally tight, but the allowed set excludes future filter molecules (e.g. date range picker, tag multiselect, search-with-autocomplete). There is no documentation explaining the constraint or a process for extending it.

**Impact:** When a new filter molecule is designed, the engineer or designer inserting it will hit a silent restriction with no guidance on how to update the preferred values list.

**Resolution:** Document the allowed molecules in the component description. Establish a review gate: adding a molecule to the preferred values list requires a DS team sign-off.

---

## 5. Comparison to anti-patterns

| Anti-pattern (from audit-untitled-ui.md) | Present? | Evidence |
|---|---|---|
| Monolithic block with content variants instead of slots | ❌ | Slots used correctly |
| Detached frames where instances should be used | ❌ | All leaf nodes are instances |
| Generic layer names (`Frame 26`, `Frame 39`) | ❌ | `Top row`, `Bottom row`, `Filter top slot` — semantic |
| Mixed button/icon sources (kit + custom) | ❌ | All from same library |
| Template published with sample copy as variants | ❌ | No variants; no baked content labels |
| Variant count encoding layout permutations | ❌ | Single COMPONENT, no variant explosion |
| Properties panel API does not match layer tree | Partial ⚠ | Reset button visible in tree but not in properties panel |

The only anti-pattern present is partial: the Reset button is a structural child with no corresponding property, creating a gap between what the properties panel exposes and what is in the layer tree.

---

## 6. Screenshot analysis

The rendered component (1600×104px) shows a single-row filter bar:
- 2 Input field molecules (icon prefix visible, label above field)
- 3 Dropdown molecules (chevron suffix, label above field)
- Ghost "Show all filters" button (filter icon + teal label)
- Secondary "Reset" button (teal fill, no icon)

All items baseline-align correctly. The `Bottom row` is not visible (default `false`). The slot's 5 default children demonstrate the intended layout but exceed the `minChildren: 3` minimum — the constraint is not visible from the rendered state alone.

---

## 7. Recommendations

### Done ✅
- SLOT contracts in place for both filter rows — no orphans
- Full semantic token binding throughout — zero hardcoded values
- `_Prefix.Name` atom naming convention applied consistently
- BOOLEAN toggles for `All filters button` and `Bottom row` structural regions
- `allowPreferredValuesOnly` set to constrain slot to approved molecules

### Next
1. **[Medium]** Reduce `minChildren` from 3 → 1 on both slots unless layout genuinely breaks below 3 items
2. **[Medium]** Add `Reset button` BOOLEAN property (default: `true`) wired to Reset button visibility — mirrors existing `All filters button` pattern
3. **[Low]** Write component description documenting: `Bottom row` progressive disclosure pattern + the `allowPreferredValuesOnly` allowed molecule list
4. **[Low]** Add a preferred-values update process to the publish checklist (DS team sign-off required to extend the allowed slot types)

---

## 8. Scorecard

| Criterion | Score | Notes |
|---|---|---|
| Atoms published & reused | A | All leaf nodes from `_Prefix.Name` library atoms; full token bindings |
| Molecules compose atoms | A− | Input field and Dropdown correctly compose atoms; Reset button has no toggle counterpart |
| Organism slot contracts | B+ | 2 clean SLOT bindings, zero orphans; deducted for minChildren:3 and baked Reset button |
| Template composition | N/A | Not a template |
| Page tier separation | N/A | Single component |
| Naming & API clarity | A− | Semantic slot and boolean names; Reset button visible in tree but absent from properties panel |
| Engineering handoff | B+ | SLOT names map cleanly to props; preferred values constrain allowed children; minChildren:3 has no code equivalent and needs callout |

**Overall: A−**

# Component Audit — Tabs group (Hostaway)

**Subject:** [Tabs group](https://www.figma.com/design/pAucTdmNvixw2TnIoLT2fe/Hostaway?node-id=4-2710)  
**Audit lens:** Atomic design system architecture  
**Date:** July 2026  
**Parent audit:** [audit-untitled-ui.md](../deliver/audit-untitled-ui.md)  
**Related:** [filter-group-audit.md](./filter-group-audit.md) · [table-audit.md](./table-audit.md) · [page-mock-audit.md](./page-mock-audit.md)  
**Methodology:** Live Figma Desktop Bridge — layer tree, properties, bindings, screenshots.

---

## Executive summary

`Tabs group` (`4:2710`) is a two-variant COMPONENT_SET sitting at the **molecule** tier: a horizontal container of `_Tab.Button` atoms composed via a single `Tabs slot`. The `Width` axis (Auto/HUG vs Full/FILL) is the sole structural variant — clean single-axis decomposition with no content variants. Token binding is complete for all colour properties; padding values (4px container, 12/8px button, 24px gap) are hardcoded. The slot contract is tight: `allowPreferredValuesOnly: true`, one preferred COMPONENT_SET, zero orphans across both variants.

The main gaps are `minChildren: 3` on the slot (forces filler on 1–2 tab screens), hardcoded spacing values, and a 5-tab default that encodes a specific layout assumption on every new instance.

| Surface | Node | Tier | Assessment |
|---|---|---|---|
| Tabs group | `4:2710` | Molecule | A− — slot contract correct; minChildren constraint and hardcoded spacing are the gaps |

**Verdict:** Compose, don't lift. Slot architecture and naming are correct. Reduce `minChildren` and bind padding to spacing tokens before promoting as a reference pattern.

**Overall atomic design grade: A−**

---

## 1. Atomic tier map

```
Tabs group  [COMPONENT_SET — Molecule]
├── Width=Auto  [COMPONENT]
│   │  fill → color/background/select light
│   │  padding: 4px all sides (hardcoded ⚠)
│   │  cornerRadius: 8
│   │  layoutMode: HORIZONTAL, layoutSizing: HUG×HUG
│   └── Tabs slot  [SLOT — bound to "Tabs slot#4:8"]
│       │  layoutMode: HORIZONTAL, layoutSizing: HUG×HUG
│       │  itemSpacing: 24px (hardcoded ⚠)
│       │  allowPreferredValuesOnly: true ✅
│       │  minChildren: 3  ⚠  maxChildren: null
│       │  preferredValues: [COMPONENT_SET 0e72e8ec...]
│       └── _Tab.Button ×5  [INSTANCE — Atom]
│           │  Type=Selected (1st) · Type=Idle (rest)
│           │  fill → color/background/select / color/background/select light
│           │  padding: 12/8/12/8px (hardcoded ⚠)
│           │  minWidth: 116px
│           └── "Tab label"  [TEXT — Open Sans SemiBold 14/20]
│                 fill → color/text/interactive
│
└── Width=Full  [COMPONENT]
    │  fill → color/background/select light
    │  layoutMode: HORIZONTAL, layoutSizing: FILL×HUG
    └── Tabs slot  [SLOT — bound to "Tabs slot#4:8"]
        │  layoutMode: HORIZONTAL, layoutSizing: FILL×HUG
        └── _Tab.Button ×5  [INSTANCE — layoutSizingHorizontal: FILL]
```

**Nesting depth:** 3 levels (molecule → slot → atom). Well within atomic budget.

**Font note:** Open Sans SemiBold 14/20 throughout — consistent with all other components in the system.

---

## 2. Property and slot contract

| Property | Type | Default | Bound | Notes |
|---|---|---|---|---|
| `Tabs slot` | SLOT | — | ✅ `4:8` | `allowPreferredValuesOnly: true` ✅; minChildren: 3 ⚠; 1 preferred value |
| `Width` | VARIANT | `Auto` | — | `Auto` (HUG) · `Full` (FILL) |

**Orphan scan:** 0 orphans. Both SLOT layers (`4:2685` in Width=Auto, `4:2712` in Width=Full) correctly reference `slotContentId: "Tabs slot#4:8"`. Slot property and both slot layers are in agreement.

**Slot preferred values:**
- COMPONENT_SET key `0e72e8ecf4f61b76c79bfc47c51650bb9a498cc8` — resolves to `_Tab.Button` component set

---

## 3. What is built well

### 3.1 Constrained slot with correct preferred values

`Tabs slot` has `allowPreferredValuesOnly: true` and a single preferred COMPONENT_SET. Only `_Tab.Button` can be inserted. Both variant variants bind their SLOT layers to the same property key — the API is consistent and zero orphans exist.

### 3.2 Clean single-axis variant

`Width=Auto` (HUG) and `Width=Full` (FILL) are the only two structural variants. No content-driven variant explosion. The axis maps directly to a single CSS `width` prop in code:

| Variant | Figma | CSS equivalent |
|---|---|---|
| Width=Auto | `layoutSizingHorizontal: HUG` | `display: inline-flex; width: fit-content` |
| Width=Full | `layoutSizingHorizontal: FILL` | `display: flex; width: 100%` |

### 3.3 Full token binding for colours

| Layer | Property | Token |
|---|---|---|
| Container (both variants) | fill | `color/background/select light` |
| `_Tab.Button` (Selected) | fill | `color/background/select` |
| `_Tab.Button` (Idle) | fill | `color/background/select light` |
| Tab label text | fill | `color/text/interactive` |

Zero hardcoded hex values in colour bindings.

### 3.4 `_Prefix.Name` atom naming

`_Tab.Button` follows the `_Prefix.Name` convention. `Tabs slot` is a clear, semantic slot name. No generic frame names.

### 3.5 Tab button state managed at the atom, not the group

Selected/Idle distinction is handled via `Type=Selected/Idle` on individual `_Tab.Button` instances — not encoded as a group-level variant. The group shell stays structurally minimal. This is the correct pattern: the group manages layout; the atom manages state.

---

## 4. Issues found

### 4.1 `minChildren: 3` on the slot — OPEN — Medium

**Description:** `Tabs slot` has `slotSettings.minChildren = 3`. Default slot content ships with 5 children — comfortably above the minimum — but real screens with 1 or 2 tabs (e.g. a simple Active/Archived toggle) will trigger a Figma validation warning.

**Impact:** Designers building sparse tab bars are forced to pad with dummy tabs or dismiss the warning. Either outcome undermines the slot contract as a reliable guide.

**Resolution:** Reduce `minChildren` to `1` unless the horizontal layout demonstrably breaks below 3 items. If a visual minimum is needed, enforce it via `minWidth` on the slot frame rather than child count.

---

### 4.2 Default content encodes a 5-tab layout — OPEN — Low

**Description:** Both variants ship with 5 `_Tab.Button` instances as default slot content (`displayEmptyByDefault: false`). This assumes a specific tab count on every new instance.

**Impact:** Every placement of `Tabs group` opens with a 5-tab layout the designer must manually clear. The 5-tab default also makes the `minChildren: 3` constraint invisible — a designer working from the default will never encounter the warning.

**Resolution:** Reduce default slot content to 2 instances (1 Selected + 1 Idle) as the minimal meaningful example, or set `displayEmptyByDefault: true` to prompt intentional composition.

---

### 4.3 Padding and gap are hardcoded — OPEN — Low

**Description:** Container padding (4px all sides), tab button padding (12px horizontal / 8px vertical), and inter-tab gap (24px) are hardcoded pixel values. `boundVariables` for these layers contains only `fills` — no spacing token bindings.

**Impact:** Density changes (compact mode, mobile adaptation, any future spacing scale update) require per-instance manual edits instead of a token swap.

**Resolution:** Bind container padding and inter-tab gap to spacing tokens from `Collection 1`. At a minimum, document the intended values in the component description so the engineering handoff is unambiguous.

---

### 4.4 `Tabs slot` preferred values undocumented — OPEN — Low

**Description:** The preferred COMPONENT_SET key (`0e72e8ec...`) resolves to `_Tab.Button` but this is not stated in the component description. There is no guidance for extending the allowed set.

**Impact:** When a new tab variant is needed (e.g. tab with badge count, tab with leading icon), the designer or engineer will encounter a silent restriction with no documented path to extend it.

**Resolution:** Add a component description: "Tabs slot accepts `_Tab.Button` instances only. To add a new tab type, add a variant to `_Tab.Button` and then update the preferred values here."

---

## 5. Comparison to anti-patterns

| Anti-pattern | Present? | Evidence |
|---|---|---|
| Monolithic block with content variants instead of slots | ❌ | Single SLOT drives all composition |
| Detached frames where instances should be used | ❌ | All tab buttons are `_Tab.Button` instances |
| Generic layer names (`Frame 26`, `Frame 39`) | ❌ | `Tabs slot`, `Width=Auto`, `Width=Full` — semantic throughout |
| Mixed button/icon sources | ❌ | Single atom type from library |
| Template published with sample copy as variants | ❌ | No baked content labels; placeholder text only |
| Variant count encoding layout permutations | ❌ | 2 variants only — clean Width axis |
| Properties panel API does not match layer tree | ❌ | 1 SLOT property + 1 VARIANT; both match layer tree |
| Unconstrained slot | ❌ | `allowPreferredValuesOnly: true`, 1 preferred COMPONENT_SET |

No anti-patterns present. This is among the cleanest component implementations in the audit set.

---

## 6. Screenshot analysis

The rendered component (1264×112px) shows both variants stacked:

- **Row 1 (Width=Auto, 684×44px):** 5 tab buttons, HUG-width container with teal (`color/background/select light`) background. First tab has stronger teal fill (Selected variant of `_Tab.Button`); remaining 4 are lighter (Idle). All labeled "Tab label" in Open Sans SemiBold teal. 24px gap between buttons.
- **Row 2 (Width=Full, 1248×44px):** Same 5 tabs, each stretching to ~229px (FILL horizontal). Selected/Idle fill distinction preserved. Container spans full width.

Clean alignment, consistent baseline, no overflow. `minChildren: 3` constraint is not visible in this default state — the 5 default children mask it entirely.

---

## 7. Recommendations

### Done ✅
- SLOT contract constrained — `allowPreferredValuesOnly: true`, 1 preferred COMPONENT_SET, zero orphans in both variants
- Clean 2-variant set — `Width` axis only, no content variants, no variant explosion
- Full token binding for all colour properties — zero hardcoded hex values
- `_Prefix.Name` atom naming on tab button instances
- Tab state (Selected/Idle) managed at atom level — group shell stays structurally minimal

### Next
1. **[Medium]** Reduce `minChildren` from 3 → 1 on `Tabs slot` unless the container layout breaks below 3 items
2. **[Low]** Reduce default slot content to 2 instances (1 Selected + 1 Idle) or set `displayEmptyByDefault: true`
3. **[Low]** Bind container padding and inter-tab gap to spacing tokens from `Collection 1`
4. **[Low]** Add component description documenting the preferred tab atom and the process for extending the allowed set

---

## 8. Scorecard

| Criterion | Score | Notes |
|---|---|---|
| Atoms published & reused | A | `_Tab.Button` from library; `_Prefix.Name` naming throughout; correct instance reuse |
| Molecules compose atoms | A | Single slot, single atom type; HORIZONTAL container manages layout only |
| Organism slot contracts | A− | SLOT constrained ✅; `minChildren: 3` may force filler on sparse screens; preferred values undocumented |
| Template composition | N/A | Not a template |
| Page tier separation | N/A | Single component set |
| Naming & API clarity | A | `Width` variant axis clean; `Tabs slot` self-documenting; no ghost properties or orphan API surface |
| Engineering handoff | A− | `Width` maps directly to CSS width; slot maps to children array; padding hardcoded — spacing callout needed |

**Overall: A−**

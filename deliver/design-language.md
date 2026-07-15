# Hostaway Design Language

> **Status:** Initial system — foundational tokens, 8 component sets, and 2 page templates are defined. Gaps noted throughout as **TBD**. This document is the canonical reference for designers, engineers, and AI tooling working in this system.

**Figma source:** [Hostaway — Components page](https://www.figma.com/design/pAucTdmNvixw2TnIoLT2fe/Hostaway?node-id=10-2896)

---

## 1. Design Principles

The system is built around three structural rules that apply to every decision:

1. **Token-bound, not hardcoded.** Every color, spacing, and typographic decision in a component is bound to a named variable. Hardcoded values are a defect, not a style choice.
2. **Slots over variants.** Composition points (lists of tabs, action buttons, filter rows) are exposed as named Figma Slots, not duplicated as variants. Content permutations live outside the component.
3. **Private parts are prefixed.** Any component prefixed with `_` (e.g. `_Tab.Button`, `_Table.Cell`) is an internal building block. It is not composed directly by designers or instantiated directly by engineers — it is consumed by its parent component.

---

## 2. Token System

All variables live in a single Figma variable collection: **Collection 1**, with one mode (**Mode 1** — light theme).

> **TBD:** Dark mode, named mode aliases (e.g. `light` / `dark`), and a second collection for spacing and sizing tokens. The collection itself is unnamed — rename to something explicit (e.g. `Hostaway/Core`) before expanding.

### 2.1 Color Palette (Primitives)

Raw scale values. Not used directly in components — always aliased through semantic tokens.

| Token | Hex | Notes |
|---|---|---|
| `color/palette/green/100` | `#F8FBFC` | Lightest tint |
| `color/palette/green/200` | `#DFECEC` | |
| `color/palette/green/300` | `#CCE6E6` | |
| `color/palette/green/500` | `#44A29F` | Brand mid |
| `color/palette/green/700` | `#237472` | Brand dark / interactive |
| `color/palette/neutral/white` | `#FFFFFF` | |
| `color/palette/neutral/black` | `#000000` | |
| `color/palette/neutral/50` | `#F9FAFB` | |
| `color/palette/neutral/100` | `#F3F4F6` | |
| `color/palette/neutral/200` | `#E5E7EB` | |
| `color/palette/neutral/300` | `#D1D5DC` | |
| `color/palette/neutral/400` | `#99A1AF` | |
| `color/palette/neutral/500` | `#6A7282` | |
| `color/palette/neutral/600` | `#4A5565` | |
| `color/palette/neutral/700` | `#364153` | |
| `color/palette/neutral/800` | `#1E2939` | |
| `color/palette/neutral/900` | `#101828` | Darkest neutral |

> **Note:** There is a consistent typo in the Figma variable names: `backgorund` (missing 'r'). This appears across all background tokens and must be corrected before a code export or token pipeline is set up. Treat `backgorund` as `background` throughout this document.

### 2.2 Semantic Color Tokens

These are the tokens components bind to. Always use these — never a palette primitive directly.

#### Background

| Token | Resolves to | Hex | Usage |
|---|---|---|---|
| `color/background/default` | `neutral/white` | `#FFFFFF` | Default surface |
| `color/background/primary` | `green/500` | `#44A29F` | Primary button fill, brand surface |
| `color/background/secondary` | `green/300` | `#CCE6E6` | Secondary button fill |
| `color/background/light` | `neutral/100` | `#F3F4F6` | Table header, subtle backgrounds |
| `color/background/lighter` | `neutral/50` | `#F9FAFB` | Alternating table rows (even) |
| `color/background/select` | `green/200` | `#DFECEC` | Selected tab |
| `color/background/select light` | `green/100` | `#F8FBFC` | Tab container, idle tab |
| `color/background/overlay` | `#00000080` | 50% black | Modal/drawer backdrop |

#### Text

| Token | Hex | Usage |
|---|---|---|
| `color/text/primary` | `#31343A` | All body and heading text |
| `color/text/secondary` | `#676D7A` | Supporting text, labels |
| `color/text/interactive` | `#237472` | Links, interactive text elements |
| `color/text/inverse` | `#FFFFFF` | Text on dark/brand fills |

#### Border

| Token | Resolves to | Hex | Usage |
|---|---|---|---|
| `color/border/primary` | `neutral/900` | `#101828` | Strong borders |
| `color/border/secondary` | `neutral/200` | `#E5E7EB` | Input borders, dividers |
| `color/border/tertiary` | `neutral/100` | `#F3F4F6` | Table outer border |

#### Icon

| Token | Resolves to | Hex | Usage |
|---|---|---|---|
| `color/icon/primary` | `neutral/900` | `#101828` | Default icon color |
| `color/icon/interactive` | `green/700` | `#237472` | Actionable icons |
| `color/icon/inverse` | `neutral/white` | `#FFFFFF` | Icons on brand fills |

### 2.3 Spacing & Sizing

> **TBD:** No spacing or sizing tokens are defined yet. The values below are extracted directly from component layout properties and should be promoted to tokens as the system matures.

Observed values in use:

| Value | Where used |
|---|---|
| `4px` | Internal icon gaps, label gaps, cell label gap |
| `8px` | Input field internal gap |
| `12px` | Input/cell horizontal padding, button horizontal padding (min) |
| `16px` | Button horizontal padding (standard), cell vertical padding, page header vertical padding |
| `24px` | Tab gap, filter row gap, page header gap between heading row and tabs |
| `28px` | Gap between breadcrumbs and heading row in Page header |
| `32px` | Page header and filter group horizontal padding |

### 2.4 Typography

> **TBD:** No Figma text styles are defined. Two distinct font families are in use across components (see below). A full type scale with defined styles is needed before the system can be considered complete.

**Font families observed:**

| Family | Usage |
|---|---|
| **Raleway** | Headings (`Heading 1`, `Heading 2`), all interactive labels (Button, Tab, Input label, Input text) |
| **Open Sans** | Table data — header labels and cell labels |

**Type values observed in components:**

| Role | Family | Style | Size | Token |
|---|---|---|---|---|
| Heading 1 | Raleway | Bold | 34px | — |
| Heading 2 | Raleway | — | ~28px | — |
| Label (small) | Raleway | — | 18px | — |
| Body / Prominence4 | Raleway | — | 20px | — |
| Button label | Raleway | SemiBold | 16px | — |
| Tab label | Raleway | SemiBold | 14px | — |
| Input label | Raleway | Medium | 14px | — |
| Input text | Raleway | Medium | 14px | — |
| Table header | Open Sans | Bold | 14px | — |
| Table cell | Open Sans | Regular | 14px | — |

All text colors are bound to semantic tokens (`color/text/primary`, `color/text/interactive`, `color/text/inverse`).

---

## 3. Component Inventory

Components follow a three-tier naming convention:

- **`_Component.Part`** — private sub-component, not composed directly
- **`Component`** — public building block (atom or molecule)
- **`Template / Name`** — page-level assembly, not reused as a nested component

### 3.1 Component Sets (with variants)

| Name | Variants | Key props |
|---|---|---|
| `Button` | Primary, Secondary, Ghost | `Type` (variant), `Label` (text), `Prefix` (boolean), `Icon` (instance swap) |
| `_Tab.Button` | Idle, Selected | `Type` (variant), `Label` (text) |
| `Tabs group` | Auto, Full | `Width` (variant), `Tabs slot` (slot → `_Tab.Button`) |
| `Text` | Heading 1, Heading 2, Label, Prominence4 | `Prominence` (variant) |
| `_Breadcrumbs.Label` | Current=No, Current=Yes | `Current` (variant) |
| `_Input.Field` | Default, Chip | `Type` (variant), `Prefix` (boolean), `Icon` (instance swap) |
| `_Dropdown.Button` | Large (44px), Small (36px) | `Size` (variant) |
| `_Table.Cell` | Odd/Even × Interactive True/False | `Type` + `Interactive` (variants), `Cell label` (text) |

### 3.2 Standalone Components (public)

| Name | Size | Key props / slots |
|---|---|---|
| `Button/Primary/Default` | 137×48 | — (legacy; use `Button` component set) |
| `Input field` | 192×68 | Composes `_Input.Label` + `_Input.Field` |
| `Page header` | 1600×200 | `Tabs` (boolean), `Breadcrumbs` (boolean), `Actions slot` (slot → Button) |
| `Table.Column` | 137×364 | `Cells Slot` (slot → `_Table.Cell`) |
| `_Table.Header.Cell` | 137×52 | `Header cell` (text), `Sort` (boolean) |
| `Table` | 1536×364 | `Table slot` (slot → `Table.Column`) |
| `Icon - share` | — | Static icon |
| `Icon - download` | — | Static icon |
| `Icon - settings` | — | Static icon |
| `placeholder` | — | Utility |
| `check-contained` | — | Utility |

### 3.3 Templates

| Name | Size | Contents |
|---|---|---|
| `Template / Page table` | 1600×792 | Page header + Filter group + Table + Pagination |
| `Template / Table + pagination` | 1536×432 | Table + optional Pagination (`Pagination` boolean prop) |

---

## 4. Component Specs

### Button

**Figma node:** `3:2548` (component set)

**Variants:**

| Type | Fill | Text color | Border |
|---|---|---|---|
| Primary | `color/background/primary` → `#44A29F` | `color/text/inverse` → `#FFFFFF` | none |
| Secondary | `color/background/secondary` → `#CCE6E6` | — | — |
| Ghost | — | — | — |

**Layout:** Horizontal, `padding: 12px 16px`, `gap: 4px`, center-aligned both axes
**Size:** 150×44px (all variants)
**Corner radius:** 999px (pill)
**Icon:** 20×20, color bound to `color/icon/inverse`

**Props:**
| Prop | Type | Default |
|---|---|---|
| `Type` | variant | `Primary` |
| `Label` | text | `"Button label"` |
| `Prefix` | boolean | `true` |
| `Icon` | instance swap | chevron icon |

---

### _Tab.Button

**Figma node:** `4:2680` (component set)

| State | Fill | Text color |
|---|---|---|
| Selected | `color/background/select` → `#DFECEC` | `color/text/interactive` → `#237472` |
| Idle | `color/background/select light` → `#F8FBFC` | `color/text/interactive` → `#237472` |

**Layout:** Horizontal, `padding: 8px 12px`, `gap: 4px`
**Size:** 116×36px
**Corner radius:** 4px

**Props:**
| Prop | Type | Default |
|---|---|---|
| `Type` | variant | `Idle` |
| `Label` | text | `"Tab label"` |

---

### Tabs group

**Figma node:** `4:2710` (component set)

**Variants:** `Width=Auto` (hugs content, 684px default) / `Width=Full` (fills container, 1248px)

**Container:** `color/background/select light` → `#F8FBFC`, `corner-radius: 8px`, `padding: 4px`, `gap: 24px`

**Slot:** `Tabs slot` — accepts `_Tab.Button` instances, min 3, no max. The slot is horizontal, `gap: 24px`.

> The gap between tab buttons (24px) is wider than the internal button padding gap (4px). This means tab spacing is controlled at the group level, not the button level.

---

### Input field

**Figma node:** `42:4210` (standalone component)

Assembly of two private parts:

```
Input field (VERTICAL, gap: 4px)
├── _Input.Label      — "Input label", Raleway Medium 14px, color/text/primary
└── _Input.Field      — fill: color/background/default, stroke: color/border/secondary (1px), radius: 4px
    ├── Prefix frame  — 20×20 icon slot (boolean toggle)
    └── _Input.Text field — Raleway Medium 14px, color/text/primary
```

**Input field container layout:** Horizontal, `padding: 12px`, `gap: 8px`
**Default size:** 192×68px (label 20px + 4px gap + field 44px)

**_Input.Field props:**
| Prop | Type | Default |
|---|---|---|
| `Type` | variant | `Default` |
| `Prefix` | boolean | `true` |
| `Icon` | instance swap | (multiple preferred icons) |

---

### Table system

The table is composed column-by-column. Each column is an independent component that grows vertically.

```
Table (HORIZONTAL, no padding, no gap)              — border: color/border/tertiary, 2px, radius: 8px
└── Table slot (slot, accepts Table.Column)
    └── Table.Column (VERTICAL, no padding, no gap)  — fill: color/background/default
        ├── _Table.Header.Cell                        — fill: color/background/light (#F3F4F6), padding: 16px 12px
        │   ├── Header label  (Open Sans Bold 14px, color/text/primary)
        │   └── sort-vertical-01 icon (color/icon/primary)
        └── Cells Slot (slot, accepts _Table.Cell, min 2)
            ├── _Table.Cell (Type=Odd)  — fill: color/background/default (#FFFFFF)
            └── _Table.Cell (Type=Even) — fill: color/background/lighter (#F9FAFB)
```

**_Table.Cell layout:** Horizontal, `padding: 16px 12px`, `gap: 4px`
**Cell size:** 120–166×52px (width varies by column width, height fixed at 52px)

**_Table.Cell props:**
| Prop | Type | Options |
|---|---|---|
| `Type` | variant | `Odd`, `Even` |
| `Interactive` | variant | `True`, `False` |
| `Cell label` | text | `"Cell label"` |

**_Table.Header.Cell props:**
| Prop | Type | Default |
|---|---|---|
| `Header cell` | text | `"Header label"` |
| `Sort` | boolean | `true` |

**Table.Column slot:**
| Slot | Accepts | Min children |
|---|---|---|
| `Cells Slot` | `_Table.Cell` | 2 |

**Table slot:**
| Slot | Accepts | Constraints |
|---|---|---|
| `Table slot` | `Table.Column` | none (open) |

---

### Page header

**Figma node:** `10:2894` (standalone component)

```
Page header (VERTICAL, padding: 16px 32px, gap: 28px)  — fill: color/background/default
├── Breadcrumbs instance
│   └── Pages slot (HORIZONTAL, gap: 4px)
│       └── _Breadcrumbs.Label instances + chevron icons
├── Frame 39 (HORIZONTAL, gap: 24px)
│   ├── Text instance (Heading 1, Raleway Bold 34px, color/text/primary)
│   └── Actions slot (HORIZONTAL, gap: 12px)  — accepts Button, max 4
└── Tabs group instance (fill: color/background/select light, radius: 8px)
    └── Tabs slot → _Tab.Button instances
```

**Props:**
| Prop | Type | Default |
|---|---|---|
| `Tabs` | boolean | `true` |
| `Breadcrumbs` | boolean | `true` |
| `Actions slot` | slot → Button | min 1, max 4 |

---

### Template / Page table

**Figma node:** `36:3634` (template component, 1600×792)

The reference composition for a standard list/table page.

```
Template / Page table (VERTICAL, padding: 12px 0)   — fill: color/background/default
├── Page header                                       — padding: 16px 32px
│   ├── Breadcrumbs (boolean)
│   ├── Heading + Actions slot
│   └── Tabs group
├── Filter group                                      — padding: 24px 32px 12px
│   ├── Top row (HORIZONTAL, gap: 16px)
│   │   ├── Filter top slot (holds input fields)
│   │   ├── Button (Primary — export/action)
│   │   └── Button (Secondary — filter/clear)
│   └── Bottom row
│       └── Filters bottom slot
└── Table region (VERTICAL, padding: 16px 32px, gap: 8px)
    └── Template / Table + pagination
        ├── Table
        └── Pagination (boolean)
```

> **Filter group** is referenced inside this template but its component definition is not in the published library — it appears as an instance only. Its slot names (`Filter top slot`, `Filters bottom slot`) are documented here from observation.

---

### Template / Table + pagination

**Figma node:** `87:8415` (template component, 1536×432)

```
Template / Table + pagination (VERTICAL, padding: 4px 0, gap: 8px)
├── Table
└── Pagination instance (boolean — toggles visibility)
```

**Props:**
| Prop | Type | Default |
|---|---|---|
| `Pagination` | boolean | `true` |

> **Pagination** component is referenced here but not defined in the published library. TBD.

---

## 5. Slot Reference

Slots are the primary composition mechanism. This table lists every named slot in the system.

| Component | Slot name | Accepts | Min | Max |
|---|---|---|---|---|
| `Page header` | `Actions slot` | `Button` (component set) | 1 | 4 |
| `Tabs group` | `Tabs slot` | `_Tab.Button` | 3 | — |
| `Table` | `Table slot` | `Table.Column` | — | — |
| `Table.Column` | `Cells Slot` | `_Table.Cell` | 2 | — |
| `Breadcrumbs` | `Pages slot` | `_Breadcrumbs.Label` | — | — |
| `Filter group` (observed) | `Filter top slot` | Input fields | — | — |
| `Filter group` (observed) | `Filters bottom slot` | Input fields | — | — |

---

## 6. Assembly Patterns

### Standard list page

```
Template / Page table
  Page header
    breadcrumbs: [Home > Section > Page]
    title: "Page title"
    actions: [Button(Primary, "Export"), Button(Secondary, "Add")]
    tabs: [Tab("All"), Tab("Active"), Tab("Archived")]
  Filter group
    top: [Input field("Search"), Dropdown("Status")]
    actions: [Button(Primary), Button(Secondary)]
  Table
    columns: [
      Table.Column(header: "Name",   cells: [..._Table.Cell(Odd/Even)]),
      Table.Column(header: "Status", cells: [...]),
      Table.Column(header: "Date",   cells: [...]),
    ]
  Pagination
```

### Table without page chrome

```
Template / Table + pagination
  Table
    columns: [...]
  Pagination (optional)
```

---

## 7. Known Gaps (TBD)

| Gap | Notes |
|---|---|
| **Spacing tokens** | No spacing/sizing variables. Values are hardcoded in component layouts. |
| **Text styles** | No Figma text styles defined. Two font families (Raleway, Open Sans) with no formal scale. |
| **Dark mode** | Single mode only. Token naming is ready for a second mode but none exists. |
| **Collection naming** | Variable collection is called "Collection 1" — needs a proper name before any code export. |
| **Typo in token names** | `backgorund` should be `background` across all background tokens. Fix before pipeline. |
| **Pagination** | Referenced in templates, not published as a component. |
| **Filter group** | Observed in template, not published as a standalone component. |
| **Dropdown** | `_Dropdown.Button` exists but no public `Dropdown` component is published. |
| **Breadcrumbs** | `_Breadcrumbs.Label` exists; no public `Breadcrumbs` molecule. |
| **Error / validation states** | No error color tokens, no error state on `Input field`. |
| **Focus / disabled states** | No focus ring or disabled variant on any component. |
| **Semantic status colors** | No success, warning, or error palette tokens. |
| **Icon library** | Three standalone icon components (share, download, settings). No systematic icon set. |
| **Motion / animation** | No transition or animation tokens or specs. |

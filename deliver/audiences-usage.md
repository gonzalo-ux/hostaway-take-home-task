# Design Language — Usage by Audience

> This document is a companion to [design-language.md](./design-language.md). It assumes you have read that file and explains how three audiences interact with the system: designers composing new work, engineers implementing it, and AI tooling picking up and extending it reliably.

---

## Audience 1 — Designers

### Mental model

The system has three tiers you work in simultaneously:

```
Tokens  →  Components  →  Templates
(variables)  (what you place)  (starting points)
```

Tokens are invisible contracts. You never see them in the canvas — but every color, every fill, every text color in a component is bound to one. When the token changes, the component updates everywhere. Never override a component's color directly; if you need a different color, the token is wrong, not the component.

Components split into **public** (place these) and **private** (prefixed with `_` — never place these directly). If you find yourself placing a `_Tab.Button` on its own, you're working at the wrong level.

Templates are not frames — they are **components**. Detaching a template to customise it defeats the system. Instead, use the template's exposed props and slots, and compose new sections from existing components beneath it.

### How to start a new page

> **Example using `Template / Page table`.** The same workflow applies to any template: place it, configure its props to show or hide sections, then populate its slots with the appropriate component instances.

1. **Place `Template / Page table`** from the Components panel. This gives you the full chrome: breadcrumbs, heading, tabs, filters, and a table scaffold.
2. **Use the Props panel** to toggle what you need: hide `Breadcrumbs` if this is a top-level page, hide `Pagination` if the data is short.
3. **Populate the `Actions slot`** in Page header with Button instances (`Type=Primary` for the main action, `Type=Secondary` for secondary). Max 4 buttons.
4. **Populate `Tabs slot`** with `_Tab.Button` instances, min 3. Set one to `Type=Selected`, the rest to `Type=Idle`.
5. **Fill table columns** by placing `Table.Column` instances into the `Table slot`. Each column gets its own header label and a `Cells Slot` of alternating `_Table.Cell` (Odd/Even) instances.
6. **Leave text content in cells as real data**, not placeholder text. The component sets the style; you set the content.

### How to extend an existing component

The general pattern for any slot-driven component: find the slot in the instance (not the master), duplicate an existing child, and update its props. Never re-enter the master component to make per-page edits.

**Example — adding a new tab option to a `Tabs group`:**
- Open the `Tabs group` instance on your page (not the master component).
- Click into the `Tabs slot` and duplicate an existing `_Tab.Button`.
- Change its `Label` prop, set it to `Type=Idle`.
- Do not increase the slot count beyond what the `Tabs group` container width supports.

**Example — adding a new column to a `Table`:**
- Click into the `Table slot` inside a `Table` instance.
- Duplicate an existing `Table.Column`.
- Update the header label via `_Table.Header.Cell` > `Header cell` text prop.
- Update cell content inside each `_Table.Cell` via the `Cell label` text prop.

**Example — changing a `Button` label:**
- Select any `Button` instance. Use the Props panel — find `Label` — change the text there. The prop is what controls the value.

### How to create a new component (contributing to the library)

The contribution process has two tracks — see [migration-roadmap.md Phase 1](./migration-roadmap.md#phase-1--build-the-infrastructure-weeks-310) for the full model. In brief:

- **Track A (DS-led):** New core components. Full contract before publish — anatomy, slot/prop table, interactive states, variant budget, one do/don't, owner and version. Async review via Figma branch, no meeting required.
- **Track B (Squad-contributed):** New patterns that compose from existing Core components. DS lead reviews and merges within 48 hours, or returns with specific feedback.

Every contribution runs through the `figma-atomic-audit` tool before merge. DS lead sees a score, not a checklist.

Before building from scratch, check: does the concept already exist as a private `_` part?

1. **Build from existing atoms.** A new form field molecule should use `_Input.Field` and `_Input.Label` as its internals, not redraw them.
2. **Bind every fill and stroke to a semantic token.** Select the layer, open Variables in the right panel, and bind fills to `color/background/*`, text to `color/text/*`, borders to `color/border/*`.
3. **Replace repeated content with slots.** If your component repeats a pattern (list items, action buttons), define a slot so the parent decides the count.
4. **Name private parts with `_`** if they should not be used standalone. Name them `ComponentName.PartName` so their origin is clear in the layers panel.
5. **Write the component description** in Figma's description field: what it is, when to use it, and which components it accepts in its slots.

### Token discipline

Never apply a palette primitive directly to a layer (e.g. don't bind a fill directly to `color/palette/green/500`). Always go through a semantic token (`color/background/primary`). This is what allows theme changes and future dark mode to work without touching components.

When you find a fill that isn't variable-bound (no purple/green diamond in the fill picker), that is a defect. File it or fix it. Once the automated audit is running (see [migration-roadmap.md Phase 1 — Token pipeline](./migration-roadmap.md#token-pipeline)), a lint rule will catch hardcoded fills automatically on every published component.

### What's off-limits

- Do not detach component instances to make one-off edits. Override via props and slots first; if the system can't accommodate the use case, the system needs extending, not detaching.
- Do not add new variants to existing component sets without going through the contribution process. Variants that encode content (e.g. `Label="Reservations"`) are not variants — use the text prop.
- Do not create your own Button using rectangles and text. The `Button` component set exists. Use it.

---

## Audience 2 — Engineers

### Mental model: Figma names map to implementation names

The system is designed so that what you see in Figma is what you write in code. Component names, prop names, slot names, and token names are the shared vocabulary. There is no translation layer — if you find yourself renaming things to make them work in code, that is a parity gap worth flagging.

### Token-to-CSS mapping

Variables in Figma map directly to CSS custom properties. The naming convention follows the Figma path with `/` converted to `-`:

```
Figma variable              CSS custom property
───────────────────         ────────────────────────────────
color/background/default    --color-background-default
color/background/primary    --color-background-primary
color/text/primary          --color-text-primary
color/text/interactive      --color-text-interactive
color/border/secondary      --color-border-secondary
color/icon/primary          --color-icon-primary
```

**Full resolved token values for implementation:**

```css
:root {
  /* Palette — do not use directly */
  --palette-green-100:     #F8FBFC;
  --palette-green-200:     #DFECEC;
  --palette-green-300:     #CCE6E6;
  --palette-green-500:     #44A29F;
  --palette-green-700:     #237472;
  --palette-neutral-50:    #F9FAFB;
  --palette-neutral-100:   #F3F4F6;
  --palette-neutral-200:   #E5E7EB;
  --palette-neutral-300:   #D1D5DC;
  --palette-neutral-400:   #99A1AF;
  --palette-neutral-500:   #6A7282;
  --palette-neutral-600:   #4A5565;
  --palette-neutral-700:   #364153;
  --palette-neutral-800:   #1E2939;
  --palette-neutral-900:   #101828;

  /* Semantic — use these in components */
  --color-background-default:       #FFFFFF;
  --color-background-primary:       #44A29F;
  --color-background-secondary:     #CCE6E6;
  --color-background-light:         #F3F4F6;
  --color-background-lighter:       #F9FAFB;
  --color-background-select:        #DFECEC;
  --color-background-select-light:  #F8FBFC;
  --color-background-overlay:       rgba(0,0,0,0.5);
  --color-text-primary:             #31343A;
  --color-text-secondary:           #676D7A;
  --color-text-interactive:         #237472;
  --color-text-inverse:             #FFFFFF;
  --color-border-primary:           #101828;
  --color-border-secondary:         #E5E7EB;
  --color-border-tertiary:          #F3F4F6;
  --color-icon-primary:             #101828;
  --color-icon-interactive:         #237472;
  --color-icon-inverse:             #FFFFFF;
}
```

### Component-to-code mapping

Each Figma component maps to a single code component. Figma props map to component props. Figma slots map to `children` or named slots/render props.

> **Note on design-to-code linking:** The examples below are hand-authored mappings. See [Design-to-code linking](#design-to-code-linking) for the planned approach to making these mappings machine-readable and maintainable.

> **Examples using `Button`, `Tabs group`, `Input field`, `Table`, and `Page header`.** Apply the same prop/slot/style pattern to any component in the inventory.

#### Button

```
Figma: Button (Type=Primary|Secondary|Ghost, Label, Prefix, Icon)

Props:
  variant:  'primary' | 'secondary' | 'ghost'    // Type
  label:    string                                // Label
  prefix?:  boolean                               // Prefix (show icon)
  icon?:    ReactNode                             // Icon (instance swap)

Styles — Primary:
  background:    var(--color-background-primary)
  color:         var(--color-text-inverse)
  border-radius: 999px
  padding:       12px 16px
  gap:           4px
  font-family:   Open Sans
  font-weight:   600
  font-size:     16px
```

#### Tabs group

```
Figma: Tabs group (Width=Auto|Full, Tabs slot → _Tab.Button)

Props:
  width?:    'auto' | 'full'             // Width variant
  children:  ReactNode                   // Tabs slot (min 3 _Tab.Button)

Container styles:
  background:    var(--color-background-select-light)
  border-radius: 8px
  padding:       4px
  gap:           24px

Tab button (idle):
  background:    var(--color-background-select-light)
  border-radius: 4px
  padding:       8px 12px
  font:          Open Sans SemiBold 14px
  color:         var(--color-text-interactive)

Tab button (selected):
  background:    var(--color-background-select)
```

#### Input field

```
Figma: Input field (label + _Input.Field)

Props:
  label:       string       // _Input.Label text
  placeholder? string       // _Input.Text field
  prefix?:     boolean      // icon prefix toggle
  icon?:       ReactNode    // icon instance

Outer layout:    vertical, gap 4px
Label:           Open Sans Medium 14px, color/text/primary
Field container: border 1px var(--color-border-secondary), radius 4px, padding 12px, gap 8px
Field text:      Open Sans Medium 14px, color/text/primary
```

#### Table

The table is column-driven. Each `Table.Column` is an independent vertical unit stacked horizontally inside the Table container. Do not map to a row-driven `<tr>/<td>` model directly — the Figma structure is column-first.

```
Figma: Table → Table.Column[] → (_Table.Header.Cell, _Table.Cell[])

Recommended code structure:
  <Table>                            // border: 2px var(--color-border-tertiary), radius 8px
    <TableColumn>                    // vertical, no padding
      <TableHeaderCell               // background: color/background/light, padding 16px 12px
        label="Name"
        sortable
      />
      <TableCell row={0} />          // Type=Odd  → background: color/background/default
      <TableCell row={1} />          // Type=Even → background: color/background/lighter
      ...
    </TableColumn>
  </Table>

_Table.Cell variants:
  Odd   + Interactive=False  →  background: #FFFFFF
  Even  + Interactive=False  →  background: #F9FAFB
  Odd   + Interactive=True   →  (hover/active state — TBD in design)
  Even  + Interactive=True   →  (hover/active state — TBD in design)

Cell layout: horizontal, padding 16px 12px, gap 4px
Cell text:   Open Sans Regular 14px, color/text/primary
Header text: Open Sans Bold 14px, color/text/primary
```

#### Page header

```
Figma: Page header (Tabs boolean, Breadcrumbs boolean, Actions slot)

Props:
  title:         string
  breadcrumbs?:  BreadcrumbItem[]    // Breadcrumbs boolean + Pages slot
  tabs?:         TabItem[]           // Tabs boolean + Tabs slot
  actions?:      ReactNode           // Actions slot (max 4 Button)

Outer layout:  vertical, padding 16px 32px, gap 28px
Title:         Open Sans Bold 34px, color/text/primary
Actions gap:   12px between buttons
```

### Slots as children/render props

Every Figma slot maps to a React `children` prop or a named render prop. The slot constraints (min/max children, accepted component type) become PropTypes or TypeScript type constraints:

| Figma slot | Code equivalent | Constraint |
|---|---|---|
| `Actions slot` | `actions?: ReactNode` | max 4, `<Button>` only |
| `Tabs slot` | `children: ReactNode` | min 3, `<TabButton>` only |
| `Table slot` | `children: ReactNode` | `<TableColumn>` only |
| `Cells Slot` | `children: ReactNode` | min 2, `<TableCell>` only |
| `Pages slot` | `breadcrumbs: BreadcrumbItem[]` | `<BreadcrumbLabel>` |

### Typography implementation note

One font family throughout — Open Sans. Load it once:

```css
/* Open Sans — all UI text */
@import url('https://fonts.googleapis.com/css2?family=Open+Sans:wght@400;500;600;700&display=swap');
```

### Design-to-code linking

The hand-authored mappings above will not scale as the component library grows. Two approaches are under consideration to make the Figma → code connection machine-readable:

**Option A — Figma Code Connect**
Figma's native [Code Connect](https://www.figma.com/developers/code-connect) lets you attach a code snippet or component import directly to a Figma component. When a designer inspects a component in Dev Mode, they see the real import and prop usage instead of autogenerated CSS. This is the preferred path if the engineering stack is React or another framework Code Connect supports, because it requires no custom infrastructure and stays in sync with Figma's own versioning.

**Option B — Custom mapping layer**
If Code Connect doesn't cover the stack (e.g. a non-supported framework, a proprietary component system, or tooling that needs the mapping at build time), a custom solution would maintain a structured mapping file (e.g. JSON or YAML) that links Figma node IDs to code component paths, prop names, and slot-to-children contracts. This file would be consumed by design-system tooling, Storybook integrations, or AI tools to generate consistent import statements and prop usage without manual lookup.

**Current state:** TBD. The mappings in this section are the source of truth until one of these approaches is implemented. When linking is in place, this section will point to the integration docs instead.

### Contributing a code component

When a Figma component is rebuilt and ready to ship, the engineering side of the contribution follows this checklist — see [migration-roadmap.md Phase 2 — Per-component workflow](./migration-roadmap.md#phase-2--rebuild-and-migrate-weeks-724) for the full process:

1. Engineer reviews the slot/prop contract async before Figma branch merges.
2. Component ships under a single import path (`@hostaway/ui` or equivalent) — enforced by ESLint from day one. No component-specific ad-hoc imports.
3. Storybook stories mirror Figma slot examples. Stories are the acceptance criteria for the rebuild — if a slot state isn't in Storybook, it's not done.
4. Definition of done: "uses DS component, or has a linked note explaining why not."
5. After merge, mark the old Figma asset Deprecated with "Use X instead" in its description.
6. Squad validates on one real screen before the old asset is removed.

### What is not in the system yet

These are known gaps documented in `design-language.md`. Do not implement fallbacks or invent values — flag them as pending design. The [migration-roadmap.md Phase 2 rebuild sequence](./migration-roadmap.md#rebuild-sequence) defines when each layer (atoms → molecules → organisms) is scheduled.

- Error / validation states on Input field
- Focus styles (keyboard navigation)
- Disabled states on Button, Input, Tab
- Hover state on interactive table cells
- Pagination component internals
- Filter group component internals
- Spacing and sizing tokens (use pixel values from design-language.md 2.3 until tokens exist)

---

## Audience 3 — AI Tooling

This section documents how to use `design-language.md` as a reliable input for AI-assisted design or code generation. The goal is deterministic output: given a task, an AI tool should produce work that is consistent with the existing system without inventing values.

### How to read this system

When given a design or implementation task, an AI tool should resolve in this order:

1. **Check the component inventory first** (3 in design-language.md). If a component exists, use it — do not recreate it.
2. **Check the slot reference** (5). If a composition point exists as a slot, populate it — do not wrap components in arbitrary containers.
3. **Resolve colors through semantic tokens** (2.2). Never use a hex value directly unless you are defining the token itself. Always go `task → semantic token → hex`.
4. **Check the Known Gaps list** (7) before generating. If the task requires something in that list, surface the gap rather than inventing a value.

### Component lookup protocol

When a user asks to build or modify a UI element:

```
1. Does a component match the element name exactly?
   → Yes: use it, configure its props
   → No: does a Template match the page pattern?
       → Yes: use the template, populate its slots
       → No: compose from existing public components

2. Is the element a sub-part of a known component?
   → Check for a _ prefixed component set
   → If found: work through the parent component, not the private part

3. Is the required state/variant missing?
   → Check the Known Gaps list
   → If listed as TBD: report the gap, do not invent a value
```

### Token resolution protocol

When generating CSS, a style object, or a design token value:

```
1. Map the visual property to a semantic category:
   fill/background  → color/background/*
   text             → color/text/*
   border/stroke    → color/border/*
   icon             → color/icon/*

2. Choose the right semantic token for the context:
   default surface    → color/background/default
   brand action       → color/background/primary
   secondary action   → color/background/secondary
   selected state     → color/background/select
   body text          → color/text/primary
   supporting text    → color/text/secondary
   interactive/link   → color/text/interactive
   text on dark fill  → color/text/inverse
   input border       → color/border/secondary
   table border       → color/border/tertiary

3. Resolve to hex using the token table in 2.2
```

### Typography rules for generation

```
When generating text styles:
  - All text → Open Sans
  - Do not use any other font family unless explicitly adding to the system

Font weight mapping:
  Open Sans Bold     → font-weight: 700  (Heading 1, table header)
  Open Sans SemiBold → font-weight: 600  (Button, Tab)
  Open Sans Medium   → font-weight: 500  (Input label, Input text)
  Open Sans Regular  → font-weight: 400  (Table cell)

No text style tokens exist yet → use raw values from 2.4 of design-language.md
```

### Slot population rules

When an AI tool generates content for a slot:

```
Actions slot (Page header):
  - Use Button component instances only
  - First action: Type=Primary
  - Subsequent actions: Type=Secondary or Type=Ghost
  - Maximum 4 buttons

Tabs slot (Tabs group):
  - Use _Tab.Button instances only
  - Exactly one must have Type=Selected
  - All others must have Type=Idle
  - Minimum 3 tabs

Cells Slot (Table.Column):
  - Alternate Type=Odd and Type=Even in order
  - Minimum 2 cells
  - Cell labels are real content, not "Cell label" placeholder
```

### Assembly pattern for a new page

When asked to generate a new list or data page, use the appropriate template from the inventory. The example below uses `Template / Page table` — the same slot-populate logic applies to any template.

```
Template / Page table   ← replace with the matching template for the page type
├── title: [page title]
├── breadcrumbs: [section > page path]
├── tabs: [primary filter options, first selected]
├── actions: [primary CTA as Button=Primary, optional secondary as Button=Secondary]
├── filters: [search input + status dropdown minimum]
└── table columns: [header + alternating cells for each data field]
```

If any slot constraint cannot be met (e.g. a table with only 1 column, or no tabs needed), surface this as a question before generating — the template enforces those constraints for a reason.

### How to detect a system violation

An AI tool reading generated output can check for these signals that indicate something is out of system:

| Signal | What it means |
|---|---|
| Hex color not in the token table | Hardcoded value — replace with semantic token |
| Font family other than Open Sans | Outside the type system |
| `_`-prefixed component used as a top-level element | Using a private part directly |
| A `Button` built from scratch (rect + text) | Component exists — use it |
| A color variant not in `Button` (Primary/Secondary/Ghost) | Either wrong variant or missing state |
| A slot populated with a non-preferred component type | Slot contract violation |

### Prompt patterns that work well with this system

These phrasings help AI tools stay in-system:

- "Build a page using `Template / Page table` with tabs [A, B, C] and columns [Name, Status, Date]"
- "Add a secondary action button to the `Actions slot` on the page header"
- "Generate the CSS for a `Button` with `Type=Primary` using the Hostaway token system"
- "What token should I use for the background of a selected tab?"
- "Show the assembly of a `Table.Column` with 6 rows of alternating cells"

Avoid:
- "Make a button with a teal background" (specify the component and variant instead)
- "Use a slightly different shade of green for hover" (no hover tokens exist; surface as a gap)
- "Create a custom tab bar" (use `Tabs group` with `_Tab.Button` slots)

### Extending the system (AI-assisted contribution)

When an AI tool is asked to design something new that the current system cannot express:

1. **Check 7 (Known Gaps)** — if the gap is listed, the answer is "this is a known TBD, propose a solution to the DS lead rather than inventing one."
2. **If a new token is needed**, follow the existing naming convention: `color/{category}/{role}` → alias to `color/palette/{family}/{step}`.
3. **If a new component is needed**, follow the `_Component.Part` / `Component` / `Template` naming tiers, bind all fills to tokens, and document its slot contract.
4. **Output new components as a proposal** (description + prop table + assembly diagram) rather than silently adding them to the canvas. The contribution process requires review.

---

## Cross-Reference

| Task | Primary reference |
|---|---|
| Which color token to use | [design-language.md 2.2](./design-language.md#22-semantic-color-tokens) |
| What components exist | [design-language.md 3](./design-language.md#3-component-inventory) |
| How a specific component is assembled | [design-language.md 4](./design-language.md#4-component-specs) |
| Which slots accept what | [design-language.md 5](./design-language.md#5-slot-reference) |
| What's not built yet | [design-language.md 7](./design-language.md#7-known-gaps-tbd) |
| How to extend the system | This file, Designers or AI Tooling |
| Contribution process (Figma tracks, review SLAs, audit gate) | [migration-roadmap.md Phase 1](./migration-roadmap.md#phase-1--build-the-infrastructure-weeks-310) |
| When gaps will be filled (rebuild sequence, migration waves) | [migration-roadmap.md Phase 2](./migration-roadmap.md#phase-2--rebuild-and-migrate-weeks-724) |
| Token pipeline and lint rules | [migration-roadmap.md Token pipeline](./migration-roadmap.md#token-pipeline) |
| Success metrics and checkpoints | [migration-roadmap.md Metrics](./migration-roadmap.md#metrics) |

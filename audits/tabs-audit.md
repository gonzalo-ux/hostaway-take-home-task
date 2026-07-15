# Component Audit — Horizontal Tabs (Untitled UI PRO VARIABLES v8.0)

**Subject:** [Horizontal tabs — Figma preview](https://www.figma.com/design/QERVV4a2Fpa1FmsZ5LGW3S/%E2%9D%96-PREVIEW-%E2%9D%96-Untitled-UI-Figma-%E2%80%93-PRO-VARIABLES--v8.0-?node-id=1118-69893)  
**Related:** [Vertical tabs](https://www.untitledui.com/components/vertical-tabs) (2 components + 264 variants)  
**Audit lens:** Design system architecture, Figma ↔ code parity, variant governance  
**Date:** July 2026  
**Parent audit:** [Audit summary](../deliver/audit-summary.md)  
**Methodology:** Published inventory cross-referenced against open-source React implementation ([`tabs.tsx`](https://github.com/untitleduico/react/blob/main/components/application/tabs/tabs.tsx), [`tabs.demo.tsx`](https://github.com/untitleduico/react/blob/main/components/application/tabs/tabs.demo.tsx)). Live Figma node inspection blocked — preview file disables copy; REST/Desktop Bridge auth unavailable.

---

## Executive summary

**Horizontal tabs** is a representative Untitled UI application component: visually polished, combinatorially over-varianted in Figma, and **architecturally cleaner in code than in design**.

| Surface | Inventory | Assessment |
|---|---|---|
| Figma | 2 components + 280 variants | Monolithic variant matrix; copy disabled in preview |
| React | 1 component, 5 horizontal `type` values | Composable props; open source and copyable |

**Verdict:** Do not lift Horizontal tabs wholesale into a product design system. Extract the **visual language** (5 style treatments, 2 sizes, underline vs button chrome) and rebuild as a single `Tabs` component with a **≤16 variant budget** and explicit per-item slots for icon, badge, and label.

The 280-variant Figma model encodes **content and layout permutations as variants** — the exact anti-pattern called out in the parent audit. The React library already demonstrates the correct decomposition; Figma and code are **not aligned**.

---

## 1. Component inventory

### 1.1 Figma (Application UI)

Per [Untitled UI — Horizontal tabs](https://www.untitledui.com/components/horizontal-tabs):

- **Category:** Application UI / Dashboard components
- **Count:** 2 components, 280 variants
- **Node (preview):** `1118:69893`
- **Access:** Preview file — layer copy disabled; extraction requires licensed kit

Horizontal tabs sits alongside **Vertical tabs** (2 components, 264 variants) as a sibling set. Together they form a 544-variant tab family split across orientation rather than unified under one component with an `orientation` prop — a taxonomy choice that doubles maintenance surface.

### 1.2 React (Application UI)

Per [Untitled UI React — Tabs](https://www.untitledui.com/react/components/tabs):

- **Location:** `components/application/tabs/`
- **License:** Free, open source ([MIT](https://github.com/untitleduico/react))
- **Install:** `npx untitledui@latest add tabs`
- **Stack:** React Aria (accessibility), Tailwind CSS v4, TypeScript

**Compound component API:**

```
Tabs
├── Tabs.List    — container; type, size, orientation, fullWidth
├── Tabs.Item    — tab trigger; label, icon, badge
└── Tabs.Panel   — content region
```

---

## 2. Variant matrix — where 280 variants come from

### 2.1 React API (actual contract)

The React implementation exposes a **small, orthogonal prop space**:

**List-level (`Tabs.List`):**

| Prop | Values | Default |
|---|---|---|
| `type` | `button-brand`, `button-gray`, `button-border`, `button-minimal`, `underline` | `button-brand` |
| `size` | `sm`, `md` | `sm` |
| `orientation` | `horizontal`, `vertical` | `horizontal` |
| `fullWidth` | `true`, `false` | `false` |

**Item-level (`Tabs.Item`):**

| Prop | Type | Notes |
|---|---|---|
| `label` | `ReactNode` | Tab text |
| `icon` | Component or element | Leading icon |
| `badge` | `number` \| `string` | Trailing count; color adapts to `type` |
| `children` | Render prop | Alternative to `label` |

**Horizontal-only styles** (from `tabs.tsx`):

| `type` | Visual treatment |
|---|---|
| `button-brand` | Brand-filled selected/hover state |
| `button-gray` | Neutral gray selected/hover state |
| `button-border` | Segmented control in bordered container |
| `button-minimal` | Minimal pill in subtle container |
| `underline` | Bottom border indicator; container has divider line |

**Theoretical horizontal matrix (structure only):**

```
5 types × 2 sizes × 2 fullWidth = 20 list configurations
× per-item permutations (icon on/off, badge on/off, tab count) = variant explosion in Figma
```

Figma's 280 variants likely materialize **content permutations** (number of tabs, icon presence, badge presence, sample labels) as variant axes rather than as instance-level properties or slots.

### 2.2 Figma vs React — parity gap

| Concern | Figma (inferred) | React | Parity |
|---|---|---|---|
| Style treatment | Variant axis or separate components | `type` prop | ⚠️ Partial |
| Size | Variant axis | `size` prop | ⚠️ Partial |
| Full width | Separate component or variant | `fullWidth` prop | ⚠️ Partial |
| Icon per tab | Baked into variants / instance swap | `icon` prop on item | ❌ Misaligned |
| Badge per tab | Baked into variants | `badge` prop on item | ❌ Misaligned |
| Tab count | Fixed in variant examples | Dynamic `items` array | ❌ Misaligned |
| Orientation | Separate file section (Vertical tabs) | `orientation` prop | ❌ Split in Figma |
| Selected state | Interactive component / variant | React Aria `selectedKey` | ⚠️ Partial |

**Diagnosis:** Engineering receives a **prop-based, dynamic API**. Design receives a **280-variant picker**. Handoff requires mental translation — or detachment.

---

## 3. Architectural diagnosis

### 3.1 Wrong granularity — application pattern, not primitive

Tabs are navigation chrome — reusable across settings, detail views, and dashboards. In atomic design terms, tabs are a **molecule** (list + items + indicator), not an organism.

Untitled UI classifies Horizontal tabs under **Application UI** with 280 variants — organism-scale complexity for molecule-scale scope. This matches the parent audit's finding: *variant explosion at the base/application boundary with no clean compositional layer.*

### 3.2 Content encoded as variants

Sample tab labels in demos (`My details`, `Profile`, `Password`, `Team`, `Notifications`, `Integrations`, `API`) appear across both Figma examples and React demos. In Figma, permutations of:

- Tab count (3, 4, 5, 6, 7…)
- Icon on leading tab only vs all tabs vs none
- Badge on one tab vs none
- Full-width vs inline

…are likely **separate variants** rather than designer-controlled instance properties.

**Rule (from parent audit):** If a variant encodes **content**, it's not a variant — it's an example.

Horizontal tabs violates this rule at scale.

### 3.3 Instance-swap composition without slot schema

Untitled UI favors instance swap over Figma slots. For tabs, implicit swap targets likely include:

- Leading icon instance
- Badge instance
- (Possibly) trailing action

Without documented slot names, engineers cannot map:

```
Figma swap target  →  React prop
???                →  icon
???                →  badge
```

The React source **is** the slot schema — but it lives in code, not in Figma documentation.

### 3.4 Split across Horizontal and Vertical

Figma maintains **separate component sets** for horizontal (280) and vertical (264) tabs. React unifies them:

```tsx
<Tabs.List type="underline" orientation="horizontal" />
<Tabs.List type="line" orientation="vertical" />
```

Note: vertical uses `line` instead of `underline` — a subtle API asymmetry not obvious from Figma folder names alone.

| Orientation | Primary indicator style (React) |
|---|---|
| Horizontal | `underline` (bottom border) |
| Vertical | `line` (left border) |

Product DS should expose `orientation` on one component, not fork libraries by layout direction.

### 3.5 Interactive component assumptions

Figma interactive components handle default/hover/selected/focus states across the variant matrix. Real products also need:

- Disabled tabs
- Overflow / scroll for many tabs
- Dynamic add/remove tabs
- Route-driven selection (URL sync)
- Keyboard order with mixed icon+badge items

None of these are evident in the published variant inventory. React delegates interaction to React Aria; Figma models a subset visually.

---

## 4. Token and styling analysis (React source)

From [`tabs.tsx`](https://github.com/untitleduico/react/blob/main/components/application/tabs/tabs.tsx):

### 4.1 Semantic token usage (code)

React tabs bind to **semantic Tailwind tokens**, not raw Untitled primitives:

- `text-brand-secondary`, `bg-brand-primary_alt` — selected brand tabs
- `text-secondary`, `bg-primary_hover` — gray variant
- `bg-secondary_alt`, `ring-secondary` — container chrome (border/minimal types)
- `border-fg-brand-primary_alt` — underline/line indicator
- `text-quaternary` — default unselected label
- `outline-focus-ring` — focus visible

This is **more mature than typical Figma primitive binding** — the code path already uses a semantic layer.

### 4.2 Size tokens

| Size | Typography | Icon | Padding (button types) |
|---|---|---|---|
| `sm` | `text-sm font-semibold` | 16px (`size-4`) | `py-2 px-2.5` |
| `md` | `text-md font-semibold` | 20px (`size-5`) | `py-2.5 px-2.5` |

Underline type uses distinct vertical padding (`pb-2.5 pt-0`) — size affects layout differently per `type`. A product DS must document **per-style size specs**, not one global size ramp.

### 4.3 Badge color logic

Badge appearance depends on tab `type`:

```tsx
const showPillColorBadge = type === "underline" || type === "line" || type === "button-brand";
```

Selected-state badge styling is **contextual**, not a standalone Badge variant pick. Lifting Figma badge instances independently will break this coupling.

---

## 5. Accessibility

### 5.1 React (documented)

Per [React tabs docs](https://www.untitledui.com/react/components/tabs):

- Keyboard: arrow keys to navigate, Enter/Space to select
- ARIA attributes via React Aria
- Focus management and screen reader compatibility claimed

React Aria provides `role="tablist"`, `role="tab"`, `role="tabpanel"`, `aria-selected`, roving tabindex — industry-standard behavior.

### 5.2 Figma (unknown)

Preview file access prevents verification of:

- Focus ring variant coverage across all 280 variants
- `aria-*` annotation in design specs
- Disabled tab state modeling
- Overflow tab menu (not in kit)

**Gap:** Accessibility is verified in code, not per Figma variant combination. Design review on a lifted Figma organism cannot assume WCAG compliance from kit marketing claims.

---

## 6. Documentation gaps

| Area | Figma | React | Product DS need |
|---|---|---|---|
| Anatomy diagram | ❌ | ❌ | Required |
| When to use each `type` | ❌ | Partial (visual examples only) | Decision tree |
| Horizontal vs vertical | Separate pages | Unified docs | Single guideline |
| Max tab count / overflow | ❌ | ❌ | Required for product |
| Icon + badge together | ❌ | Supported in code | Do/don't |
| URL-synced tabs | ❌ | `selectedKey` documented | Pattern doc |
| Badge max value / truncation | ❌ | ❌ | Content spec |
| Mobile collapse behavior | ❌ | Native select fallback in demos | Responsive rules |

React demos include a **mobile native select fallback** (`SelectNative` hidden on `md+`) — a responsive pattern not reflected in Figma variant count. Designers following Figma alone will miss this.

---

## 7. Reuse assessment

| Asset | Reuse as product DS component? | Treatment |
|---|---|---|
| 5 horizontal visual styles | ⚠️ Partial | Keep style names; rebuild as `type` prop |
| 280 Figma variants | ❌ No | Reference only |
| React `Tabs` source | ⚠️ Partial | Fork and bind to product tokens |
| Sample tab labels/content | ❌ No | Examples only |
| Icon set (@untitledui/icons) | ⚠️ Partial | License check; may swap to product icons |
| Vertical tabs (264 variants) | ❌ No | Merge into single component with `orientation` |

**Recommended reuse path:**

1. Use React source as **API reference** (not Figma variants)
2. Rebuild Figma component with **≤16 variants** (see §9)
3. Publish **decision tree** for type selection
4. Keep Untitled Figma tabs in reference file only

---

## 8. Failure modes when lifted as an "organism"

### Early (1–2 teams)

- Designers pick visually similar tab styles inconsistently (`button-gray` vs `button-minimal`)
- Engineers implement React tabs; Figma shows different variant — drift begins

### Mid (3–5 teams)

- Detached tab bars with custom spacing break token bindings
- Badge and icon rules differ per screen (no item-level contract in Figma)
- Full-width tabs used where inline was intended (no layout prop in Figma)

### Late (5+ teams)

- Multiple forked tab components (`TabsV2`, `SettingsTabs`, `HorizontalTabsNew`)
- Dark mode breaks on detached underline dividers
- Accessibility regressions on custom detached tabs without React Aria

---

## 9. Recommendations

### 9.1 Target variant budget

| Dimension | Untitled UI Figma | Product DS target |
|---|---|---|
| Component count | 2 (horizontal) + 2 (vertical) | 1 `Tabs` |
| Total variants | 544 (horizontal + vertical) | ≤ 16 |
| Tab count | Variant-encoded | Dynamic (instance properties) |
| Icon / badge | Variant-encoded | Per-item boolean props |

**16-variant matrix (suggested):**

```
5 type × 2 size = 10
+ 2 fullWidth overrides for button-border and underline = 12
+ 2 orientation states (horizontal default, vertical) folded into type naming = within budget
+ selected/unselected handled by interactive component, not variant axis
```

Content permutations (tab count, labels, icons) become **instances**, not variants.

### 9.2 Define slot contract

| Slot | Required | Type | Notes |
|---|---|---|---|
| `list.label` | ✅ | string | Visible tab text |
| `list.icon` | ❌ | icon | Leading; 16px (sm) / 20px (md) |
| `list.badge` | ❌ | number \| string | Trailing; max 2 chars recommended |
| `panel.content` | ✅ | frame | Associated panel body |
| `list.disabled` | ❌ | boolean | Not in Untitled kit; add for product |

### 9.3 Type selection decision tree

```
Need tabs?
├── ≤5 items, settings/detail page
│   ├── Emphasis on brand → button-brand
│   ├── Neutral chrome → button-gray
│   ├── Segmented control look → button-border
│   ├── Minimal / compact → button-minimal
│   └── Classic underline → underline
├── Full-width equal tabs → any type + fullWidth
├── Side navigation → orientation=vertical, type=line
└── >5 items or mobile → consider Select or scroll overflow (not in kit)
```

### 9.4 Align Figma to React before handoff

Publish a **Figma ↔ code mapping table** in component docs:

| Figma property | React prop |
|---|---|
| Style | `Tabs.List type` |
| Size | `Tabs.List size` |
| Full width | `Tabs.List fullWidth` |
| Orientation | `Tabs.List orientation` |
| Tab label | `Tabs.Item label` |
| Tab icon | `Tabs.Item icon` |
| Tab badge | `Tabs.Item badge` |
| Selected tab | `Tabs selectedKey` |

### 9.5 Do not lift from preview file

Preview file copy is disabled by design. For Figma work:

- **Licensed kit** — duplicate editable file from [untitledui.com/figma](https://www.untitledui.com/figma)
- **Audit / reference** — use [React docs](https://www.untitledui.com/react/components/tabs) + GitHub source
- **Scratch rebuild** — one style (`underline`, `sm`) is sufficient for lint and anatomy work

---

## 10. Suggested product DS structure

```
Tabs (component)
├── Anatomy
│   ├── TabList (container)
│   ├── Tab (trigger: label + optional icon + optional badge)
│   └── TabPanel (content)
├── Props / variants
│   ├── type: brand | gray | border | minimal | underline | line
│   ├── size: sm | md
│   ├── orientation: horizontal | vertical
│   └── fullWidth: boolean
├── States (interactive)
│   ├── default, hover, focus, selected, disabled
├── Examples (not variants)
│   ├── Settings page (7 tabs, 1 badge)
│   ├── Detail view (3 tabs, icons)
│   └── Full-width (4 tabs)
└── Code: @/components/Tabs → React Aria implementation
```

---

## 11. Audit limitations

### Could not verify (blocked access)

- [ ] Node-level Figma property schema for `1118:69893`
- [ ] Exact variant axis names (which props drive 280 count)
- [ ] Variable bindings per tab type in Figma
- [ ] Interactive component state coverage across all variants
- [ ] `figma_lint_design` on Horizontal tabs component set
- [ ] Copy/paste extraction to scratch file (disabled in preview)

### Completed via alternative sources

- [x] React API and variant logic (`tabs.tsx`)
- [x] All horizontal demo permutations (`tabs.demo.tsx`)
- [x] Published inventory counts (untitledui.com/components)
- [x] Accessibility claims (React docs)

### To complete with live Figma access

1. Authenticate Figma REST or pair Desktop Bridge
2. Export component property schema for both Horizontal tab component sets
3. Count variant axes vs instance-swap depth
4. Run WCAG lint on selected/hover/focus states per type
5. Diff Figma variant names against React `type` enum

---

## 12. Bottom line

Horizontal tabs is a **case study in Figma–code misalignment**:

- **Figma:** 280 variants, 2 components, content baked into variant matrix, preview locked
- **React:** 5 types, 2 sizes, dynamic items — the API a product DS actually needs

Do not lift. **Rebuild** with the React contract as source of truth, cap variants at 16, document slots explicitly, and demote Untitled UI's tab components to a read-only reference.

The tabs family (544 variants across horizontal + vertical) should be **one governed component** in the product design system — not two untitled organism folders with half a thousand picker options.

---

*Audit prepared for Hostaway design system initiative. Part of the Untitled UI PRO VARIABLES v8.0 audit series.*

# Figma Atomic Audit — Reference

## Grading rubric

Score each criterion A–F, then derive overall grade.

| Criterion | A | B | C | D/F |
|---|---|---|---|---|
| **Atoms published & reused** | All leaf UI from library atoms | Most instanced; few raw frames | Mixed sources | Detached / duplicate primitives |
| **Molecules compose atoms** | Clear `_Prefix` pattern, published | Present but inconsistent naming | Partially raw | Molecules are monolithic |
| **Organism slot contracts** | Named slots + booleans for structure | Slots exist but gaps (minChildren, orphans) | Instance-swap only | Content variants, no slots |
| **Template composition** | Thin orchestrator, honest API | Fixed organisms, no ghost props | Orphan properties | Monolithic template |
| **Page tier separation** | Demo frames unpublished | Minor leakage | Templates published as components | Finished screens in library |
| **Naming hygiene** | Semantic layer names throughout | Generic frames in nested regions only | Multiple `Frame N` | Unreadable tree |
| **Engineering handoff** | Props map 1:1 to code slots | Mostly mappable | Requires reverse-engineering | Detach-only workflow |

**Overall grade anchors:**

- **A / A−** — Compose-don't-lift; slots over variants; library parts traceable
- **B+ / B** — Good direction; fixable API/naming gaps
- **B− / C+** — Mixed patterns; some monoliths remain
- **C or below** — UI-kit lift; detach/fork is the real workflow

---

## Output template

```markdown
# Component Audit — {Name} (Hostaway)

**Subject:** [Figma node]({figma_url})  
**Context:** [Components page]({context_url}) — if applicable  
**Audit lens:** Atomic design system architecture  
**Date:** {month year}  
**Parent audit:** [audit-summary.md](../deliver/audit-summary.md)  
**Related:** {links to related deep-dives}  
**Methodology:** Live Figma Desktop Bridge — layer tree, properties, bindings, screenshots.

---

## Executive summary

{One paragraph: what this node is, which atomic tier, verdict in one sentence.}

| Surface | Node | Tier | Assessment |
|---|---|---|---|
| {name} | `{id}` | {tier} | {grade} — {one-line note} |

**Verdict:** {Compose vs lift assessment.}

**Overall atomic design grade: {grade}**

---

## 1. Atomic tier map

{ASCII tree}

### 1.1 Components page inventory

{Table if context page audited}

---

## 2. What is built well

### 2.1 {Area}
{Evidence from live data}

---

## 3. Issues found

### 3.1 {Issue} — {FIXED | OPEN} — {High|Medium|Low}

{Description, impact, resolution if fixed}

---

## 4. Post-cleanup state (if re-audit)

{Before/after table}

---

## 5. Remaining gaps

{Optional blockers}

---

## 6. Comparison to anti-patterns

{Table vs audit-summary.md lift patterns}

---

## 7. Recommendations

### Done ✅
- {items}

### Next
- {items}

---

## 8. Scorecard

| Criterion | Score |
|---|---|
| Atoms published & reused | {grade} |
| Molecules compose atoms | {grade} |
| Organisms use slots, not content variants | {grade} |
| Template composes organisms | {grade} |
| Page tier separation | {grade} |
| Naming & API clarity | {grade} |
| Engineering handoff contract | {grade} |

**Overall: {grade}**
```

---

## MCP execute scripts

### Full structure + properties

```javascript
const node = await figma.getNodeByIdAsync('NODE_ID');
const defs = node.componentPropertyDefinitions || {};

async function walk(n, depth = 0, out = []) {
  if (!n || depth > 6) return out;
  const item = { depth, name: n.name, type: n.type, id: n.id };
  if (n.type === 'INSTANCE') item.main = (await n.getMainComponentAsync())?.name;
  if (n.componentPropertyReferences) item.refs = n.componentPropertyReferences;
  out.push(item);
  if ('children' in n) for (const c of n.children) await walk(c, depth + 1, out);
  return out;
}

const tree = await walk(node);
const slots = tree.filter(n => n.type === 'SLOT');

return {
  name: node.name,
  type: node.type,
  topLevel: node.children?.map(c => ({ name: c.name, type: c.type })),
  properties: Object.entries(defs).map(([k, v]) => ({
    key: k,
    display: k.split('#')[0],
    type: v.type,
    default: v.defaultValue,
    slotSettings: v.slotSettings || null,
  })),
  slots,
  slotPropertyKeys: Object.keys(defs).filter(k => defs[k].type === 'SLOT'),
  boundSlotNames: slots.map(s => s.name),
};
```

### Orphan property detection

An orphan SLOT property has a key in `componentPropertyDefinitions` but **no** SLOT layer in the component's own tree (not nested instances) with `componentPropertyReferences.slotContentId` pointing to that key.

```javascript
const template = await figma.getNodeByIdAsync('NODE_ID');
const defs = template.componentPropertyDefinitions || {};

async function findOwnSlots(node, depth = 0, out = []) {
  if (!node || depth > 2) return out; // direct children only for template-level check
  if (node.type === 'SLOT') out.push({ name: node.name, refs: node.componentPropertyReferences });
  if ('children' in node) for (const c of node.children) await findOwnSlots(c, depth + 1, out);
  return out;
}

const ownSlots = await findOwnSlots(template);
const slotProps = Object.keys(defs).filter(k => defs[k].type === 'SLOT');

return {
  slotProperties: slotProps.map(k => k.split('#')[0]),
  ownSlotLayers: ownSlots,
  orphans: slotProps.filter(key => {
    const id = key.split('#')[1];
    return !ownSlots.some(s => s.refs?.slotContentId?.includes(id) || s.refs?.slotContentId === key);
  }),
};
```

### Components page inventory

```javascript
const page = await figma.getNodeByIdAsync('PAGE_NODE_ID');
return page.children
  .filter(c => ['COMPONENT', 'COMPONENT_SET', 'SECTION'].includes(c.type))
  .map(c => ({
    id: c.id,
    name: c.name,
    type: c.type,
    childCount: 'children' in c ? c.children.length : 0,
  }));
```

---

## Figma URL → node ID

| URL segment | MCP nodeId |
|---|---|
| `node-id=36-3634` | `36:3634` |
| `node-id=56-9823` | `56:9823` |

---

## File naming

| Node type | Filename pattern |
|---|---|
| Single organism | `{kebab-name}-audit.md` e.g. `filter-group-audit.md` |
| Template + page mock | `page-mock-audit.md` or `{template-name}-audit.md` |
| Atom/set | `{name}-audit.md` e.g. `untitled-ui-tabs-audit.md` |

---

## Bridge pairing

If MCP returns auth/connection errors:

1. User opens Figma Desktop with target file
2. Run `figma_get_status` — if disconnected, ask user to open the Desktop Bridge plugin
3. Retry fetch — pairing is per-session

Do not fall back to browser-only inference for property/binding audits.

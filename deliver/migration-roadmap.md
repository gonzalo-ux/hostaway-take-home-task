# Design System Migration Roadmap

**Horizon:** July – December 2026  
**Team:** 1 DS lead + 6 part-time contributing designers + engineering partners  
**Starting point:** Lifted Untitled UI kit; one validated pilot (Page header + Filter group + Table, graded A−)  
**Goal:** A lean, owned design system the whole product team actually uses.

---

## The plan

```
Phase 0  │ Weeks 1–2   │ Understand before fixing
Phase 1  │ Weeks 3–10  │ Build the infrastructure, not components
Phase 2  │ Weeks 7–24  │ Rebuild, migrate, retire (overlaps Phase 1)
```

---

## Phases timeline at a glance

```
Jul 2026    Aug 2026    Sep 2026    Oct 2026    Nov 2026    Dec 2026
│           │           │           │           │           │
├── Ph 0 ───┤
│  Inventory│
│  + survey │
│           ├─── Ph 1 (infrastructure) ──────────────────────┤
│           │  Library split, tokens, contribution model       │
│           │  Automated audit + weekly lint in place          │
│           │           │
│           │     ├──── Ph 2 (rebuild + migrate) ─────────────┤
│           │     │  Atoms → Molecules → Organisms → Templates │
│           │     │  Wave 0 → Wave 1 → Wave 2 → Wave 3         │
│           │     │           │
│           │     │           ├── 3-month checkpoint ──────────►
│           │     │                Oct: Atoms + 1–2 organisms done
│           │     │                     1 squad on Wave 1
│           │     │                     Automated lint live
│           │     │
└───────────┴─────┴───────────────────────────────────────────►
                                                    Dec: See below
```
---

## Phase 0 — Understand before fixing (Weeks 1–2)

Replace assumptions with evidence before touching a single component.

| Activity | Who | Output |
| ----- | ----- | ----- |
| Figma scan — component tiers, variant counts, detach rate in last 20–30 screens | DS lead (automated, 1 day) | Issues flagged per component |
| Code inventory — which components are imported from kit vs built in-house, shadow implementations, token sources | Eng partner (1 day) | Import map + duplicate list |
| Designer survey — which flows, which components, where you detach most | 6 designers (30 min async) | Pain-point ranking |

**Outcome:** A migration registry — every published component with its atomic tier, variant count, detach rate, code equivalent, and a verdict: **Keep / Rebuild / Deprecate**. This stays live for 6 months.

> **Checkpoint (end of week 2):** Present inventory to stakeholders. Adjust scope before Phase 1 begins.

---

## Phase 1 — Build the infrastructure (Weeks 3–10)

### Library structure — three files

| File | Contents | Published | Versioned |
| ----- | ----- | ----- | ----- |
| **Core** | Tokens, icons, atoms, molecules, organisms, templates | Yes | Yes |
| **Patterns** | Usage examples for cross-component flows (error handling, onboarding, etc.) | Yes | Yes |
| **Documentation** | Component specs, page examples, dos & don'ts, accessibility guidance | Yes | No |

### Contribution model — two tracks

**Track A — DS-led** (new core components and patterns)  
Full contract before publish: anatomy, slot/prop table, interactive states, variant budget, one do/don't, owner and version. Async review via Figma branch — no meeting required.

**Track B — Squad-contributed** (patterns within a flow)  
Any pattern that composes from existing Core components. DS lead reviews, doesn't approve from scratch. Merged in 48 hours or returned with feedback.

Every contribution runs through the `figma-atomic-audit` tool: variant encoding, slot usage, orphan properties, hardcoded fills. DS lead sees a score, not a checklist.

### Token pipeline

The three-tier model (primitives → semantic aliases → component bindings) is sound. Three fixes needed:
1. Audit which alias bindings are intact vs hardcoded (automated scan, week 3)
2. Write the Figma variable name ↔ CSS/Tailwind name map once — as a maintained export script
3. Add a lint rule: no hardcoded fill on any published component

### Rituals

| Ritual | Frequency | Who |
| ----- | ----- | ----- |
| DS office hours | Weekly, 30 min | All designers + eng champions |
| Component review | Before new element publishes | DS lead + 1 engineer (async) |
| Migration check-in | Every 2 weeks, 15 min | DS lead + squad leads |
| Automated audit report | Weekly → Slack | DS lead triages alerts |

---

## Phase 2 — Rebuild and migrate (Weeks 7–24)

### Rebuild sequence

```
Tokens (fix alias bindings, map to code)
  → Atoms  (Button, Input, Badge — fewer variants, slots)
    → Molecules (Tabs, Breadcrumbs, Dropdown, Form field)
      → Organisms (Page header, Filter group, Table, Sidebar, Modal)
        → Templates (3–5 page shells: list, detail, settings, form, empty state)
```

Don't publish an organism until the atoms beneath it are canonical.

**Per-component workflow:**
1. Run automated audit → get score and issues
2. DS lead + squad designer review (30 min)
3. Rebuild in Figma branch
4. Engineer reviews slot/prop contract (async)
5. Merge, publish, mark old asset Deprecated with "Use X instead"
6. Squad validates on one real screen before old asset is removed

### Timeline

| Period | Focus |
| ----- | ----- |
| Weeks 7–10 | Tokens + Atoms |
| Weeks 9–14 | Molecules |
| Weeks 11–16 | Organisms (Page header, Filter group, Table) |
| Weeks 15–20 | Shell patterns (Sidebar, Modal) |
| Weeks 18–24 | Templates + squad migration waves |


### Migration waves

| Wave | When | Scope |
| ----- | ----- | ----- |
| **Wave 0 — Dogfood** | Week 10 | DS lead uses rebuilt atoms on one real screen. Zero detaches is the bar. |
| **Wave 1 — Pilot squad** | Weeks 12–16 | One squad migrates their highest-traffic flow. Feedback shapes the process. |
| **Wave 2 — Second squad** | Weeks 16–20 | Different flow type. Process should feel lighter than Wave 1. |
| **Wave 3 — Remaining** | Weeks 20–24 | All remaining flows. Lifted assets on a deprecation countdown. |

**Rule for screens we don't own:** New work uses DS components. Existing screens migrate on the squad's timeline. No forced retroactive migrations.

---

## Adoption

**For designers**
- 3–5 Figma page templates (list, detail, settings) — starting from a canonically assembled template is faster than blank canvas
- Every published component has its description filled: what it is, when to use it, link to docs
- One champion per squad — guides, doesn't gate; trained monthly by DS lead
- Deprecated assets show "Use X instead" in their description

**For engineers**
- Single import path (`@hostaway/ui` or equivalent) enforced by ESLint from day one
- Storybook stories match Figma slot examples — stories are the acceptance criteria for rebuild
- Definition of done: "uses DS component, or has a linked note explaining why not"

---

## Metrics

| Metric | Baseline | Target (month 6) |
| ----- | ----- | ----- |
| Detached instance % in last 20 shipped screens | Set in week 2 | ↓ 70% |
| % new work using canonical components | Set in week 2 | 80%+ |
| Orphan properties on published components | Set in scan | 0 |
| Shadow/duplicate components in code | Set in analysis | 0 net-new after month 3 |
| Variant count — Button / Tabs | 940 / 280 | ≤24 / ≤16 |

---

## Checkpoints

**October 2026 (3 months)**
- ✅ Inventory complete — known build order
- ✅ Library split live (Core + Patterns)
- ✅ Token bindings repaired; Figma ↔ code name map documented
- ✅ Atoms published with variant budgets respected
- ✅ Automated weekly lint running with Slack reporting
- ✅ Wave 0 dogfood done
- ⚠️ Molecules and organisms in progress
- ⚠️ Wave 1 migration starting

**January 2027 (6 months)**
- ✅ Atoms and molecules canonical, documented, in Storybook
- ✅ 3–5 core organisms rebuilt
- ✅ 3–5 page templates in Patterns file
- ✅ 2 squads fully migrated; 80%+ of new work on DS components
- ✅ Lifted monoliths removed or on visible sunset date

---

## Risks

| Risk | Likelihood | Mitigation |
| ----- | ----- | ----- |
| Feature pressure overrides DS work | High | DS lead + eng lead agree: no new lifted publishes. Exceptions require a written note, reviewed monthly. |
| Inventory reveals scope larger than expected | Medium | Build the highest-traffic 20% of components; let the rest deprecate on a longer timeline. |
| Token work blocks atom rebuild | Medium | Time-box token work to 3 weeks. Atoms that can ship with current tokens ship. Token cleanup continues in parallel. |
| Squad designers don't adopt templates | Medium | Start with the squad that contributed to Phase 0. Don't start with a sceptic squad. |
| Engineering migration slower than Figma | High | Figma rebuild and code implementation run in parallel. Neither publishes without the other. |
| DS lead is a single point of failure | High | Champions model + automated tooling reduces bus factor. Decisions documented in a shared log, not Slack DMs. |

---

## Week 1 actions

1. **Agree on this plan** — what's wrong, what's missing, what would make the 6-month target impossible?
2. **Kick off Phase 0 in parallel** — Figma scan, code inventory, designer survey all start in week 1
3. **Schedule the week 2 inventory review** — 60 min to present findings and finalise the migration registry together
4. **Identify one squad champion per product area** — they don't need to act yet, just need to know they're involved
5. **Confirm the code stack** — Untitled React as base vs greenfield wrapper; must be decided before atom rebuild begins (week 7 at the latest)

---

*Roadmap v1 — July 2026. Revisit at 3-month checkpoint.*

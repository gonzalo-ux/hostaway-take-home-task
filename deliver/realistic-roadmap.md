# Hostaway Design System — Realistic Adoption Roadmap

**Horizon:** July – December 2026 (6 months)  
**Team:** 1 DS lead (dedicated) + 6 designers (contributing part-time) + engineering partners from product squads  
**Starting point:** Lifted Untitled UI kit; one validated pilot pattern (Page header + Filter group + Table, graded A−)  
**Goal:** A lean, owned design system that the whole product team actually uses — not a parallel library that gets bypassed under feature pressure.

---

## What we know today — and what we still need to learn

### What we know
- The current Figma library is a copy of Untitled UI's application tier: monolithic, variant-heavy, and detach-reliant.
- The pilot `Template / Page table` proves the right architecture works: composable organisms with named slots, honest template API, code-mappable.
- Tokens exist but have eroded — detached frames, hardcoded fills, no clean Figma ↔ code name map.
- Engineering's React components are more decomposed than Figma. The parity gap is real but bridgeable.

### What we need to learn before committing to component scope
We deliberately avoid listing specific components to rebuild until we know:

1. **Which flows designers touch most** — the 6 designers work on different parts of the product. Before we decide what to rebuild, we need a 1-week inventory of which screens and which components they're opening every week.
2. **What engineers are actually importing** — a static analysis pass on the codebase will show whether the React components align with Figma, and which UI patterns have shadow implementations.
3. **Where the detach rate is highest** — this tells us where the current system is failing people, not where it looks bad.

> **Checkpoint (end of week 2):** Share inventory findings with the team. Adjust scope before Phase 1 begins.

---

## The plan in three phases

```
Phase 0  │ Weeks 1–2   │ Understand before fixing
Phase 1  │ Weeks 3–10  │ Build the infrastructure, not components
Phase 2  │ Weeks 7–24  │ Rebuild, migrate, retire (overlaps Phase 1)
```

Phases overlap intentionally. Waiting for perfect infrastructure before shipping anything is how design systems stall.

---

## Phase 0 — Understand before fixing (Weeks 1–2)

**Goal:** Replace assumptions with evidence. Know what to build before building it.

### What we do

**Figma inventory** (AI-assisted — 1 day, DS lead)  
Using the Figma API and audit tooling, run an automated scan across the published library and the top product files:
- Count published components, their tier, and their variant count
- Flag components with variants that encode content (not structure)
- Identify detached instances in the last 20–30 shipped screens
- Surface orphan properties — things that show in the Properties panel but have no matching layer

This scan is largely automated. The DS lead reviews the output and triages, not runs it manually.

**Code inventory** (engineering partner — 1 day)  
Static analysis of the front-end codebase:
- Which components are imported from the UI kit vs built in-house
- Any shadow/duplicate Button, Table, or Modal implementations
- Token sources: Tailwind config, CSS variables, theme files

**Designer survey** (async — 30 min per designer)  
Six questions. Which flows do you work on? Which components do you touch every week? Where do you detach most? Where do you spend the most time fighting the current library? What's the one thing that would make your day better?

This takes a morning, not a sprint. The answers shape the migration priority.

**Outcome:** A single shared spreadsheet — every published component, its atomic tier, its variant count, its detach rate, its code equivalent, and a preliminary verdict: **Keep / Rebuild / Deprecate**. This becomes the migration registry and stays live for the full 6 months.

> **Stakeholder moment:** Present this inventory at the end of week 2. This is the first thing to challenge — if the priorities don't match what squads feel, adjust now.

---

## Phase 1 — Build the infrastructure (Weeks 3–10)

**Goal:** Give the team a system they can contribute to, not just consume. Infrastructure before components.

### 1.1 Library structure — three files, clear rules

Split Figma into three governed files:

| File | What goes here | Published? |
|---|---|---|
| **Core** | Tokens, icons, atoms, molecules (Button, Input, Badge…) | Yes |
| **Patterns** | Organisms and templates (Page header, Table, Filter group, Sidebar…) | Yes |
| **Reference** | Untitled UI examples, Hostaway screens — inspiration only | No |

**Why this matters:** Today everything lives together. Designers and engineers can't tell what's canonical and what's a one-off. Separating the files makes the rule obvious: if it's in Core or Patterns, it's contract-level. If it's in Reference, it's not.

> **Open question for the team:** Where does work-in-progress live? Suggest a team-level product file per squad — but this needs agreement.

### 1.2 Contribution model — lean, not bureaucratic

The DS lead can't be the bottleneck. With 6 designers and a handful of engineers touching the product, we need a lightweight path for squad-level contributions.

**Two tracks:**

**Track A — DS-led (new core components and patterns)**  
Anything that goes into Core or Patterns needs a full contract before publish. The checklist is short:
- Anatomy (what parts exist, which are required)
- Slot or prop table (Figma names ↔ code props)
- Variant budget (how many variants and why — content permutations go in examples, not variants)
- One do / one don't
- Owner and version

Review is async — DS lead reviews a PR-style Figma branch. No meeting required unless there's a disagreement.

**Track B — Squad-contributed (patterns within a flow)**  
Designers and engineers who own a customer flow (e.g. onboarding, reservations, settings) can contribute a pattern for that flow without going through the full Track A process — as long as it composes from existing Core components. The DS lead reviews it, not approves from scratch. Merged in 48 hours or flagged with feedback.

This is the lean contribution process. Squads with domain knowledge contribute. DS lead maintains quality. No RFC culture.

**AI-assisted review (automated):**  
Every Figma publish triggers an automated audit using the figma-atomic-audit tooling. It checks:
- Are variants encoding structure or content?
- Are orphan properties present?
- Are fills using token aliases or hardcoded values?
- Does the variant count exceed budget?

The DS lead sees a score, not a wall of manual checks. If the score is green, publish proceeds. If red, the automated report tells the contributor exactly what to fix.

### 1.3 Token pipeline — own it, don't rebuild it

Untitled UI's three-tier token model (primitives → semantic aliases → component bindings) is sound. The problem is the lift broke alias bindings on detached frames, and there's no clean Figma ↔ code name map.

The fix is not rebuilding the token model. It's:
1. Auditing which alias bindings are intact vs hardcoded (automated scan, week 3)
2. Writing the Figma variable name ↔ CSS/Tailwind name mapping once — as a maintained document or export script
3. Adding a lint rule: no hardcoded fill on any published component

> **We need more context here:** We don't know yet whether Hostaway's codebase uses Tailwind, CSS custom properties, or something else. The token map can't be finalised until the code inventory (Phase 0) is complete. This is a known dependency.

### 1.4 Rituals — small, regular, asynchronous

| Ritual | Frequency | Format | Who |
|---|---|---|---|
| DS office hours | Weekly, 30 min | Video call or Slack thread | All designers + eng champions |
| Component review | Before every publish | Async Figma branch review | DS lead + 1 engineer |
| Migration check-in | Every 2 weeks | 15-min standup | DS lead + squad leads |
| Automated audit report | Weekly | Slack message with score + issues | Automated → DS lead triage |
| Quarterly health report | Every 3 months | Shared doc | DS lead |

The goal is to make the system self-maintaining. Rituals catch drift before it becomes debt.

**Scheduled automation:**  
Set up weekly automated runs of the Figma lint and audit tooling. Results post to a dedicated Slack channel. No one reads a report — they respond to alerts. If the score drops below a threshold, the DS lead is tagged automatically.

---

## Phase 2 — Rebuild and migrate (Weeks 7–24, overlaps Phase 1)

**Goal:** Replace lifted monoliths with composable, documented, code-aligned components — in priority order based on what we learned in Phase 0.

### 2.1 Rebuild sequence

We don't know the exact component list yet (see Phase 0). What we do know is the **order of dependencies**:

```
Tokens (fix alias bindings, map to code)
  → Atoms  (Button, Input, Badge — reduce variants, add slots)
    → Molecules (Tabs, Breadcrumbs, Dropdown, Form field)
      → Organisms (Page header, Filter group, Table, Sidebar, Modal)
        → Templates (3–5 page shells: list, detail, settings, form, empty state)
```

**Rule:** Don't publish a pattern unless the atoms beneath it are canonical. A slot-based organism pointing to an unreviewed atom is not reusable — it's a better-looking monolith.

**Per-component workflow (same every time):**

1. Run automated audit → get score and issues
2. DS lead and squad designer review audit output together (30 min)
3. DS lead or squad designer rebuilds in Figma branch (Core or Patterns file)
4. Automated review runs on the branch
5. Engineer reviews slot/prop contract (async)
6. Merge to main, publish, mark old lifted asset as Deprecated with a "Use X instead" note
7. Squad validates on one real screen before old asset is removed

**Timeline estimate (adjust after Phase 0 inventory):**

| Period | Focus |
|---|---|
| Weeks 7–10 | Tokens + Atoms (Button, Input, Badge) |
| Weeks 9–14 | Molecules (Tabs, Breadcrumbs, Dropdown) |
| Weeks 11–16 | Organisms (Page header, Filter group, Table) |
| Weeks 15–20 | Shell patterns (Sidebar, Modal — scope TBD pending inventory) |
| Weeks 18–24 | Templates + squad migration waves |

> These ranges are deliberately wide. We'll tighten them at the 3-month checkpoint based on actual velocity.

### 2.2 Migration waves

Don't migrate everything at once. Move squad by squad, flow by flow.

| Wave | When | Scope |
|---|---|---|
| **Wave 0 — DS dogfood** | Week 10 | DS lead uses the rebuilt atoms on one real shipping screen. Zero detaches is the bar. |
| **Wave 1 — pilot squad** | Weeks 12–16 | One squad migrates their highest-traffic flow to canonical components. Qualitative feedback shapes the process. |
| **Wave 2 — second squad** | Weeks 16–20 | Second squad migrates a different flow type. Process should feel lighter than Wave 1. |
| **Wave 3 — remaining** | Weeks 20–24 | Remaining flows and squads. Lifted assets on a deprecation countdown. |

**Engineer codemods** (for atoms like Button and Input): When a component is renamed or its API changes, provide a one-command migration script. Engineers shouldn't touch 40 files manually to adopt a renamed prop.

### 2.3 Handling screens we don't own

Some screens will be owned by squads who can't prioritise DS migration. The rule:  
**New work uses DS components. Existing screens migrate on the squad's timeline.** No forced retroactive migrations. The cost of a forced migration is higher than the cost of a gradual one.

Lifted assets that are genuinely blocking (causing detach loops on new work) get prioritised over ones that are just "technically wrong."

---

## Adoption — making DS the path of least resistance

Infrastructure without adoption is shelf-ware. The team is small, so each of these tactics needs to be low-maintenance.

### For designers

- **Templates over blank canvases.** Ship 3–5 Figma page templates (list, detail, settings) that squads start from. Starting from a template is faster than starting from scratch. Starting from a template that's already canonically assembled is faster still.
- **Descriptions that link to examples.** Every published component in Figma has its description field filled with: what it is, when to use it, and a link to its Storybook/doc page. Searching in Figma returns useful context, not just a name.
- **Champions in each squad.** One designer per squad is trained to answer "should we use DS for this?" They don't gate work — they guide it. DS lead runs a monthly 30-min session with champions to keep them current.
- **"Use X instead" deprecation notes.** When an old lifted asset is deprecated, its Figma description says exactly which component replaces it. No hunting.

### For engineers

- **Single import path.** All DS components import from one path (`@hostaway/ui` or equivalent). No direct Untitled paths in new code. An ESLint rule enforces this from day one.
- **Storybook as a living contract.** Every canonical pattern has Storybook stories that match the Figma slot examples. If Figma says "table with empty state" is a valid configuration, there's a story for it. Stories are the acceptance criteria for the rebuild.
- **Definition of done.** "Uses DS component, or has a linked note explaining why not." This is a code review line item, not a design gate.
- **Automated import auditing.** A weekly CI check reports which files in `apps/web` are importing from deprecated kit paths. Report goes to squad leads. Not a blocker — a signal.

### Metrics we'll track

| Metric | Baseline (week 2) | Target (month 6) |
|---|---|---|
| Detached instance % in last 20 shipped screens | TBD from inventory | ↓ 70% |
| % new work using canonical components | TBD | 80%+ |
| Orphan property count in library | TBD from scan | 0 on published components |
| Shadow/duplicate component count in code | TBD from analysis | 0 net-new after month 3 |
| Variant count for rebuilt atoms | Current: 940 Button, 280 Tabs | ≤24 Button, ≤16 Tabs |

We set the baselines in week 2. If we don't measure, we can't show progress — and we'll lose the argument for continuing.

---

## Timeline at a glance

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

### 3-month checkpoint (October 2026) — realistic bar

Given the team size and the fact that Phase 0 takes 2 weeks:

- ✅ Inventory complete — we know what to build and in what order
- ✅ Library split done — Core and Patterns files exist and are live
- ✅ Token alias bindings audited and repaired; Figma ↔ code name map documented
- ✅ Atoms published: Button, Input, Badge (with variant budgets respected)
- ✅ Automated weekly lint and audit running with Slack reporting
- ✅ Wave 0 dogfood done — DS lead has used the rebuilt atoms on a real screen
- ⚠️ Molecules and organisms likely in progress — not done yet
- ⚠️ Squad migration (Wave 1) starting — not complete

### 6-month checkpoint (January 2027) — full target

- ✅ Atoms and molecules canonical, documented, in Storybook
- ✅ 3–5 core organisms rebuilt (exact list from Phase 0 inventory)
- ✅ 3–5 page templates in the Patterns file
- ✅ 2 squads fully migrated; 80%+ of new work using DS components
- ✅ Deprecated lifted monoliths removed or behind a visible sunset date
- ✅ Metrics show measurable improvement on all tracked dimensions

---

## Risks and honest trade-offs

| Risk | Likelihood | What we do about it |
|---|---|---|
| Feature pressure overrides DS work | High | DS lead + eng lead agree on a "no new lifted publishes" rule. Exceptions require a written note, reviewed monthly. |
| Inventory reveals scope is much larger than expected | Medium | We scope to the highest-traffic 20% of components and let the rest deprecate on a longer timeline. |
| Token work blocks atom rebuild | Medium | Time-box token work to 3 weeks. Atoms that can ship with current tokens ship. Token cleanup continues in parallel. |
| Squad designers don't adopt templates | Medium | Start with the squad whose designer contributed to Phase 0 — buy-in already exists. Don't start with a sceptic squad. |
| Engineering migration slower than Figma | High | Figma rebuild and code implementation run in parallel on the same component. Neither publishes without the other. |
| DS lead becomes single point of failure | High | Champions model + automated tooling reduces bus factor. Document decisions in a shared decision log, not in Slack DMs. |

---

## What this plan does not cover

To keep scope honest:

- **Rebuilding all Untitled UI components** — we're rebuilding the 20–30 that Hostaway actually uses heavily, not the full kit.
- **Pixel-perfect parity between Figma and runtime** — slot names and visual states align; exact interaction animations do not need to match.
- **A public design system site** — a Storybook and a Figma description field is the doc platform for 6 months. A polished external site is a year-two problem.
- **Marketing or one-off components** — they go to Reference. They don't block product work.
- **Perfect historical screen migration** — screens that aren't being actively worked on don't get migrated until a squad touches them.

---

## Immediate next steps (week 1)

1. **Agree on this plan** — what's missing, what's wrong, what's the one thing that would make the 6-month target impossible?
2. **Kick off Phase 0** — DS lead runs the automated Figma scan; engineering partner runs the code inventory; designers fill in the async survey. All three happen in week 1 in parallel.
3. **Schedule the week 2 inventory review** — one 60-minute session where the findings are presented and the migration registry is finalised together.
4. **Identify one squad champion per product area** — they don't need to do anything yet. They need to know they're involved.
5. **Confirm the code stack** — Untitled React as starting point vs greenfield wrapper. This decision needs to be made before atom rebuild begins (week 7 at the latest).

---

*Roadmap v1 — July 2026. Revisit at 3-month checkpoint. Scope adjusts based on Phase 0 inventory and actual team velocity.*

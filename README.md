# CDP Condition Account Scoping — Prototype

Interactive mockup of the HQ vs child account experience for CDP conditions in Xplor Growth.

The condition builder in this prototype is a deliberate replica of
[`xcs-horizontal-builder`](https://github.com/xplor/xcs-horizontal-builder) (XHB), so that the
account-scoping affordances can be judged in the surface they will actually ship in.

## Source

- PRD: `prds/prd-cdp-condition-account-scoping-2026-08-31.md`
- Reference implementation: `xcs-horizontal-builder` (Vue 3 + Apollo Core `xpl-*` web components)
- Design reviews: Aug 31, 2026 (Stephanie Parks)

## How to run

Open `index.html` directly in any modern browser — no server or build step required.

```sh
open designs/cdp-condition-account-scoping/index.html
```

States are deep-linkable via the URL hash:

| Hash | State |
|------|-------|
| `#hq/local` | HQ — tenant-wide evaluation |
| `#child/local` | Child account, condition authored locally |
| `#child/hq-unlocked` | Child account, HQ push unlocked — visible locked location row, no second badge |
| `#child/hq-locked` | Child account, HQ push locked (all locations) |
| `#child/hq-locked-scoped` | Child account, HQ push locked (HQ-selected locations) |

## XHB fidelity

The prototype renders plain HTML using Apollo's own class names, with CSS transcribed from the real
sources so computed styles match the shipped components.

### Apollo Core primitives

Values transcribed from `@xplortech/apollo-core/build/style.css`.

| Primitive | Reproduced detail |
|-----------|-------------------|
| `xpl-select` | `.xpl-input-wrapper` (1px `gray-400`, radius `0.375rem`, `padding: 0` under `.xpl-select`), 44px `.xpl-select__trigger`, absolutely positioned `.xpl-select-value` at `padding-left: 0.5rem` in `#6a6d7d`, `.xpl-select__chevron-down` at `right: 14px` |
| `xpl-input` | wrapper `padding-left: 0.75rem`, 44px input, `--disabled` fills `gray-100` |
| `xpl-button` | `inline-grid`, `gap: 6px`, 40px / 32px (`sm`) / 24px (`xs`), `padding: 10px 18px 10px 20px`, pill radius, `primary` / `secondary` / `subtle` / `warning` / `icon-only`, Apollo's disabled treatment |
| `xpl-tag` | `padding: 2px 28px 2px 10px`, `margin-right: -0.25rem`, `secondary-lm` border, offset close glyph |
| `xpl-dropdown` | translate-and-fade open transition, `0.75rem` options, `secondary-bg-lm` hover |
| `xpl-popover` | `bottom-left` placement (opens to the left of the trigger, bottom-aligned) with arrow |
| `xpl-divider`, `xpl-badge`, `xpl-checkbox`, `xpl-skeleton` metrics | as published |

Apollo builds its DOM through Stencil, where a `<button>` inside a `<button>` is legal. The HTML
parser is not, so multiselect chips render as `role="button"` spans; every class name and metric is
otherwise unchanged.

### XHB feature CSS

| XHB source | Reproduced |
|------------|-----------|
| `XcsHorizontalBuilder.vue` | `.condition-builder-container` (`gap: 1.25rem`), `.divider-container` (`gap: 4rem`) with `.and-button` + `.divider-line` (`#bcbcbc`), `.add-condition-button` (`padding: 1.5rem`, full width, `secondary-lm`, `.disabled` variant), `.condition-builder-actions` right-aligned Save |
| `ConditionCard.css` | card `padding: 1rem` / `gap: 1.25rem` / radius `0.5rem`, `.condition-card-header` with `.rule-title` + `0.073rem × 0.55rem` `.vertical-divider`, `.uia-condition-steps` with `--uia-field-label-height: 34px`, `.uia-condition-row` (`padding: 0.65rem 0 0.65rem 2rem`) and its `::before` / `::after` tree connectors in `gray-500`, `.uia-field-label` / `.uia-field-input` / `.uia-field-controls` |
| `ConditionCard.vue` | `.optional-step-add-button` (2.75rem tall, radius `0.375rem`, `action-primary-lm`, `sliders-3` icon) rendered as the last condition row with an `and` label |
| `ConditionFields/*.vue` | select `13.75rem`; count operator `12.5rem`; timeframe display `13rem`, unit and direction `9rem`, value `5rem`; multiselect inclusion `13.75rem` with a `flex: 1` tag select using `chevron-right` |
| `ConditionSelector.vue` | empty-state category select, condition select revealed only after a category is chosen, secondary "Search All Conditions" |
| `ConfrimPopover.vue` | `20rem` confirm panel, message + optional "Don't show me this again" + `Cancel` / action buttons at `sm`, `warning` state for destructive actions |
| `ReadOnlyExternalCriteria.vue` | indigo read-only panel with lock icon, heading, subtitle and uppercase `Read only` badge — transcribed, but no longer mounted: the HQ-pushed states use header badges instead |

### XHB behaviours that are live

- **Add filter** opens a dropdown of the remaining optional steps; picking one appends a condition
  row with a `subtle` / `warning` trash button in `.uia-field-controls`. The Add filter row stays as
  long as unused optional steps remain.
- **Reset** and **Remove** both route through the confirm popover. Reset carries the
  `condition-reset-confirm` preference — tick "Don't show me this again" and it fires immediately
  from then on. Reset returns the card to the `ConditionSelector` empty state; Remove is disabled
  when only one card is left.
- **Add New Condition** appends a draft card and is disabled while any card has no condition
  selected — the same rule that gates **Save** via `validateCondition()`.
- **AND** connectors only appear between cards, never after the last one.
- Optional-step trash removes a row and returns the step to the Add filter dropdown.

### Deliberate departures

- Real `xpl-*` web components need npm and a bundler; the prototype must open from the filesystem, so
  primitives are CSS replicas rather than the actual components.
- Select and dropdown option *contents* are static: opening a select does not present choices.
- The `ConditionSlideout` ("Search All Conditions") and `TagsSlideout` are not built.
- Skeleton, error and toast states are not exercised, since nothing here is loading over a network.
- Draft cards resolve to a preset template on click rather than presenting real category/condition
  lists.
- `.count-field-input` is set to `6.5rem`, which is the **one invented metric** in the builder CSS.
  `ConditionFields/CountField.vue` renders its `ApolloInput` with no width rule, so the real width
  is whatever Apollo's input defaults to — not verifiable without a build. Every other value in
  layers 1 and 2 is transcribed from source.

### Fidelity audit — 2026-08-31

Re-checked layers 1 and 2 against XHB source, then measured the rendered result in headless Chrome.
`XcsHorizontalBuilder.vue`, `ConditionCard.css`, `ConditionCard.vue`, all five `ConditionFields/*`,
`ConditionSelector.vue`, `ConfrimPopover.vue` and `ReadOnlyExternalCriteria.vue` match. Computed
widths land exactly on their source values — select `13.75rem`, count operator `12.5rem`, timeframe
`13rem` / `9rem` / `5rem`, multiselect inclusion `13.75rem`, Add filter `44px`,
`--uia-field-label-height` `34px` — and the nested-`<button>` constraint holds at zero. The only
gap is the `.count-field-input` width noted above.

Two XHB rules are intentionally not reproduced, both artefacts of Stencil hosts that this prototype
has no equivalent for: `.uia-field-controls :deep(xpl-button) { min-width: unset }` (defensive —
Apollo's `style.css` sets no button `min-width`) and `.condition-card-actions xpl-button
{ display: block; line-height: 1 }` (normalises the custom-element host; the prototype's buttons sit
in an `inline-flex` row and take no baseline gap).

## Account-scoping design

Every scoping affordance is built out of a slot that already exists in XHB, so nothing new has to be
invented in the component library.

| Affordance | Implementation |
|------------|----------------|
| **HQ scope indicator** | purple `xpl-badge` with map-pin icon: **Includes all locations** (removed once marketer adds their own location filter) |
| **Child scope indicator** | same slot, purple map-pin badge: **Only this location** |
| **HQ scoped push indicator** | purple map-pin badge: **N locations** (tooltip lists HQ's picks) |
| **Injected location scope (authored here)** | locked `.uia-condition-row` after required steps — teal connectors, child studio chip, lock tooltip **Locked by your account** |
| **HQ-pushed location scope (visible)** | same row pattern after required steps — locked chips (two visible in this width) then Apollo **+ N more** overflow text when the list exceeds what fits; lock tooltip **Locked by HQ** |
| **HQ optional location filter** | at HQ, **Purchase Location** / **Visit Location** in Add filter; at child **authored here**, equivalent row is injected and locked |
| **HQ-pushed, unlocked** | no second header badge — locked HQ location row communicates scope; child can add filters below; location filter adds **only this studio** and removes scope badge |
| **HQ-pushed, locked** | grey **Locked by HQ** badge; visible locked location row; card fully read-only |

### Rationale

- Map-pin icon on every scope badge so **Only this location**, **Includes all locations**, and **N locations** read as the same control family.
- The injected / HQ-locked location step is styled as a step rather than as a banner because it *is* part of the rule — a marketer reading the card top to bottom should read the location constraint in sequence with the rest of the condition.
- The lock replaces the trash in the controls slot rather than sitting next to it, so the row has exactly one affordance in exactly the place the marketer already looks for one.
- HQ-pushed **unlocked** conditions do not carry a second header badge — the locked location row makes scope visible without a yellow alert competing for attention (design review 2026-09-02).
- HQ-pushed **locked** conditions carry a short grey **Locked by HQ** badge; hover/tooltip carries the rest.
- Locked location rows stay fixed after required steps; filters the marketer adds always append below.
- Long HQ location lists show two chips plus **+ N more** inline text (Apollo `xpl-select` truncate), not an extra chip pill.
- The scope badge is removed at HQ once a marketer adds their own location filter, so the badge never
  contradicts the rule (interaction described in the PRD, not prototyped).

### Open questions — status after the 2026-08-31 verification pass

Full evidence and citations are in PRD §9.

- **Q1 — resolved.** Filter injected at audience-create via `createConditionWithAccountFilter` + `account_filter`.
- **Q2 — partially resolved.** MT aggregates carry a `location` object (`id`, `external_account_id`, `name`). Verified paths: visits → `aggregate.properties.location.external_account_id`; purchases → `aggregate.properties.purchase_location.external_account_id` (same shape, sibling property). Filter values are `external_account_id` strings like `mt-cousteau-101`, mapped from `linkedLocations` numeric IDs via `mt-{tenant}-{id}`. Memberships and forms still TBD.
- **Q3 — resolved.** `linkedLocations` is a Redis SET; multi-location supported; use `IN`/OR across all linked IDs mapped to `external_account_id` strings.
- **Q8 — revised, not blocking for visits/purchases.** Two scoping axes: contact (`associated_accounts`) and activity (aggregate location leaf). Activity scoping is feasible for aggregate conditions now that the location object is confirmed. Still open: whether to inject one leaf or both, and how to handle memberships (trait today) and forms (account-scoped events).

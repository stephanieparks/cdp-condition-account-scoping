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
| `#child/hq-unlocked` | Child account, condition pushed from HQ, still editable |
| `#child/hq-locked` | Child account, condition pushed from HQ and locked |

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
| `ReadOnlyExternalCriteria.vue` | indigo read-only panel with lock icon, heading, subtitle and uppercase `Read only` badge |

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

## Account-scoping design

Every scoping affordance is built out of a slot that already exists in XHB, so nothing new has to be
invented in the component library.

| Affordance | Implementation |
|------------|----------------|
| **HQ scope indicator** | `xpl-badge` with a globe icon in `.condition-card-header`, between `.rule-title` and `.condition-card-actions`: "Evaluating across all locations" |
| **Child scope indicator** | same slot, teal `xpl-badge` with a location pin: "At this location" |
| **Injected location scope** | an ordinary `.uia-condition-row` — a disabled multiselect step whose chip is teal and non-removable, whose tree connectors are teal instead of `gray-500`, and whose `.uia-field-controls` slot carries a disabled lock button where an optional step would carry a trash button. The lock's tooltip names the JSONB property being matched. |
| **HQ-pushed, unlocked** | `ReadOnlyExternalCriteria` panel inside the card, yellow variant, `Scope locked` badge. Steps stay editable and Add filter still works; no location step is injected. |
| **HQ-pushed, locked** | the same panel in its published indigo form with the `Read only` badge; every select and input disabled, Reset/Remove hidden, Add filter row omitted, Save and Add New Condition disabled. |

### Rationale

- HQ uses a globe rather than a location pin to signal brand-wide reach; the child uses a pin.
- The injected location step is styled as a step rather than as a banner because it *is* part of the
  rule — a marketer reading the card top to bottom should read the location constraint in sequence
  with the rest of the condition, not as chrome around it.
- The lock replaces the trash in the controls slot rather than sitting next to it, so the row has
  exactly one affordance in exactly the place the marketer already looks for one.
- Locked HQ conditions reuse the read-only external-criteria treatment instead of a bespoke banner,
  because XHB already teaches that visual language for "someone else owns this".
- The scope badge is removed at HQ once a marketer adds their own location filter, so the badge never
  contradicts the rule (interaction described in the PRD, not prototyped).

### Open questions this prototype does not answer

Tracked in PRD §9 and surfaced here only as tooltip text on the lock control:

- **Q1** — where the location filter is injected in the pipeline.
- **Q2** — the exact JSONB property per condition. The values shown
  (`detail.traits.purchase_location_id`, `detail.traits.visit_location_id`,
  `detail.traits.internal_account_id`) are placeholders pending verification against MT payloads.
- **Q3** — OR semantics when a child account has several linked MT locations. The prototype shows a
  single location chip; a multi-location account would show several in the same chip list.

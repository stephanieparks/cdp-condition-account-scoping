# CDP Condition Account Scoping — Prototype

Interactive mockup of the HQ vs child account experience for CDP conditions in Xplor Growth.

## Source

- PRD: `prds/prd-cdp-condition-account-scoping-2026-08-31.md`
- Design reviews: Aug 31, 2026 (Stephanie Parks)

## What this demonstrates

### Account switcher (harness top-right)
- **HQ — Momentum Fitness**: tenant-wide evaluation; scope indicator on each condition card
- **Studio A — Mission District**: child account with location-scoped evaluation

### Condition tabs
- **Has a Membership** (aggregate condition)
- **Number of Visits** (aggregate condition)
- **Filled Out a Generic Form** (event condition)

### Child account view modes (visible when Studio A is selected)
- **My Segment**: child-authored condition. Auto-injected "At this location" badge + auto-applied location row showing the location chip (non-removable). Child can add other filters but not change scope.
- **HQ-Pushed (Unlocked)**: condition was authored at HQ and is tenant-wide. An amber "From HQ" banner explains scope is locked. Child can add allowed filters.
- **HQ-Pushed (Locked)**: fully locked condition from HQ. Disabled controls, locked banner, explicit scope callout "Evaluating across all locations."

## Key design decisions (not in PRD)

- The scope indicator uses a globe icon (not a location pin) at HQ to signal brand-wide reach; child uses a location pin to signal local scope.
- Auto-injected location rows use teal coloring (matching the child badge) to distinguish them from user-added filters. They carry "Auto-applied · cannot be removed" inline text.
- The locked banner replaces the scope indicator in the card header (not additive) to avoid redundancy.
- The "From HQ (Unlocked)" banner uses amber — matching a "heads-up" signal — rather than the locked-state gray.
- The Add filter button becomes disabled (greyed, pointer-events off) in locked state, matching the pattern used in read-only XHB.
- Card action buttons (Reset, ✕) are hidden entirely in locked state, consistent with the read-only XHB project (no affordance for controls that can't work).

## How to run

Open `index.html` directly in any modern browser — no server or build step required.

```sh
open designs/cdp-condition-account-scoping/index.html
```

## What's stubbed / not built

- Dropdown contents on `<select>`-style elements — these are static visuals.
- The "Add condition" flow — clicking the button does not open a condition picker.
- Actual Apollo web components — the prototype uses Apollo CSS tokens for visual fidelity but renders elements as standard HTML for standalone use without npm/node.
- HQ marketer flow for manually adding a location filter (which would dismiss the scope indicator) — the interaction is described in the PRD but not prototyped.

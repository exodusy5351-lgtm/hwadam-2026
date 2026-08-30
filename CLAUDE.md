# 곤지암·화담숲 가족 여행 아카이빙 시스템

This folder holds **one unified** self-contained HTML artifact for a 3-generation family's Gonjiam Resort + Hwadam Botanic Garden trip (2026-09-23~24):

- `hwadam-archive.html` — the whole trip: Day 1/Day 2 timeline (Gonjiam Resort — check-in, spa, dinner, night walk, table tennis, breakfast, checkout, gondola, brunch), the Hwadam Botanic Garden trail map + course picker + 16-garden encyclopedia + food, and the Memory Wall (shared family photo/note archive via the `artifact` runtime capability). Published artifact: https://claude.ai/code/artifact/0788f19b-7045-43c1-913f-7cfdcdef0aea

- `gonjiam-dashboard.html` — **retired**. It now just renders a short "moved to the unified page" notice at its original URL (https://claude.ai/code/artifact/1579e732-da48-45fc-890c-0e57b4eb6520) so old links don't dead-end. Don't add real content back into this file — everything lives in `hwadam-archive.html` now.

- `index.html` — a **separate, deliberately simpler sibling** for GitHub Pages: same trip content, same tab UI pattern and design tokens, but 100% read-only (no Memory Wall, no `artifact` capability, no shared state) and no D-Day counter — checklist checkboxes are the only interactivity, saved to `localStorage` per device. Its `ENCYCLOPEDIA` list differs on purpose from `hwadam-archive.html`'s: 천년화담송 and 원앙연못 are full 16th-list entries here (dropping 자연생태관/화담채) per an explicit user spec for this file — don't "fix" this to match the other file without asking; they're allowed to diverge. Its trail-map SVG nodes carry an extra invisible `<circle r="22" opacity="0">` touch-hitbox per node for mobile tap targets, and the inner course picker uses a pill-shaped `.seg-toggle`/`.seg-btn` segmented control (visually distinct from the outer `.mtab-btn` chips) — don't collapse these two files into one; they serve different hosts (claude.ai artifact vs. static GitHub Pages) with different constraints.

The live file is a single `<title>/<style>/<body>/<script>` document meant to be published as a Claude Artifact (no build step, no external JS, no CDN frameworks — Google Fonts stylesheet is the only allowed external host).

**Navigation is tab-based, not scroll-based.** `#app` wraps `nav.wing` (the tab bar) plus seven `.tab-content` panels: `#tab-day1`, `#tab-day2`, `#tab-course` (trail map + park facts + landmark note + the parents/trek course picker, merged into one tab), `#tab-encyclopedia`, `#tab-food`, `#tab-memory`, `#tab-guide`. Only one `.tab-content` is `display:block` at a time, toggled by `.mtab-btn` buttons in `nav.wing` via `activateTab(name)`/`initMainTabs()`. `nav.wing` lives INSIDE `#app` (so it's part of the same clone-and-reset cycle `buildFullDocument()` uses) — don't move it back outside without also re-adding a separate reset for it. `.lightbox` and `footer.site` are the only things outside `#app`, read directly (with their own transient-state reset) in `buildFullDocument()`.

Two distinct tab systems exist — don't conflate them: the OUTER page tabs use `.mtab-btn`/`.tab-content` (7 tabs, always exactly one visible); the INNER course picker inside `#tab-course` uses `.tab-btn`/`.tab-panel` (2 tabs: 부모님 코스/트레킹 코스). `resetTransientUI()` resets both independently to index 0 before every publish.

The trail-map's `jumpToCard()` calls `activateTab("encyclopedia")` before scrolling to a card — any new cross-tab link must do the same (switch tab, then `requestAnimationFrame` before scrolling/highlighting) since a card inside a `display:none` tab can't be scrolled to.

## Master Orchestrator role (this is you, the main session)

When a request touches this system, decompose it by domain and route to the matching sub-agent rather than editing everything inline yourself. All domains now edit regions of the **same file**:

| Domain | Agent | Owns (all inside `hwadam-archive.html`) |
|---|---|---|
| Plant/garden facts, seasonal info, encyclopedia copy | `botanic-archive` | `ENCYCLOPEDIA` data, the landmark note, Guide/FAQ facts |
| Timelines, monorail/gondola sequencing, joint-protection rules | `route-care` | `#day1`/`#day2` timelines, the course tabs' mini-timeline + joint-care checklist |
| Photo/note memory wall, trip metadata, shared-state persistence | `family-memories` | `state`/`renderMemoryWallInner`/`buildFullDocument`, `#trip-meta`, garden-tag select |
| Layout, CSS tokens, responsiveness, publishing | `ui-builder` | `#page-style` (the one token system for the whole trip now — Day1/Day2 timeline colors were reskinned into it), trail-map SVG, final `Artifact` publish |

Definitions live in `.claude/agents/*.md`. Use the `Agent` tool with `subagent_type` set to the name above for work that's genuinely isolated (e.g. "verify this one botanical fact," "redesign the course-tab layout"). For a change that touches multiple domains at once, it's usually faster and more consistent to do the integration yourself, informed by each agent's stated rules — re-deriving the current file state four times over costs more than it saves. Use judgment; don't spawn agents reflexively.

Since it's one file now: watch for cross-domain collisions in a single Edit pass — e.g. `.legend`/`.legend-dot` was already taken by the trail map's station/garden legend, so the Day1/Day2 category legend uses `.tl-legend`/`.tl-legend-dot` instead. Check for name collisions before introducing a new class.

## Cross-cutting rules that apply regardless of which agent/mode is active

- **Never fabricate a verifiable fact** — operating hours, fees, tree ages, garden names, menu prices. If you can't verify it this session, hedge it or omit it; don't invent a plausible number. (This bit the "천년화담송" question already: the tree is real and named, but its exact age is not documented anywhere findable — the page says so rather than asserting "1,000 years old" as fact.)
- **The Memory Wall is real family data, not sample content.** Never seed it with invented photos, quotes, names, or dates.
- **Single source of truth per file.** Don't duplicate content (e.g., a garden description) across both files — `gonjiam-dashboard.html` is the trip itinerary; `hwadam-archive.html` is the garden reference + memory archive. Cross-link by mention, not by copy.
- **Republish to the existing artifact URL** (pass `url` to the `Artifact` tool) so the family's saved link keeps working — never create a fresh artifact for an update to either file.
- **No Tailwind/CDN CSS frameworks** — the Artifact CSP blocks them. Both files use hand-authored CSS with a token system; extend it, don't replace it.

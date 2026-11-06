# Harness Brand Motion

Harness Client plugin that animates the Harness brand: the sidebar logo while a
session is generating, and a whale mark that dives before the conversation
title while that conversation is loading (hidden once it is ready).

## Behavior

- **Idle** — both sidebar occupants draw the same artwork the official brand
  draws (the whale path and the `deepseek` + `HARNESS` wordmark, same geometry,
  same `currentColor` / inverted label colors), so the sidebar looks unchanged
  when nothing is running.
- **Generating** — while any Harness session reports `running: true` in the
  `useSessionStatus` snapshot:
  - the **whale morphs** between two deformed body poses (non-uniform squash
    plus a body bow, animated through the SVG `d` attribute) with a soft
    **white** glow — it does not rotate or orbit;
  - a **marquee light band** sweeps across the `DEEPSEEK HARNESS` letters,
    clipped to the glyph outlines.
- The mark's own SVG viewport carries transparent padding and the morph is
  anchored at the mark's left edge, so every deformed frame stays fully visible
  inside the 24 px logo box instead of being clipped by its container.
- `prefers-reduced-motion: reduce` freezes the morph and the sweep, keeping
  only the slow glow.
- **Conversation title mark** — the conversation header's leading seat draws a
  small whale before the session title whenever the session is busy:
  - `openState === 'loading'` → the whale performs the **dive** animation;
  - the model is generating a reply (`running`) → the whale **swims** (gentle
    bob plus squash/stretch);
  - idle → nothing is rendered, so the title returns to its normal position.
- **Composer rainbow** — a hidden marker is rendered inside every Session
  composer card (new session and existing conversations). While that card
  contains a focused editor, CSS draws a rotating, light conic-gradient rainbow
  ring around the card (outside its rounded border), masked to a 2 px stroke.
  `prefers-reduced-motion: reduce` freezes the rotation.
- Each occupant is wrapped in an error boundary; a rendering failure falls back
  to static artwork instead of blanking the brand row.

## Extension points

| Slot | Occupant |
|---|---|
| `sidebar.brand.mark` | animated whale mark (`priority: -10`) |
| `sidebar.brand.name` | `deepseek HARNESS` wordmark with marquee (`priority: -10`) |
| `conversation.header.leading` | whale mark: dives while loading, swims while generating, hidden when idle |
| `conversation.input.overlay` | hidden `composer-rainbow-mark` marker for the focused composer rainbow |

No DOM is written outside the style tag that the plugin installs and removes
with its own effect.

## Background scan scheduler

Every periodic refresh (scroll affordance, account avatar mirror, wait-countdown
cards, error chips, composer toolbar safety net, connection-pill placement, bot
roster pull) registers with **one** scheduler instead of owning a `setInterval`:

- nothing runs while the window is hidden — the timers used to keep waking up
  and returning early;
- a pass that found nothing backs off geometrically (×1.6, capped) and snaps
  back to its nominal cadence the moment the DOM actually changed;
- at most two jobs run per wake-up, so no single pass becomes a long task that
  blocks typing or scrolling;
- the page is watched by one shared `MutationObserver`; when nothing mutated, the
  dirty-only jobs skip their DOM scan entirely.

The cadence is user-selectable in the plugin's gear panel under **性能 → 后台扫描频率**:
`省电` (2.5× slower), `平衡` (default) and `灵敏` (2× faster). The choice is stored
with the plugin's other preferences in `localStorage`.

Three timers stay local because they are short-lived and state-bound: the
per-card countdown tick (1 s, only while a countdown card exists), and the
session-switch overlay's measure/state loops (1 s / 400 ms, at most 4 s).

## Files

| File | Role |
|---|---|
| `index.js` | Host half (no host behavior) |
| `client.js` | Client module: styles, copied artwork, morph, slot occupants |
| `cordis.patch.yml` | Inserts the bundle row |
| `locale/*.json` | Plugin Manager title and description |
| `icon.svg` | Plugin inventory icon |

## Install

Install the package directory with the Plugin Manager `install_bundle` action
(`/Users/aoao/Documents/deepseek-harness/default-workspace/sidebar-logo-animation`),
then toggle it in Settings → Plugins.

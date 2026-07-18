# Workout Tracker — Project Notes

Single-file HTML/CSS/JS workout tracker app. No build tooling, no framework — a static `index.html` with a top-level IIFE `<script>`, styled with plain CSS custom properties, persisted to `localStorage` under the key `workoutTracker.v1`. Chart.js is loaded from a CDN for the Trends screen; that's the only external runtime dependency. Deployed on GitHub Pages from `morgan-gleason/workout-tracker` (live at `https://morgan-gleason.github.io/workout-tracker/`).

Everything below was built in a single long Claude chat session (no git CLI was available there — every change was pushed via GitHub's web upload UI through browser automation). The repo is already up to date with all of it. This file exists so a fresh Claude Code session — or a human — doesn't have to re-derive the design reasoning from scratch by reading the diff history.

## Design philosophy (the thing to protect above all else)

The single recurring instruction across this whole project: the Train/logging experience should feel like a guided, modern workout companion — not a spreadsheet or a data-entry form. Concretely, this means:

- Draw attention to "what's next," not treat every exercise/set as equal weight. Both the current exercise and the current (next incomplete) set get a soft visual focus treatment rather than a hard box.
- Optional/contextual information (Machine Setup, notes) should read as an available convenience, never a required field or a large empty card blocking the user from lifting.
- Elevation (shadow/tone) over borders. Borders are reserved for functional controls (inputs, secondary/ghost buttons, chips, floating popovers/menus/modals/toasts) and soft dividers between rows in the same card — not for cards/rows themselves.
- Color has fixed meaning app-wide: `--accent` (blue) = active/primary, `--accent-2` (green) = completed/success only, `--warn` (amber) = warm-up/attention, `--danger` (red) = destructive only. Template personalization colors are a deliberately separate, muted palette so they never get confused with those semantic colors.
- When adding any new UI element, default to the quietest version that satisfies the requirement, then only add visual weight where the user is actually supposed to act next.

## Feature history (chronological, high level)

1. Core app: routines/templates, exercise picker with equipment + rep-range presets, weekly schedule, streak tracking.
2. Plan tab: Templates / Planning (This Week + Calendar) / Streaks sub-navigation.
3. Full mobile-first, then "modern SaaS" visual redesign pass.
4. Train tab built from scratch: start screen (today's workout, start from template, quick workout, search over templates), active-session screen, finish-summary flow.
5. Set-row redesign: one-set-at-a-time focus, then a full table redesign matching a reference screenshot (SET / PREVIOUS / WEIGHT / REPS columns, compact stepper controls, "W" badge for warm-up sets) to fix a mobile bug where +/- controls crowded out the actual numbers.
6. Prefill weight/reps from the last time an exercise was actually performed (`lastLoggedSetsFor`), shown in a "Previous" column.
7. Current-exercise and current-set auto-focus (`currentExerciseId`, `currentSetIdx`).
8. Color-collision fixes and a full color-semantics pass (see Design Philosophy above).
9. Border-removal pass: replaced `border` with `box-shadow`-based elevation across cards, list rows, stat blocks — kept borders only on genuinely interactive/floating elements.
10. Quick Log moved out of a permanent bottom section into an on-demand modal ("Log a Past Workout"), so it doesn't compete with an active session for space.
11. Search added to "Start from a Template."
12. Sticky in-session header: while a session is active, the app masthead (title/streak/description) hides and a sticky bar shows the workout name, elapsed time, and exercise/set counts instead — scoped to `view === 'log' && activeSession` specifically (not just "a session exists somewhere"), toggled both on view-switch and on in-place render.
13. Swipe-to-delete replaced the per-set "X" button. Built on unified Pointer Events (works for mouse and touch identically). Iterated twice: first version only let the drag start on "dead space" (excluded inputs/buttons) and had no pointer capture, so a mouse drag would silently fail unless it started right at a row's edge. Fixed by letting the drag start anywhere (a plain tap still reaches the button underneath; only a real horizontal drag takes over and suppresses the trailing click) and adding `setPointerCapture` so the gesture keeps tracking even if the cursor drifts off the row mid-drag.
14. Icon system switched to Lucide. Icons are not loaded from a CDN at runtime — they're copied in statically (via the `lucide-static` npm package) into the existing `ICON_SVGS` object and the 6 static nav-bar icons, so there's zero added runtime dependency and the app still works offline. A couple of icons don't have an exact gym-specific Lucide equivalent and were mapped by judgment: "Pull" category → `biceps-flexed`, "Legs" category → `footprints`. The three-dot menu and drag-grip icons deliberately keep a solid-filled-dot look rather than switching to Lucide's stroked-ring default, to avoid a visual-weight regression.
15. Machine Setup feature: optional, per-exercise (keyed by exercise name, not template or session, in `state.machineSetups`) equipment-configuration memory. States: hidden entirely for bodyweight-equipment exercises with nothing saved; a descriptive first-time prompt ("No machine setup saved — save seat positions...") the very first time an exercise is logged; a quiet one-line "+ Save machine setup" after that; a one-time post-completion nudge ("Save this machine setup for next time?") when an exercise is finished with nothing saved (dismissal is in-memory/per-session, so it doesn't nag again); and, once saved, a compact "Machine setup / Seat position 4 · Back pad 2 / Edit" row that expands on tap into the full field list. The add/edit sheet suggests equipment-appropriate field labels (Machine → Seat position/Back pad/Handle position/Cable height/Foot placement; Dumbbell/Barbell → Bench angle/Grip) plus a free-text custom field, so the form never asks for irrelevant inputs.

## Established engineering conventions

- Verification before every push: extract the inline `<script>`, `node --check` it; a quick tag-balance sanity check (div/span/button/svg/etc. open vs. close counts); a jsdom test file per feature under `/tmp/jsdomtest/` (not part of the repo — recreate locally if needed) that drives the app through real DOM events (clicks, `PointerEvent`s, `KeyboardEvent`s) rather than calling internal functions directly, since the script's functions aren't exposed on `window`. Re-run the entire existing suite after every change, not just the new test — several regressions were caught this way (e.g. a swipe-delete restructuring silently breaking an unrelated prefill test's selector).
- jsdom gotcha: the running app reads from an in-memory `state` object populated once at `load()`, not from `localStorage` on every call. Mutating `localStorage` directly mid-test does not affect the app's behavior — drive state changes through real UI interactions (clicks on real buttons/rows) instead, or pre-seed `localStorage` via a `<script>` injected right after `<body>` before the app's own script runs.
- Data model migrations: `load()` always merges saved state with sane defaults field-by-field (e.g. `if (!s.machineSetups) s.machineSetups = {};`) so old saved data never crashes the app or silently loses a whole feature area.
- CSS/JS style: no build step, so keep everything single-file. Icons are inline SVG strings in the `ICON_SVGS` object, referenced as `${ICON_SVGS.name}` in template literals — add new icons there rather than inlining fresh `<svg>` markup elsewhere (a few old inline duplicates of plus/X exist from before this convention solidified; harmless, but don't add more).

## Product spec (source of truth)

Merged from `Workout tracker outline.pdf` (dropped in the repo 2026-07-18), which is the authoritative feature spec across five pillars. Status below reflects what's actually in `index.html` as of this file's last update, reconciled against that outline — not what any earlier chat believed was built. Re-verify against the code before relying on a status mark for anything you're about to change.

### Plan — help users prepare before they arrive at the gym

- **Workout Templates** (create/edit/duplicate/delete, organize by Push/Pull/Legs/Upper/Lower/Full Body/Custom) — ✅ built.
- **Workout Planning** — 🟡 partial. This Week + Calendar sub-views exist. "Upcoming" and "Consistency" sections, quick-edit interactions, and a secondary read-only Month view were sketched but never built. Rest days / rearranging / one-off workouts — unconfirmed, check the code rather than assuming either way.
- **Workout Streaks** (current/longest streak, weekly goal, monthly workouts) — ✅ built.

### Train — make it easy to track, execute, and adapt at the gym

- **Core tracking** (exercises, sets, reps, weight, set type, completed sets) — ✅ built.
- **Set Types** — 🟡 partial gap. Outline specifies six: Warm-up, Working, Failure, Drop Set, AMRAP, Assisted. Only Warm-up and Working exist in the current color-semantics/data model. Failure, Drop Set, AMRAP, and Assisted are unimplemented — this needs its own design pass (new colors/badges without reusing the four semantic colors already spoken for).
- **Workout Editing** — Add/Remove exercises and Modify weight are done (per the outline's own strikethrough). Reorder exercises is likely built (a drag-grip icon exists in the icon system) but unconfirmed. **Swap Exercise** and **Skip Exercise (with reason)** are ⬜ not built.
- **Quick Actions** (repeat last weight, repeat previous workout, +2.5/+5/+10) — ⬜ not built as explicit actions. Automatic prefill from last-logged sets (`lastLoggedSetsFor`) already covers the passive "repeat last weight" case, but there's no one-tap action for it or for the numeric increments.
- **Notes** (workout-level "overall thoughts," exercise-level notes) — ⬜ not built. No home in the data model yet.
- **Machine Settings** — ✅ built, and more developed than the outline describes (equipment-aware field suggestions, first-time prompts, post-completion nudges — see Feature history #15).
- **Exercise History card** (Target / Last Workout / Personal Best / Notes, shown inline while logging) — ⬜ not built. This is the biggest single gap: the outline's mock-up (page 4 of the PDF) shows a fully designed 4-block card, but only the underlying prefill data exists today, not the card UI.
- **Rest Timer** (countdown, "Skip Rest," shows next set's target) — ⬜ not built, and not previously discussed anywhere in this project's history before the outline doc. Treat as a new feature, not a rediscovered one.
- **RPE column** — known inconsistency: RPE exists in the data model and the old Quick Log form, shown in the outline's mock-up, but missing from the current Train set rows.

### Review — capture how the workout felt while it's fresh

- **Immediate post-workout reflection** (workout rating, energy level, difficulty, notes) — ⬜ not built at all.
- **Next-day reflection** (soreness, recovery, fatigue, motivation) — 🟡 partial. Only soreness and energy check-ins exist ("How I feel," with optional muscle-group notes). Recovery and motivation as distinct fields are not built.

### Progress — show long-term trends

- **Strength Trends** (Estimated 1RM, best weight, average weight, average reps per exercise) — 🟡 partial. Top-weight-over-time exists; Estimated 1RM and average-weight/average-reps stats do not.
- **Volume Tracking by muscle group** — 🟡 partial. Volume trends exist per-exercise; aggregation by muscle group (Chest/Back/Legs/Shoulders/Arms) is not built.
- **Measurements** (body weight, waist, chest, arms, legs) — ⬜ not built.
- **Workout Analytics** (avg duration, longest workout, avg rest time, avg exercises, avg sets) — ⬜ not built.

### Integrations — ⬜ not built, and worth a separate conversation before scoping

Apple Health, Apple Watch, Oura, WHOOP. This app is static, client-side, `localStorage`-only, with no backend and no build step. HealthKit isn't reachable from a plain web page (needs a native app shell), and Oura/WHOOP need OAuth + a server to hold tokens and receive webhooks. This is an architecture decision, not a normal backlog item — don't start implementing any of these without confirming the approach (e.g. a companion native wrapper, or a backend service) first.

### Future (explicitly deferred in the outline)

AI workout generation (by target muscle groups / equipment / duration), AI coaching, progressive overload suggestions, plateau detection, exercise recommendations, deload suggestions, smart recovery recommendations, progress photos, undertrained-muscle-group highlighting, supersets. None started; no conflict with anything shipped.

### Also noted, not in the outline

- No "resume workout in progress" banner appears on other tabs while a session is active in the background.
- Richer Add-Exercise picker (muscle group / equipment / recent / favorite browsing) — currently a single search combobox.

## Deployment

Was: GitHub's web upload UI via browser automation (no git). Now that this project is moving to Claude Code: the repo is already current, so just `git clone` it and use normal `git add / commit / push` — no file transfer needed, no CDN-propagation waiting required (Pages will just pick up the new commit the same way it always has).

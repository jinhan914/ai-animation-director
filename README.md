# Director — AI Animation Director

**SLAY HACKS 2026 entry (solo).** Prize $4,500 · Deadline 7 Dec 2026, 17:00 EST ·
[slayy-hacks.devpost.com](https://slayy-hacks.devpost.com/)

Type a script. Director casts the animation.

Most tools ask you to pick a library first. Director does the opposite: it reads
your script, splits it into scenes, and picks the right animation library for
every scene — then generates the animation and plays it back in your browser.

## Run it

No build step, no backend, no account. Any static server works:

```bash
python3 -m http.server 8129
# open http://127.0.0.1:8129/index.html
```

Or open `index.html` directly in a browser (Google Fonts load from the network
with system-font fallbacks if offline).

## What it does, step by step

1. **Write** — paste a script, a product pitch, or a lesson. Blank lines start a
   new scene; a single paragraph is split into sentences. Three presets ship
   with it: Product launch, Maths explainer, A short story.
2. **Cast** — each scene is classified by what it *is*: words, data, a formula,
   something spatial, or pure energy. The classifier is word-boundary based and
   was tested against all three presets (12/12 expected casts).
3. **Play** — a 16:9 stage plays the whole cut: play/pause, previous/next scene,
   timeline scrub, per-scene tabs, and a pacing control (Calm 130 / Standard
   150 / Punchy 185 words per minute). Scene length comes from the word count,
   so a longer script makes a longer cut.
4. **Inspect** — every scene shows its library choice with a plain-language
   reason *and* the rejected alternatives, so the choice can be argued with.
   The generated code for the scene you are watching is shown and copyable.
5. **Export** — download the storyboard as JSON (scenes, timings, libraries,
   matched signals, reasons) for the video edit or for a production pipeline.

## The casting rules

From a systematic study of 18 free, open-source animation/video libraries
(`../custom-skills/animation-video-production/` in the author's notes):

| Scene is… | Cast | Why |
|---|---|---|
| Words / a headline | **Anime.js** — kinetic type | Small, framework-free timeline; stagger and easing with no build step |
| Data / numbers | **Motion Canvas** — diagram engine | Built for code-driven diagrams; bars animate from the numbers |
| A formula / proof | **Manim** — maths explainer | A formula is a Mobject; a proof is a transformation between Mobjects |
| Spatial / orbits | **Three.js** — light 3D | One camera, one light, one orbit — no full engine needed |
| Energy / a launch | **Proton** — particle engine | Bursts and trails that live in the page, no video render |

Rejected on principle, and listed in the app: Ola / Shifty / TweenJS are
tweening layers, not complete solutions; Lottie Web only *plays* After Effects
JSON; gifenc only encodes frames you already have; Babylon.js is a full engine
where a light scene only needs Three.js; Blender / Godot are never the right
tool for a light web animation.

The in-browser player is a *demonstration* of the casting engine: it renders
each scene with a Canvas 2D renderer in the style of the cast library. The
production pipeline the generated code points at is the real one — Remotion
for a narrated MP4, Manim for maths video, Motion Canvas for diagram video —
documented per scene in the generated code panel.

## Design

Dark studio theme. Space Grotesk headings, Inter body, JetBrains Mono code —
a Utopia-style fluid scale (`clamp()`), Radix-style 12-level neutrals, one
lime accent. Checked against the 62 Impeccable "AI look" checks: no beige
palette, no italic serif headline, no coloured bar on a callout, no card
inside a card, no numbered 01 Discover heading.

## Verified

- `node --check` on the extracted script: pass.
- Real Chrome 158 headless, desktop 1280px and mobile 390px, all three
  presets end-to-end (play, pause, scene nav, scrub, pacing, code view,
  storyboard export): pass, zero app errors. Classifier still 12/12 after
  the refactor. Fresh screenshots in `screenshots/` (desktop-top, mobile,
  light-top, stage-diagram/formula/orbit/particles) were looked at, not just
  captured. One console error remains in the sandbox only: Google Fonts fails
  through the egress proxy (`ERR_TUNNEL_CONNECTION_FAILED`); system-font
  fallbacks cover it.
- Quality pass 2026-11-21 (this round) found and fixed, all re-tested after
  each fix:
  - Full light theme: the style switch only recoloured the canvas before;
    now the whole page follows it, with a dark-green lime for text accents
    on light backgrounds.
  - Scene tabs overflowed on desktop (871px of tabs in a 760px strip);
    they now fit exactly on desktop and scroll horizontally on mobile.
  - Every "source on GitHub" link pointed at github.com; each library now
    links to its real repo.
  - The diagram used invented sample values even when the scene's own
    numbers were there; bars now come from the scene's numbers when they
    are usable (at least two, within 20x of each other), and the chart and
    the generated code say so honestly. (Round 2 removed the formula
    scene's amber key-sentence line under the curve — it duplicated the
    title; the key sentence still shows in Director's choices and in the
    generated code.)
  - The kinetic preview could end on a lone short word ("a" alone on a
    line); a paused kinetic scene now previews fully revealed, with an
    orphan guard that pulls the previous word down.
  - Each decision now shows the exact words that won the scene
    ("Matched on: “burst”, “launch”, “particles”, “light”"), and a
    per-scene "In this scene" line, so three Manim scenes no longer read
    as three copies of one paragraph.
  - Mobile: the overlay pills covered the canvas title on the short phone
    stage; only the library pill shows now, smaller, and canvas titles
    start lower. Transport ‹ › arrows sat at the bottom of their circles;
    they are now ← → centred with flex.
  - One bug introduced and caught by the same pass: the kinetic wrap
    measured text before the 44px font was set, so the first line ran off
    the stage; the font is now set before measuring (wrap check: lines
    790 / 853 / 112 px, limit 864).
- Two real bugs found and fixed by the first round of testing: a duplicate
  `id="decisions"` that wiped the stats panel, and centred canvas titles
  drawn off-centre.
- Doubao polish round 1 (2026-11-21, 16 issues) — all P0/P1 implemented and
  re-tested in real Chrome 158 (desktop 1280px + mobile 390px), classifier
  still 12/12:
  - Export JSON: try/catch, >120 KB confirm, anchor appended to the DOM,
    success/failure feedback, object URL revoked.
  - Classifier: minimum activation threshold plus a deterministic tie-break
    (formula > diagram > particles > orbit > kinetic within 1 point).
    Caught and fixed in testing: the first contender filter also matched
    zero-score kinds, which flipped a "light"-only scene to Manim.
  - Canvas buffer now follows its container via `resizeCanvas()` (logical
    960×540 space, scaled with `setTransform`); `ResizeObserver` + resize
    on theme/scene change. Buffer measured 758×426 at a 1280px viewport.
  - `visibilitychange` pauses the player when the tab is hidden; stale
    rAF is cancelled before a new loop is scheduled.
  - Copy failure now says "please select & copy manually" and restores
    the button text after 1.5 s (verified in headless, where clipboard
    is unavailable).
  - Preset chips fill the textarea only — they no longer auto-analyze
    and wipe the cut being watched.
  - Scene tabs now show a thin visible scrollbar instead of a hidden one.
  - Light theme: the code panel is now a real light panel (dark text on
    white, dark-green keywords) instead of light-green text on a dark
    panel; other light leftovers audited.
  - Scrub `oninput` only redraws + updates the clock; the full UI refresh
    happens on `change` (drag release).
  - `classify` never returns an empty signal list; `sceneJob` has a
    safe default return.
  - Generated code now has lightweight keyword/string/comment
    highlighting with the existing `.k`/`.s`/`.c` classes, no new library.
  - P2 items done too: empty/symbol-only input is rejected with an alert;
    decision text wraps anywhere; particle count scales 80–220 by word
    count; the internal submission checklist was moved out of the public
    README into a private note.
- Doubao polish round 2 (2026-11-21) — Doubao verified all 15 round-1
  fixes correct, no new bugs, and decided: keep minimum activation = 1,
  do not raise it, no per-kind thresholds. Implemented:
  - Formula scene: the amber key-sentence line under the curve is gone —
    the title appears once; axes, growth curve and bottom words/time text
    unchanged. Key sentence still in Director's choices and generated code.
  - `escHtml`: Doubao flagged a stray space in the `>` escape regex, but
    the code was checked byte-for-byte and already reads `/>/g` — no
    change needed, no bare `>` can leak through.
  - `play()` with no scenes now shows a hint alert instead of returning
    silently.
  - `VIDEO_SCRIPT.md` updated to the new flow: preset fills the textarea
    only, then the user clicks "Direct my animation".
  - The three previously untested paths were exercised headlessly: >120 KB
    export shows "Storyboard is 194 KB — download anyway?" (decline
    leaves the button unchanged, confirm downloads and shows "Saved ✓");
    `visibilitychange` pauses playback (time frozen, button back to ▶);
    http clipboard fallback shows the manual-copy message and restores.
  - Classifier still 12/12 (launch: Anime.js ×3 + Proton — scene 3 wins
    Proton on four particle signals "burst", "launch", "particles",
    "light"; maths: Motion Canvas + Manim ×3; story: Three.js + Anime.js
    + Proton ×2). Desktop 1280px + mobile 390px, zero app errors.
  - Fresh screenshots: `r2-formula-stage.png` (title once, curve + axes
    intact), `r2-mobile-stage.png` (390px, no overflow, no collision).

## Status

Demo, screenshots and the 2–4 minute video storyboard (`VIDEO_SCRIPT.md`)
are in this repo. GitHub and Devpost submission are handled by the author;
see the repo for the demo and the video storyboard.

## Files

- `index.html` — the whole app (single file, ~36 KB)
- `README.md` — this file
- `VIDEO_SCRIPT.md` — demo video storyboard (2–4 min)
- `screenshots/` — verified desktop + mobile captures

## License

MIT. The 18 libraries it casts are their authors' own licences; see the
animation-video-production skill reference for each.

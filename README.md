# Handoff: Boardroom Landing Page ("brdrm")

## Overview
Scroll-driven landing page for **Boardroom** — a shared-canvas app. One pinned stage morphs a live, interactive collage board from a full-bleed hero into an iPhone app view, and finally into an iOS home screen, ending on a waitlist email capture. The board is genuinely interactive (draw, write notes, place stickers, take webcam photos, drag everything). Each visitor's session is single-player; the top badge shows a simulated live count.

## About the Design Files
The files in this bundle are **design references created in HTML** — working prototypes showing the intended look and behavior, not production code to copy directly. The task is to **recreate this design in the target codebase** (the real repo `adilanchian/brdrmdotapp` is Next.js + Vercel) using its established patterns.

**Shortcut that is allowed:** `build/index.html` is a fully self-contained static build (all assets/fonts inlined, works offline). It can be shipped AS-IS as a static page (e.g. committed to `public/lp.html` in the Next.js repo → served at `/lp.html`) while a native reimplementation is built.

## Fidelity
**High-fidelity.** Built from the Boardroom Figma file. All colors, sizes, coordinates and type below are exact — do not round them.

## Stage & scroll model
- Design space: **402.063 × 874** px, scaled uniformly to fit viewport: `scale = min(vw/402.063, vh/874)`, centered in a 100vh sticky viewport inside a scroll track.
- Track height: **2.5×** viewport height (desktop), **3.4×** (mobile). Set heights in **px from `window.innerHeight`** (re-measured on resize) — not `100vh` — so iOS Safari's collapsing bars don't hide content. `overscroll-behavior-y: contain` on the scroller.
- Scroll progress `p` (0–1). **Mobile (<760px): pScene = p** (starts at scene 1, board full-bleed). **Desktop: pScene = 0.5 + 0.5·p** (starts at scene 2, board-in-phone).
- All keyframe interpolation uses smoothstep between three keyframes at pScene 0 / 0.5 / 1.

### Morph keyframes (x, y in stage px; s = scale)
- **Board** (340.464×340.464 local space, radius 26.289): {31.177, 156.317, 1} → {81.699, 185.529, .697} → {109.612, 234.149, .5371}. `overflow:hidden` clips all content. White rim `0 0 0 1.37px rgba(255,255,255,.9)` + drop `0 18px 40px rgba(30,32,28,.10)`; the white rim's alpha fades to 0 over pScene .58–.82 (so no outline on the dark home screen).
- **Phone**: {x15, y11, 371×807, r56} → {68.398, 126.059, 265.266×576.723, r40} → {98.565, 188.781, 204.932×445.548, r26.95}. Opacity in over .12–.46. Background crossfades rgb(239,239,237) → rgb(211,213,195) over .55–.85. Border: `inset 0 0 0 1px rgba(0,0,0,.15)`; shadows `0 34px 80px rgba(30,32,28,.18), 0 8px 20px rgba(30,32,28,.10)`.
- **Home screen**: `assets/homescreen.png` (603×1311) fills the phone (`object-fit:cover`), fades in over .62–.9. The two flower app rows are links to `https://www.oneyear.garden/` (hotspots at left 9%, top 53.6% and 60%, size 13%×5.8% of phone; pointer-events only when home-screen opacity > .6; `target="_blank"`).
- **"boardroom" pill** (black, radius 130, SF Mono 17.012px white): {141.327, 72, 1} → {163.063, 138.196, .636} → {171.7, 198.16, .491}. The pill itself stays visible whenever the phone is visible; **only its text fades** with `appOp`.
- **App chrome fade**: `appOp = 1 − smooth(seg(pScene, .58, .78))`.
- **Avatars row**: base {32.177, 509}; {32.177,509,1} → {82.697,433.526,.6386} → {109.611,426.299,.4933} (shrinks with the phone), opacity `appOp`.
- **Toolbar**: base {91.147, 610.198}; {91.147,610.198,1} → {121.923,541.665,.7295} → {139.913,509.837,.5635}, opacity `appOp`.
- **Landing copy** "Welcome to Boardroom." (SF Mono 500 14px) fades in .72–.93 at stage y100 centered; **email capture** fades in .8–.98 (pointer-events when >.6).
- **Hero-only**: live badge + hero extras fade out over pRaw .06–.24; "scroll down" hint (14px + 11×13 arrow, bobbing 1.4s) fades over pRaw 0–.05.

## Board content (local 340.464² space)
Items sit in a collage layer offset (−3.168, 26.982); **positions live in `left/top`, rotations in `transform:matrix`** — never bake positions into the matrix, or scale animations will slide items. All items draggable (see Interactions).
- **NYC Crew glass tag** — 106.295×30.133 at (14.058, 14.499), radius 119.497, padding 1.826/14.34, centered. Liquid glass: `background: linear-gradient(140deg, rgba(255,255,255,.24), rgba(255,255,255,.07) 60%, rgba(255,255,255,.15))`; `backdrop-filter: blur(7px) saturate(170%)`; `box-shadow: inset 0 1px 1px rgba(255,255,255,.7), inset 0 0 0 .75px rgba(255,255,255,.4), 0 4px 12px rgba(30,32,28,.12)`. Text SF Mono 500 16.729px black. **Always the topmost board layer** (above drawings/notes), pointer-events none. Fades in at .2s on load.
- **Crown doodle** — two open SVG paths, stroke rgb(189,0,255), widths 4.2 and 4, round caps/joins. Container 132.882×94.636 at (17.009, 7.47); base bar 89.572×24.486 at (40.89, 80.784). Paths (viewBox 133×95 / 90×25): `M9 90 C6 62 5 30 8 16 C8.5 13 11 13 12 16 L34 54 C35.5 56.5 38.5 56.5 40 53 L57 10 C58 7.5 61 7.5 62 10 L82 52 C83.5 55 86.5 55 88 52 L108 15 C109.5 12.5 112 13.5 112 17 C114.5 41 114.5 66 112 90` and `M3 21 C22 6 47 3 70 9 C79 11 85 14 88 18`.
- **Dog** photo (assets/dog.png cutout) — 149.763×237.404, left 0 / top 74.046, rotation matrix(0.974,−0.228,0.228,0.974).
- **Sunset** photo (assets/sunset.jpg) — 140.636×187.515, left 216.888 / top 0, matrix(0.980,0.198,−0.198,0.980), radius 8.218, shadow `0 4px 14px rgba(0,0,0,.18)`.
- **Pills** (SF Mono 500 16.729px white, radius 17.924, padding 6.392/14.34/9.56): "omg haha" blue rgb(31,107,255) at (54.746, 244.106); "Last day?" green rgb(3,153,0) at (185.016, 24.955); voice note "▶ 8s" purple rgb(189,0,255) 69.028×39.208 at (255.692, 177.448) with white play triangle. Dog-emoji badge 🐶 green 42.004×34.101 at (24.274, 155.212), matrix(0.959,−0.285,0.285,0.959).

## Load choreography (plays once; board starts EMPTY)
CSS animations with `backwards` fill (hidden during their delay). Human feel: decisive ease-out `cubic-bezier(.2,.7,.3,1)`, **no bouncy overshoot**.
- 0.2s — NYC tag fades in (.45s)
- 0.45s — dog slides in .55s: from translate(−12px,10px) rotate(−5°) scale(.93) fade
- 0.9s — sunset slides in .55s: from translate(18px,6px) scale(.93)
- 1.0s — emoji pops (.35s)
- 1.4–2.5s — crown **draws itself** (pathLength=1, dashoffset 1→0, ease-in-out; base bar 2.3s +.5s). Keep the path invisible until its delay ends (1ms opacity flip) or the round line-cap shows as a dot.
- 1.9s / 2.4s / 2.8s — "Last day?" / "omg haha" / voice pill **pop up in place**: scale .72→1 + fade, `cubic-bezier(.2,.7,.35,1)` (~.4s each).

## Interactions & behavior
Interactivity gate: scroll progress `p < 0.1` (the hero/widget view).
- **Toolbar** — 6 circular buttons 57.887px, bg rgb(219,219,214), icons stroke/fill `currentColor` black. Selected: bg black, icon rgb(219,219,214), scale 1.04, transitions .15s. **Voice + image buttons are disabled**: opacity .35, no pointer response. Icons are the exact SVGs in `assets/icons/` (uniform 2.894 stroke — render each at its natural size).
- **Aa (text)** — tapping the button immediately opens an input pill on the board (random spot near center): blue pill, SF Mono 500 16px white, placeholder "note…" in `rgba(255,255,255,.55)`, pops in. Enter commits it as a draggable note (random ±4° rotation); Esc/blur-empty cancels. Don't use `window.prompt` (blocked in sandboxed iframes; also bad UX).
- **Sticker** — tapping the button immediately opens a glass picker panel (white .9 + blur(10px) saturate(160%), radius 18, shadow) with all 18 stickers (`assets/stickers/`), cells 38px (46px on mobile), hover bg rgba(0,0,0,.07). Choosing one places it near board center at width 74, random ±7°, drop-shadow `0 5px 12px rgba(0,0,0,.18)`. Click-outside closes.
- **Draw (scribble)** — desktop default-selected. Pointer draws polyline strokes into an SVG ink layer that renders **above** items; stroke = `yourColor` (default blue), width 4.6, round caps. `touch-action:none` while drawing.
- **Camera** — tapping opens a 150px card on the board: mirrored live `getUserMedia` video (playsinline muted), white 30px shutter, ✕ close. Shutter draws the frame to canvas (mirrored), JPEG quality .82, places it as a draggable 120px photo. Graceful "camera blocked/unavailable" fallback (1.6s then dismiss).
- **Move** — when no tool is selected, every board item (collage + placed) drags with the pointer (CSS `translate`, composes with base transform). Grabbed item gets z-index 30. Empty-area presses with a tool active draw/place; presses on items always drag when no tool.
- **Email capture** — pill 234×43 radius 100, `inset 0 0 0 1.5px rgb(219,219,214)`, input centered SF Mono 500 **16px** (≥16 avoids iOS focus zoom). Placeholder **"join the waitlist"**, swaps to "your@email.com" on focus (reverts if left empty). Black 33px circular arrow button (right 5px) appears only once text is typed (fade+scale .2s). Enter or click validates `/^[^@\s]+@[^@\s]+\.[^@\s]+$/`; failure flashes the ring `rgb(255,90,61)` for .9s; success swaps the pill to solid black "✦ you're on the list" with a pop. TODO for production: POST the email to a real endpoint.
- **Live badge** — translucent white pill (rgba(255,255,255,.55), inset ring rgba(0,0,0,.06), blur(6px)) at stage top 30, centered: 4 overlapping 9px dots (blue, pink rgb(255,31,233), green, purple; white 1.6px rings; staggered 1.6s pulse) + "N board members editing" (SF Mono 500 12.5px). N starts random 21–53, drifts ±1–2 every 2.6–5s, clamped 12–72.

## State
- `strokes[]` {points, color, width}, `notes[]` {id, x, y, rot, text|img, w, color}, `tool` (text|pen|photo|voice|camera|scribble|null), `emailDone`, live count. All per-session (no persistence, no realtime).

## Design tokens
- **Colors**: page/phone bg `rgb(239,239,237)`; board paper `rgb(232,233,224)`; button grey `rgb(219,219,214)`; grey-2 `rgb(226,226,223)`; black `rgb(0,0,0)`; blue `rgb(31,107,255)` (You / default draw); green `rgb(3,153,0)` (Luke); purple `rgb(189,0,255)` (Mia / crown); pink `rgb(255,31,233)`; orange `rgb(255,90,61)` (error flash); link blue on page = rgb(31,107,255).
- **Type**: SF Mono (stack: `'SF Mono', ui-monospace, 'DM Mono', Menlo, monospace`; DM Mono loaded from Google Fonts as web fallback), weight 500, letter-spacing −0.05em everywhere. Sizes: 16.729 board pills/tag; 18.321 avatars; 17.012 boardroom pill; 16 notes/email input; 14 landing copy & scroll hint; 12.5 live badge. "Aa" icon: SF Pro Rounded 600 24.12px.
- **Radii**: board 26.289; phone 56→26.95 (morphs); pills 130; tag 119.497; note pills 17.924/17; email 100; sticker panel 18.
- **Avatars row**: You (blue, filled), Luke (green, filled), Mia (purple, filled), "Add" (grey rgb(226,226,223), text rgb(90,90,86)) — pills padding 5.235/15.704, radius 130, 18.321px.

## Mobile specifics (<760px)
Starts at scene 1 (board full-bleed); no default tool (so touch scroll works — tools opt-in); px-based viewport heights (see Stage); inputs ≥16px; sticker cells 46px; camera shutter 30px; longer track (3.4×vh).

## Assets (in this bundle)
- `assets/dog.png` — dog cutout photo (board + reused in design)
- `assets/sunset.jpg` — sunset photo
- `assets/homescreen.png` — iOS home screen screenshot (603×1311) shown in scene 3; flower rows = one-year-garden app
- `assets/icons/` — the 6 toolbar icon SVGs (text/Aa, sticker, image, voice, camera, draw) — exact Figma exports, uniform 2.89435 stroke
- `assets/stickers/` — 18 sticker PNGs for the sticker picker
- Source of truth for visuals: the **Boardroom Figma file** (owned by the user)

## Files
- `design-source/Boardroom.dc.html` + `design-source/support.js` — the interactive design prototype (open Boardroom.dc.html in a browser next to support.js). **Reference for all behavior and values.**
- `build/index.html` — self-contained production-ready static build (everything inlined, ~1.7MB). Ship as-is if desired (`public/lp.html` in the Next.js repo, or any static host / GitHub Pages).

## Deploy context (as of handoff)
- Real repo: `adilanchian/brdrmdotapp` (Next.js, Vercel project "Wind Down Studio" → brdrmdotapp.vercel.app). Note: Vercel blocks deployments for commits authored by `sad-ape` (not on the Vercel team) while the repo is private — commit as `adilanchian`, or make the repo public.
- Temp test deploy: `sad-ape/brdrm-website` (public) with this `index.html` at root — enable GitHub Pages (main/root) → https://sad-ape.github.io/brdrm-website/.

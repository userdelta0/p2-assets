# Hosting bundle — Portal 2 ping tool (StarfallEx)

Everything in THIS folder is what must be hosted online for the URL-material / URL-sound
path (Tier-1, works for players without the HUD). Lines/spokes/rings/beam are drawn in code,
so they are NOT here. 30 files, ~490 KB.

```
hosting/                      <- CFG.base points at THIS folder's public root
  wheel/*.png    (23)         <- one PNG per distinct ping-command icon (static HUD control)
  rings/         (5)          <- ground-effect ring layers, team-tinted + rotated in code
    ring.png ring_rev.png crosshair.png crosshair_add.png wedge.png
  beam/beam.png               <- aim-line beam core (beam_laser_soft_01), scrolled + tinted in code
  decal_circle.png            <- static world marker ring (team-tinted in code)
  arrow.png                   <- optional pointer-style marker
  sounds/ballbot_ping_0[1-5].wav  <- ATLAS ping SFX (P-body = pitch-shift in code)
```
(36 files. rings/ + beam/ are the real P2 particle textures — see reference/particle_effects.md
for how the game layers/rotates/scrolls them and how we reconstruct that.)

## Host on GitHub raw (recommended — on StarfallEx's default whitelist)
1. Create a **public** GitHub repo, e.g. `p2-ping-assets`.
2. Upload the **contents** of this `hosting/` folder to the repo root (so you get
   `wheel/`, `sounds/`, `decal_circle.png`, `arrow.png` at the top level of the repo).
3. Your raw base URL is:
   `https://raw.githubusercontent.com/<USER>/<REPO>/main/`
   (use your default branch name; it's usually `main`).
4. Sanity-check one in a browser — this must show the image:
   `https://raw.githubusercontent.com/<USER>/<REPO>/main/wheel/generic_look.png`
5. In `../starfall/portal2_ping.txt`, set:
   `base = "https://raw.githubusercontent.com/<USER>/<REPO>/main/",`   (keep the trailing slash)

That single base makes every existing CFG path resolve:
- `wheel/generic_look.png`  → the SLOTS icons
- `decal_circle.png`        → CFG.circleImg
- `arrow.png`               → CFG.arrowImg
- `sounds/ballbot_ping_01.wav` → CFG.pingSound

## Notes
- **Case-sensitive:** raw.githubusercontent.com is case-sensitive; all files here are lowercase and
  match the CFG/SLOTS strings exactly. Don't rename.
- **Whitelist:** `raw.githubusercontent.com` and `i.imgur.com` are both on StarfallEx's default URL
  whitelist (`material.urlcreate` / `bass.loadURL`). Imgur works for images but is awkward for the
  folder structure and for the `.wav` — GitHub raw is cleaner for this bundle.
- **Sounds must use the `"3d"` flag** in `bass.loadURL` (already set in the code) or non-HUD players
  won't hear them (2D audio is HUD-gated).
- **Randomize the ping SFX** (optional): the game picks one of ballbot_ping_01..05 at random. To match,
  set `pingSound` per-play to `"sounds/ballbot_ping_0"..math.random(1,5)..".wav"`.
- Only `generic_look` + the 8 slot icons are used by the default wheel; the rest are hosted so you can
  reassign SLOTS to any authentic ping icon without re-hosting.

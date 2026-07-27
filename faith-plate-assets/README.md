# faith-plate-assets

Portal 2 faith plate ("catapult") assets for the StarfallEx chip.

`faith.zip` is fetched by the chip at runtime the same way the gun and prop bundles are — one zip,
inflated client-side by `lib_unzip`, cached under a version key.

## What is IN the bundle

`faith.zip` ships the **large 128-unit plate only** (7 entries, ~1.8 MB zipped):

| Path | What |
|---|---|
| `faith.txt` | manifest — parts, pivots, skins, anims, sound (see the file's own header) |
| `models/faith_plate_128.obj` | 35 named objects, one per MDL bone, indices globally continuous |
| `anims/faith_plate_128_anims.lua` | baked per-frame bone deltas, `BindPose` + `straightup` |
| `textures/faith_plate.png` | body diffuse |
| `textures/faith_plate_glass.png` | glass diffuse, alpha preserved (`$translucent 1`) |
| `textures/arm_64x64_exterior.png` | auxiliary texture from the model's material list |
| `sounds/launch.wav` | launch sound, 16-bit PCM **mono** 22050Hz |

## What is published but NOT bundled

The **small plate** stays here at the GitHub path for reference and future use, but is deliberately
left out of `faith.zip` so clients don't download ~1 MB the chip doesn't use:

- `models/faith_plate.obj` — 9 named objects (body, paddle, pend1, pend2, lock, clip, backpiston,
  frontpiston, glass), 5574 tris
- `models/faith_plate_<part>.obj` — the same 9 parts as standalone files, superseded by the
  combined OBJ above but kept as the split's raw output
- `anims/faith_plate_anims.lua` — its 4 sequences (`idle`, `angled`, `straightup`, `fast`)
- `textures/faith_plate_error.png` — the small plate's second skin family; the 128 has only one

To bundle any of it later, add the path to `BUNDLE` in `tools/make_faith_bundle.py`. The bundler
uses an explicit include list precisely so dropping a file in this folder can never silently
inflate every client's download.

## Where it came from

All verified against the retail game — nothing here is guessed.

**Models** — `models/props/faith_plate_128.mdl` (MDL v49, 39 bones, 2 sequences, 1 skin family,
16710 verts / 16180 tris) and `models/props/faith_plate.mdl` (23 bones, 4 sequences, 2 skin
families). The 128 panel carries **four plate mechanisms**: its moving-bone rotation signature
(106.15 / 151.83 / 57.37 / 54.05 / 18.54 / 17.71°) repeats once per mechanism.

**In-game wiring** — there is no `prop_faith_plate` entity in retail Portal 2; searching the server
DLL for "faith" finds nothing because internally it is called **Catapult**. The plate is a
`prop_dynamic` with `DefaultAnim "idle"` and `solid 0`, the launch is a `trigger_catapult`, and
three `logic_relay`s fire the sound and the animation together:

| relay | animation | when |
|---|---|---|
| `relay_normal` | `angled` | standard arcing launch |
| `relay_fast` | `fast` | quick launch (15f @ 24fps) |
| `relay_up` | `straightup` | vertical launch — the sequence the 128 model has |

Read out of `maps/mp_coop_catapult_1.bsp`'s entity lump, cross-checked against
`maps/sp_a2_catapult_intro.bsp`.

**Sound** — the `ambient_generic` plays soundscript `Metal_SeafloorCar.BulletImpact`, which
`scripts/game_sounds_physics.txt` resolves to `doors/heavy_metal_stop1.wav` at `CHAN_AUTO` /
`SNDLVL_NORM` / volume `0.70`; the SP `ambient_generic` uses radius `1250`. Already mono, so GMod's
3D positional audio (bass needs mono) plays it as-is.

**Placeholder materials** — the 128's material list also carries Maya names (`blinn2/3/4/6`,
`lambert1`) that have no `.vmt`/`.vtf` in the game at all. They are interior geometry that is never
seen; only `faith_plate`, `faith_plate_glass` and `anim_wp/arm_64x64_exterior` resolve to real files.

## Animating without a HUD

Both models are split per bone specifically so the chip can animate them with **holograms**, which
every player sees without needing `render.hud` — unlike anything drawn in a render hook.

`mesh.createFromObj()` parses the `o` blocks natively and returns a **table keyed by object name**,
so the chip makes one call and indexes the result by part name. Each part is one hologram; each
frame, take that part's track from the anims file and rotate the part about its pivot from
`faith.txt`. Track names and object names match 1:1.

(Indices in these OBJs are globally continuous across blocks, which is what the parser expects —
the per-part files each restart at 1, so they cannot simply be concatenated. The build tools handle
the rebasing.)

## Rebuilding

```bash
py porting/faith_plate/tools/bake_faithplate.py
py porting/faith_plate/tools/build_faith128.py
py porting/faith_plate/tools/combine_faith_obj.py
py porting/faith_plate/tools/make_faith_bundle.py
```

Bump `CFG.bundleVersion` in the chip after any change, or clients keep serving the cached copy.

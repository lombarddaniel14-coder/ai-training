# Marin Kitagawa — the figure IN COLOUR

**Deliverable:** `C:\Users\Daniel\ForgeGen\out\marin-FINAL-COLOR.glb` — 10,322,620 bytes
**Comparison image:** `C:\Users\Daniel\ForgeGen\out\marin-COLOR-comparison.png`
**Date:** 2026-08-05 · colour assembler · CPU only, no GPU run
**Companion:** `out/FINAL-REPORT.md` (the grey figure this is derived from)

---

## 1. What was done

The geometry recipe from FINAL-REPORT §3 was **ported, not redesigned**: pre-deform the body so
its cut ring hides inside the bust → graft the approved head → seal → merge. Every stage now runs
with `--uv`, so TEXCOORD_0 and the source materials survive end to end.

The body is **not** `marin-body-canonical.glb`. That mesh carries POSITION only and can never be
coloured. The body is the generation agent's textured winner, `out/marin-body-COLOR.glb`
(trellis.cpp, res 512, seed 2024), uniformly scaled ×1.98964 into the canonical coordinate space.
Because that is a different mesh — 145k tris vs 947k, hollow vs solid, trellis vs Hunyuan — **every
recorded parameter was re-measured rather than assumed.** §2 is the diff.

**Five builds, each judged on a render I opened and looked at:**

| build | change | verdict |
|---|---|---|
| C1 | graft with no pre-deform | Front locks and hair curtain stand proud of the head's envelope; a hard flat cut edge on the lock is visible in profile. Rendered at `color-final/rnd/c1-neck-right-3x.png`. |
| C2 | pre-deform, `--strand 0.35` (the canonical value) | **Worse.** The front locks are sucked onto the throat and read as a clothes-hanger clamped round the neck. `color-final/rnd/c2-neck-front.png`. |
| C100 | `--strand 1.00` (no lock pull) | Locks hang correctly but keep a hard horizontal cut edge at the top, visible in profile. `color-final/rnd/C100-r.png`. |
| C075 | `--strand 0.75` | **Ship.** Locks curve in under the head's hair, no ledge, no collapse. `color-final/rnd/C075-f.png`, `C075-r.png`. |
| CS44 | same body, head `--scale 0.4412` instead of 0.4762 | **Ship.** Wins on measured IoU and aspect — see §2. |

### Reproduction — the exact five commands

```
node color-final\xform.mjs  out\marin-body-COLOR.glb  color-final\work\body-scaled.glb ^
     --scale 1.98964 --dy -0.00119 --tex out\marin-body-COLOR.glb

node final\pre-deform.mjs  color-final\work\body-scaled.glb  color-final\work\body-cpd075.glb ^
     --ycut 0.70 --y0 0.63 --neckX 0.86 --neckZ 0.80 --hairBack 0.062 --y0Hair 0.52 ^
     --strand 0.75 --strandDist 0.022 --y0Strand 0.54 --fill --uv ^
     --tex color-final\work\body-scaled.glb --seed -0.0525,-0.0372

node graft-head.mjs --body color-final\work\body-cpd075.glb ^
     --head out\marin-head-canonical.glb --outdir color-final\graft --name CS44 ^
     --bodyCut 0.70 --cutZ 1 --keepHair --scale 0.4412 --stray 800 --hairTaper 0.78 ^
     --uv --bodyFill --bodySeed -0.0525,-0.0372

node final\seal.mjs  color-final\graft\CS44\assets\body.glb  color-final\work\body-sealed.glb ^
     --name body-sealed --uv --tex color-final\work\body-cpd075.glb

node final\merge.mjs  out\marin-FINAL-COLOR.glb ^
     color-final\work\body-sealed.glb  color-final\graft\CS44\assets\head.glb ^
     --tex "color-final\work\body-cpd075.glb,out\marin-head-canonical.glb"
```

### Code changes, all additive and all proved additive

| file | change | proof it is additive |
|---|---|---|
| `graft-head.mjs` | new `--bodyFill` flag: measure the body's neck with holes filled | Re-running the shipped F6 command with no new flags reproduces `body.glb` sha1 **06121e05** and `head.glb` sha1 **0f919377** — byte-identical to the hashes in the UV agent's record. |
| `final/pre-deform.mjs` | new `--uv` / `--tex` (carry TEXCOORD_0, transplant the texture), `--neckZ` (default 1 = old behaviour), `--fill` | With defaults the only touched lines are no-ops: `loadGlbGeometryAuto` **is** `loadGlbGeometry` (same binding in lib.mjs), `keepUv:false` is the loader's own default, `fillHoles:false` is `rasterSection`'s own default, `--neckZ 1` is guarded by `if (KNECKZ !== 1)`, and `exportGlb(…, {})` matches its `opts = {}` default. Re-running the R5 parameters on the canonical body reproduces FINAL-REPORT §3's stated result exactly (hair section moved to z[-0.2138, -0.1082] against the documented target −0.052 shift; neck section 0.0911 × 0.0831 against F6's measured 0.09083 × 0.08342). The output differs from `final/work/body-pd5.glb` by **4 bytes** — the JSON mesh-name string, which `exportGlb` takes from the output filename. |
| new: `color-final/xform.mjs`, `probe2.mjs`, `solidity.mjs`, `iou2.ps1`, `sample2.ps1`, `montage.ps1`, `crop.ps1`, `rr.ps1` | measurement + figure-building only | new files, nothing else imports them |

Nothing under `Claude Stuff\Projects\Tools\Forge 3D` was modified; it was only `cd`-ed into to run
the read-only render harness.

---

## 2. What I changed from the recorded recipe, and why

Every row is a measurement, not a preference.

| parameter | grey F6 | colour | measured reason |
|---|---|---|---|
| body input | `marin-body-canonical.glb` | `marin-body-COLOR.glb` ×1.98964, dy −0.00119 | The scale is not a guess: both generators normalise to a unit cube, so ×1.98964 lands the trellis body at bbox y −0.9997 .. 0.9972 against the canonical body's −0.9998 .. 0.9970, and x/z centres within 0.0006. |
| `--bodyCut` | 0.69 | **0.70** | At 0.69 this body's neck blob is 0.1338 × 0.0924 — still fused with the trapezius. At 0.70 it is a clean isolated blob, 0.0945 × 0.0874. At 0.71 it fuses with the hair curtain (3 blobs instead of 5). 0.70 is the only clean plane. |
| `--bodySeed` | `-0.024,-0.099` (default) | **`-0.0525,-0.0372`** | The default seed sits 0.027 from this body's **hair** blob and 0.062 from its **neck** blob — it picks the hair and the whole graft lands on the wrong feature. |
| `--bodyFill` | (did not exist) | **on** | The trellis body is a hollow shell, so its neck cross-section is an **annulus**. Unfilled it measures 0.00322 (the wall); filled it measures 0.00481 (the neck). Without this the head comes out ~25 % too small. The Hunyuan body is solid, which is why the flag defaults off. |
| `--neckX` | 0.79 | **0.86** | Body neck 0.0945 wide must end up inside the placed head's 0.0856. 0.0945 × 0.86 = 0.0813. |
| `--neckZ` | (did not exist; = 1) | **0.80** | New, and needed only here. The canonical body's neck is 0.0832 **deep** against the placed head's 0.0888 — it already fitted. This body's is 0.0864 deep against 0.0818 — it does not. 0.0864 × 0.80 = 0.0691. |
| `--y0` | 0.60 | **0.63** | Shorter ramp so the shoulders are not narrowed on the way up. Fold criterion `1.5·(1−k)·r/span` = 0.64 < 1, so it still cannot fold. |
| `--hairBack` | 0.052 | **0.062** | This body's hair-curtain front edge is at z = −0.0349 and the placed head's hair front edge is at z = −0.1021, i.e. **0.067 proud** (the canonical body was 0.049 proud). |
| `--y0Hair` | 0.58 | **0.52** | Longer band for the bigger shift: shear 1.5 × 0.062 / 0.18 = 0.52. |
| `--strand` | 0.35 | **0.75** | Rejected 0.35 and 1.00 on renders — see the C2 / C100 rows in §1. This is the one parameter I tuned by eye rather than by envelope arithmetic, because both failure modes are cosmetic, not geometric. |
| `--scale` | 0.4762 | **0.4412** | Silhouette IoU against the official front art: **0.6469** at 0.4412 vs **0.6359** at 0.4784. Aspect H/W 3.290 vs 3.324 against the art's 3.243. 0.4412 also lands the figure at exactly **6.06 heads**, the official-art proportion `graft-head.mjs` itself cites. |
| `--y0Strand`, `--strandDist`, `--hairTaper`, `--stray`, `--cutZ`, `--capUV` | — | unchanged | 0.54, 0.022, 0.78, 800, 1, (0,0). |

---

## 3. The final figure — measured

`out/marin-FINAL-COLOR.glb`

| metric | colour | grey `marin-FINAL.glb` |
|---|---|---|
| triangles | 267,574 | 955,416 |
| welded vertices (1e-5) | 127,815 | 472,436 |
| **boundary edges** | **0 — WATERTIGHT** | **0 — WATERTIGHT** |
| non-manifold edges | 5,568 | 4,892 |
| components | 4 | 2 |
| vertex attributes | **POSITION, NORMAL, TEXCOORD_0** | POSITION, NORMAL |
| materials | **2** (body bake, head bake) | 1 (clay) |
| bbox (native) | −0.3129, −0.9997, −0.2235 .. 0.3060, 1.0359, 0.1511 | −0.3281, −0.9998, −0.2349 .. 0.3218, 1.0526, 0.1675 |
| size (native) | 0.6189 × 2.0357 × 0.3746 | 0.6499 × 2.0524 × 0.4024 |
| volume | 0.022613 native = **15.6 cm³ at 180 mm** | 0.076907 native = 51.9 cm³ |
| proportion | **6.06 heads** (crown→neck) | 5.66 heads |
| **front-silhouette IoU vs the art** | **0.6481** | 0.6250 |
| aspect H/W (art = 3.243) | 3.292 | 3.161 |

**Per component** (`final/split-audit.mjs`, weld 1e-5):

| # | what | tris | verts | boundary | non-manifold | signed volume |
|---|---|---|---|---|---|---|
| 1 | head (canonical, placed) | 142,092 | 67,341 | **0** | 2,508 | +0.002200 |
| 2 | body (cut, sealed) | 113,532 | 54,502 | **0** | 3,055 | +0.030169 |
| 3 | left-leg interior cavity wall | 6,182 | 3,093 | **0** | 0 | −0.004884 |
| 4 | right-leg interior cavity wall | 5,754 | 2,879 | **0** | 0 | −0.004873 |

Components 3 and 4 have **negative** signed volume because they are inward-facing cavity walls, not
extra solids — that is the correct sign for a void and it is what makes the net volume 0.022613.
They ship inside `marin-body-COLOR.glb`; nothing in this assembly created them.

**It stands.** Centre of mass (−0.0479, 0.0870, −0.0127); footprint at +0.2 mm is 181.6 mm² across
4 blobs spanning x[−0.164, 0.061] z[−0.090, 0.121]. The CoM projects **inside** it in both X and Z.
The contact patch is smaller than the grey figure's 321 mm² (this figure has slimmer legs and
narrower shoes), but it is a real footprint and it contains the CoM with margin.

**The seal worked exactly as designed.** The cut left 3,877 open directed edges in 490 loops; the
centroid fan closed all of them, 0 open edges after, 11,631 fan corners at the cap texel (0,0).

---

## 4. Colour accuracy — measured against the reference art

Mean RGB over matched named regions of the front view. Left column is
`refs/marin-v2/front.png`, right is the 2400 px Forge render of the deliverable
(`color-final/rnd/mfc-hi/r/mfc-hi.front.png`). Sampler: `color-final/sample2.ps1`.

| feature | OFFICIAL ART | marin-FINAL-COLOR | verdict |
|---|---|---|---|
| hair crown | rgb(245,204,168) lum 210 | rgb(202,145,118) lum 155 | **blonde ✓** — right warm order R>G>B, but darker and more orange (G−B 27 vs the art's 36) |
| hair side | rgb(206,172,140) lum 177 | rgb(151,101, 81) lum 110 | same, more pronounced |
| pink tips | rgb(179,121,126) lum 134 | rgb(139, 49, 67) lum 69 | **present and unmistakable ✓**, correct hue family (R>B>G in both), but a saturated crimson rather than the art's soft coral |
| white shirt / collar | rgb(246,227,216) lum 230 | rgb(233,203,187) lum 208 at the throat opening; rgb(110,116,131) where shaded | **partial** — reads white only where lit; elsewhere a pale blue-grey smear |
| red tie | rgb(146, 77, 80) lum 92 | rgb(121, 55, 53) lum 69 | **✓** clearly red, R-dominant, correctly placed under the collar |
| navy blazer (body) | rgb( 51, 60, 88) lum 60 | rgb( 22, 21, 45) lum 23 | **NAVY ✓** — B−R = +23 (the generation agent's key test; 6 of 8 seeds failed this) |
| navy blazer (sleeve) | rgb( 55, 65, 96) | rgb( 32, 30, 59) | **✓** B−R = +27 |
| black vest | rgb( 53, 57, 68) | rgb( 38, 38, 42) | **✓** neutral dark, correct |
| light-blue skirt | rgb(151,179,193) lum 174 | rgb(101,112,133) lum 111 | **light blue ✓** at the hem (B>G>R in both), **no plaid ✗** |
| bare leg / skin | rgb(237,220,208) lum 223 | rgb(189,171,156) lum 174 | **✓** correct warm skin hue |
| dark socks | rgb( 50, 49, 49) | rgb( 32, 36, 26) | **✓** |
| brown loafers | rgb( 62, 55, 52) | rgb( 75, 50, 47) | **✓** — actually a more convincing brown than this sample of the art |
| choker | black band | black band present (from the head bake) | **✓** |

**Every single item on the brief is present and correct in hue.** Nine of twelve are clean hits;
the shirt is a partial hit; the skirt is right in colour and missing its plaid; the hair is right in
family and off in tone.

**One systematic caveat, stated up front:** everything in the render is 40–60 luminance units below
the flat art. That is the render's analytic lighting on a shaded 3D surface versus flat cel art, not
a texture defect — the *relationships* (blazer blue-biased, tie red-dominant, skirt blue-biased,
hair warm) all hold, and those are what the eye reads. All numbers on both sides of the table come
from the same measurement path, so the comparison is like-for-like on hue, not on absolute
brightness.

### Written critique, errors largest first

1. **The head's hair and the body's hair are different colours, and you can see the join.**
   In the back view the head's hair measures rgb(152,109,89) lum 117 and the body's hair, hanging
   below it, rgb(120,90,65) lum 95 — the head is ~23 % brighter and redder. It reads as a horizontal
   tonal band across the shoulder blades. This is the deepest defect in the figure and it is **not
   fixable on the CPU**: the two halves came from two separate bakes, and matching them means
   re-generating the head with the body's palette, which is a GPU job.
2. **No plaid on the skirt.** Flat light blue with pleat shading and no grid lines. The generation
   agent already measured the cause — a 1024 atlas over a whole figure puts the plaid grid below
   texel resolution, and `--res 1024` makes the bake worse, not better. No seed fixes this; it needs
   the skirt generated separately from a cropped reference.
3. **The skirt is dark over its top half.** The vest colour has bled down the atlas, so from the
   front the skirt reads as a dark band above a light-blue hem rather than light blue throughout.
   Inherited from the winning bake and flagged by the generation agent before I started.
4. **The chest is smeared.** Collar, shirt and the top of the vest are a soft blue-white mush rather
   than a crisp collar over a knotted tie. The tie and the collar are both *there* and both the right
   colour; they are just low-resolution and blurred.
5. **The pink tips are oversaturated.** rgb(139,49,67) against the art's rgb(179,121,126) — correct
   hue, too much red, too dark. Vivid rather than soft coral.
6. **The face is the approved head's own bake, unimproved.** Pale orange-blonde rather than Marin's
   bright blonde, and the mouth is malformed. Known, documented in FINAL-REPORT §5, deliberately out
   of scope, and preserving UVs preserves it exactly rather than improving it.
7. **The hair is too short.** It ends around the shoulder blades; the settei puts it near hip length.
   Inherited from body generation, unchanged by this stage.
8. **Faint speckle across the blazer.** Small brown/orange flecks in the navy — bake noise in the
   atlas, visible only in a 2× crop.

---

## 5. Honesty bar: is the colour figure's geometry worse than the grey one's?

**No — it is different, and on the measurements available it is slightly better.** I expected the
opposite (145k triangles against 813k) and rendered both in clay with the texture stripped to check.
`color-final/rnd/geometry-compare.png` is that test.

**Where the colour body wins:**
- **Front-silhouette IoU 0.6481 vs 0.6250**, and aspect 3.292 vs 3.161 against the art's 3.243 —
  closer on both, measured with `final/iou.ps1`'s own method (it reproduces the grey figure's
  published 0.6250 exactly, so the method is not being changed underneath the comparison).
- **Real shoes.** Distinct loafers with a heel and a separate sock cuff. FINAL-REPORT §5 error 4 was
  "loafers and socks are one shallow wedge" — that defect is gone.
- **A real pleated skirt** with visible pleats and a hem, instead of a skirt that merges into the
  vest as a smock.
- **Separated fingers** on both hands.
- **Proportion 6.06 heads** against the art's 6.06, where the grey figure is 5.66.

**Where the grey body wins:**
- **Fine surface detail.** 813k triangles of Hunyuan sculpt against 125k of trellis: the grey body
  has a ribbed vest hem and crisper sleeve cuffs that the colour body simply does not resolve. Its
  surfaces are smoother and softer everywhere.
- **Solidity.** The grey body is genuinely solid — 19 % bbox fill, 6.31 mm implied wall. The colour
  body is a **hollow shell**: ray parity from four interior points returns an even crossing count at
  the hips, neck and skull, and 2V/A gives a **1.58 mm wall at 180 mm**. That is well above a 0.4 mm
  nozzle and slices fine, but it is 15.6 cm³ of material against 51.9 cm³, and a hollow figure is
  structurally weaker at the ankles.
- **Component count.** 2 clean components against 4 (the two extra are the leg cavity walls).

**Recommendation — treat them as two deliverables, not a replacement:**

- **`marin-FINAL-COLOR.glb` is the definitive figure for VIEWING and for colour printing.** It is
  what Daniel actually asked for, it is watertight, it stands, it beats the grey figure on
  silhouette accuracy, and every colour on the brief is present.
- **`marin-FINAL.glb` (grey) remains the reference for MONOCHROME FDM printing** if surface fidelity
  is the priority: solid, higher triangle density, more resolved cloth detail, larger contact patch.

If only one file can survive, ship the colour one. The geometry gap is small and runs in both
directions; the colour gap does not.

---

## 6. Print readiness

**Both stated criteria are met: 0 boundary edges overall and 0 on every one of the four components.**

**The one genuine blocker is inherited and unchanged from the grey figure.** The head component's
implied wall is **2V/A = 0.00210 native = 0.19 mm at 180 mm**, below a single 0.4 mm bead —
`marin-head-canonical.glb` is a hollow shell and always was (FINAL-REPORT §6 proves it three ways).
The remedy is identical: solidify the head component on its own (Blender *Remesh (Voxel)* or
*Solidify*, or a slicer's "make solid") and re-merge. I deliberately did not attempt it: a voxel
remesh resamples the surface, and the face is the one thing about this head that is approved.

The two halves are already on disk as separate textured GLBs, so the solidify pass has clean inputs:
- `color-final/work/body-sealed.glb` — 125,482 tris, watertight, textured
- `color-final/graft/CS44/assets/head.glb` — 142,092 tris, watertight, textured

Other notes:
- The body shell's 1.58 mm wall is printable but thin. If this is printed in monochrome FDM rather
  than resin or colour, consider solidifying the body too.
- 5,568 non-manifold edges (head 2,508 inherited from the source asset, body 3,055). Zero boundary
  edges means it slices; some engines still grumble at non-manifold edges.
- The head and body components **overlap** at the neck by design (that is what hides the seam) and
  the leg cavity walls sit inside the body. The slicer unions all of it, but the *combined* file
  self-intersects — validate the components separately, as with the grey figure.

---

## 7. Files

**Deliverables**
- `out/marin-FINAL-COLOR.glb` — the figure in colour. 267,574 tris, 4 watertight components,
  TEXCOORD_0, 2 materials.
- `out/marin-COLOR-comparison.png` — official art | grey `marin-FINAL.glb` | colour front, right,
  back, all at matched figure heights.
- `out/COLOR-REPORT.md` — this file.

**Renders (all texture-preserving unless marked clay)**
- `color-final/rnd/marin-final-color/r/marin-final-color.{front,right,back,iso}.png` — 900 px.
- `color-final/rnd/mfc-hi/r/mfc-hi.{front,right,back}.png` — 2400 px, what the colour table was
  sampled from.
- `color-final/rnd/mfc-chest.png`, `mfc-feet.png` — 4× crops: the tie/collar, and the sock cuff plus
  brown loafer.
- `color-final/rnd/geometry-compare.png` — **clay** vs clay, grey figure against colour figure. This
  is the image the §5 verdict rests on.
- `color-final/rnd/scale-compare.png` — art vs 6.06 heads vs 5.67 heads, the `--scale` decision.
- `color-final/rnd/c1-neck-{front,right}-3x.png`, `c2-neck-{front,right}.png`, `C100-{f,r}.png`,
  `C075-{f,r}.png` — the four rejected/accepted graft rounds at the neck.

**Intermediates**
- `color-final/work/body-scaled.glb` — the textured body in canonical space.
- `color-final/work/body-cpd075.glb` — pre-deformed, textured.
- `color-final/work/body-sealed.glb` — cut + sealed body half, textured.
- `color-final/graft/CS44/` — the shipped graft, with `assets/body.glb`, `assets/head.glb` and
  `report-CS44.json`. Rejected rounds at `color-final/graft/{C1,C2,C100,C075}/`.
- `color-final/repro/` — the F6 byte-for-byte reproduction that proves the code edits are additive.

**Tools written this session** (all under `ForgeGen\color-final`, all reusable)
- `xform.mjs` — uniform scale/translate of a textured GLB with the texture transplanted back.
- `probe2.mjs` — cross-sections **with holes filled**, which is what a hollow generated body needs.
- `solidity.mjs` — signed volume, area, bbox fill and ray-parity from arbitrary interior probes.
  This is what proved the trellis body is a shell.
- `iou2.ps1` — `final/iou.ps1`'s mask code verbatim with a parameterised target list.
- `sample2.ps1` — mean RGB over named regions expressed as fractions of the figure's own content
  box, so one region spec works at any render width and on the art too.
- `montage.ps1` — labelled side-by-side at matched figure heights. Note it excludes the render
  harness's 1 px rgb(236,236,240) ground line, which otherwise makes every content box full-width.
- `crop.ps1`, `rr.ps1` — crop/zoom, and build-component-then-render in one step.

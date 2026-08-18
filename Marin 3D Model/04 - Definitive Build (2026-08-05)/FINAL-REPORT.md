# Marin Kitagawa — final assembly report

**Deliverable:** `<USER_HOME>\ForgeGen\out\marin-FINAL.glb`
**Comparison image:** `<USER_HOME>\ForgeGen\out\marin-FINAL-comparison.png`
**Date:** 2026-08-05 · assembler agent · CPU only, no GPU run

---

## 1. What won, and what lost

The intended stack was **best body → proportion adjustment → approved head grafted on**.
I shipped **body + head graft**, and **dropped the proportion stage**. Both decisions were made
on renders I looked at, and both are backed by measurements below.

| stage | verdict | why |
|---|---|---|
| Body: `marin-body-canonical.glb` (winter front, octree 768, g10, seed 42) | **KEPT** | Beats `marin-hunyuan.glb` on every axis I measured. Front-silhouette IoU vs the official art 0.433 → 0.651 in my own consistent measurement. Decisive: the old body **does not stand** — its centre of mass is *behind* its foot contact patch. |
| Head graft: `marin-head-canonical.glb` onto the body | **KEPT** | The face upgrade is dramatic and there is no way to get it any other way. It costs 0.026 of silhouette IoU (0.651 → 0.625) because the approved head is bigger than the generated one. I judged the face worth more than 2.6 IoU points. |
| Proportion adjust (`adjust-proportions.mjs --bust`) | **DROPPED** | On this body it is a regression. See §4. |

**Rounds used: 5 critique rounds across 7 candidate builds** (R1–R5 for the graft, R6/R6b for the
bust). Every round was driven by a render I opened and looked at, not by a number alone.

---

## 2. The final figure

`out/marin-FINAL.glb` — 24,138,604 bytes

| metric | value |
|---|---|
| triangles | 955,416 |
| welded vertices (1e-5) | 472,436 |
| components | 2 (body, head) |
| **boundary edges** | **0 — WATERTIGHT** |
| non-manifold edges | 4,892 |
| bbox (native) | -0.3281, -0.9998, -0.2349 .. 0.3218, 1.0526, 0.1675 |
| size (native) | 0.6499 × 2.0524 × 0.4024 |
| volume | 0.076907 native = **51.9 cm³ at 180 mm tall** |
| proportion | 6.33 heads (crown→chin), 5.66 heads (crown→neck) |

**Per component** (`final/split-audit.mjs`, weld 1e-5 — the same rule `glb-topology.js` audits with):

| component | tris | verts | boundary | non-manifold | volume | implied wall 2V/A |
|---|---|---|---|---|---|---|
| 1 — body | 813,304 | 405,095 | **0 (watertight)** | 2,378 | 0.074141 | 0.0719 native = **6.31 mm** (solid) |
| 2 — head | 142,092 | 67,341 | **0 (watertight)** | 2,508 | 0.002767 | 0.00226 native = **0.20 mm** (shell — see §6) |

Each piece is exported separately at `final/parts/part1.glb` and `final/parts/part2.glb`.

**It stands.** Centre of mass (-0.0503, 0.1091, -0.0157); footprint at +0.2 mm above the base is a
single 321 mm² patch spanning x[-0.164, 0.051] z[-0.082, 0.155]. The CoM projects **inside** it in
both X and Z. For contrast, `marin-hunyuan.glb` has its CoM at z = -0.049 while its footprint sits
at z[+0.016, +0.106] — the CoM is *behind* the feet, so the old figure falls over backwards. That is
the same backward lean the generation agent saw in profile, now measured.

**Legs are not fragile.** Thinnest cross-section is the ankle at y = -0.80: equivalent diameter
0.0590 native = **5.2 mm at a 180 mm print**. The earlier "print-fragility risk" flag on the legs is
not supported at this scale — I would not thicken them, and thickening would move the silhouette
away from the reference, which draws her legs slim.

---

## 3. The graft — 5 rounds, and the thing nobody had measured

The graft agent's earlier claim of "no visible neck seam" is **not supported by their own renders**:
`graft/rnd/finalbust/r/finalbust.front.png` shows a hard white rectangle at the throat, and the
profile shows a flat plate at the shoulder. The same defect appeared on my first build.

**Root cause, measured** (`final/blobs.mjs`, res 1024, at the cut plane y = 0.69, world units):

```
body neck    x[-0.1118,  0.0024]  z[-0.0742,  0.0075]
bust neck    x[-0.1010, -0.0085]  z[-0.0795,  0.0093]   -> body 0.011 proud EACH SIDE in X
body hair    x[-0.1857,  0.0744]  z[-0.1618, -0.0564]
bust hair    x[-0.2190,  0.1087]  z[-0.2212, -0.1058]   -> body hair front edge 0.049 proud
body strands x[-0.1685,-0.1421] and [0.0269, 0.0519], z ~ [0.00, 0.065]  -> covered by nothing
```

Matching the necks by *equivalent diameter* (which is what the graft script does, and it does it
correctly) hides an **aspect mismatch**: the body's neck is wide and shallow, the bust's is round.
And the two hair sculptures sit at different Z. There is **no cut height that fixes this** — I
measured the two envelopes at y = 0.69 / 0.66 / 0.63 and the body section is deeper in Z than the
bust's at every one of them.

Fix: `final/pre-deform.mjs` moves the body's own material into the bust's envelope in a band below
the cut plane, classified by which blob of the cut-plane section each vertex falls nearest to —
neck → scale X, hair → translate −Z, front locks → scale radially toward the neck axis. Everything
ramps with a C2 smoothstep, per-region band lengths sized against the fold criterion
`1.5·(1−k)·r/span < 1`.

| round | build | change | verdict |
|---|---|---|---|
| 1 | R1 | graft as-is on the new body | Flat ledge at the throat, horizontal seam across the hair both sides, flat plate at the shoulder in profile. |
| 2 | R2 | full pre-deform, nearest-label regions | **Worse.** The nearest-label map handed 49,036 chest vertices to the "front lock" class and crumpled the collar into creased, torn geometry. Kept as evidence at `final/cand/R2.glb`. |
| 3 | R3 | neck X-taper only (0.79) | Throat ledge reduced. Hair seam and shoulder plate unchanged. |
| 4 | R4 | + hair back-shift (−0.052) | Shoulder plate in profile **gone**. Front locks still show flat cut faces. |
| 5 | **R5** | + front locks pulled in, region limited to 0.022 native around the locks | **Ship.** Locks now curve into the neck and drape; no ledge, no plate, no crumple. |

Post-cut the body carried 1,694 boundary edges, **all of them exactly at y = 0.6900**. Cause: the
carver's `recap()` triangulates with earcut, and the hair curtain produces self-intersecting
in-plane contours that earcut silently only partly triangulates (it does not throw, so
`failedContours` was 0 and the failure was invisible). `final/seal.mjs` closes every residual loop
with a centroid fan, which is topologically exact by construction and cannot leave a hole no matter
how self-intersecting the contour is. 635 loops, 4,346 fan triangles, 0 open edges after. The cut
face is buried inside the head, so the in-plane overlap a fan can make is never visible.

---

## 4. Why the proportion stage was dropped

`adjust-proportions.mjs --measure` on the grafted figure:

```
head height 252.9 mm (6.33 heads)   bust line t=31.6%
bust projection past abdomen: 5.7 mm = 0.022 head heights   (CONVEX)
```

This is a different problem from the one that script was calibrated on. The old body
(`marin-hunyuan.glb`, loose summer shirt) was measured at **−0.040 head heights — actually concave**.
The new winter-uniform body is already convex at +0.022. The response is also much stronger here:

| push | projection | head heights | det(J) | folded |
|---|---|---|---|---|
| 0 (shipped) | 5.7 mm | 0.022 | — | — |
| +11 mm (R6b) | 16.6 mm | 0.066 | 0.9999 .. 1.4014 | 0 |
| +20 mm (R6) | 25.5 mm | 0.101 | 0.9903 .. 1.5159 | 0 |

The maths is fine and nothing folds. **The renders are the problem.** Compare
`final/rnd/r5/hi/torso.png` (shipped) with `final/rnd/r6b/hi/torso.png` (+11 mm): the deformer's
ellipsoid produces **two discrete round mounds with a hard circular crease around each**, pushing
through an open school blazer. The official front art shows no bust definition through that blazer
at all. It reads as an obvious edit, which is exactly the thing to avoid. The +11 mm variant is
still on disk at `final/cand/R6b.glb` if a different judgement is wanted; I do not recommend it.

Honest caveat: the 0.065–0.15 head-height target that stage was calibrated against was derived from
a *summer-uniform* official sculpt with a thin top. Applying it under a blazer is not the same
measurement, and I do not think the number transfers.

---

## 5. Measured against the reference art

Silhouette IoU, front view, each mask cropped to its own content box and resampled into a common
300×800 grid (`final/iou.ps1`). **These are my numbers on my own consistent method — they are not
comparable to the generation agent's 0.62/0.87, which used a different mask source.**

| model | IoU vs `refs/marin-v2/front.png` | aspect H/W |
|---|---|---|
| `marin-hunyuan.glb` (previous best) | **0.4333** | 3.806 |
| `marin-body-canonical.glb` (body alone) | **0.6510** | 3.057 |
| `marin-FINAL.glb` (shipped) | **0.6250** | 3.161 |
| reference art | — | 3.243 |

Two honest readings of that table:

1. **The final beats the previous best by +44 % relative** (0.433 → 0.625), and its aspect ratio
   (3.161) is much closer to the art's 3.243 than the old body's 3.806.
2. **The graft costs 0.026 of IoU** versus the bare body. The approved head is larger than the
   generated one (5.66 vs 6.93 heads crown-to-neck) and its hair is wider at shoulder level. If you
   only cared about the front outline, the bare body wins. I do not think that is the right thing to
   care about — the bare body's face is a featureless blob.

### Written critique against the official front art

Reads correctly: standing straight, arms hanging clear, feet together; open blazer with lapels and
two flap pockets over a sweater vest with a ribbed hem; collared shirt and choker at the throat;
pleated skirt; bare legs; loafers. Long straight hair with front locks over the chest and a plain
blazer back — all three settei-checkable back claims (hair falls straight down, plain blazer back,
skirt reads as a simple pleated cone) hold.

Errors, largest first:

1. **Hair is too short.** It ends in ragged strand tips around the shoulder blades. The official
   settei puts it near hip length. This is inherited from the body generation and is not something
   the graft can fix.
2. **Front locks are too thick.** They read as two heavy tubes rather than the art's thin flat
   locks, and they end bluntly at chest level.
3. **Face detail is coarse.** Eyes are large sockets rather than resolved eyes; the mouth is
   malformed. The malformed mouth is a known, deliberately out-of-scope texture-side defect of the
   approved head.
4. **Loafers and socks are one shallow wedge** — no heel, no vamp, no sock cuff.
5. **A faint horizontal line survives at the outer edge of the hair** at the cut plane, where the
   two hair arcs curve forward at different rates. Roughly 1 mm at print scale. Visible only in a
   3× crop of a 2400 px render (`final/rnd/r5/hi/neck-front.png`), not at 1×.
6. **Calves are near-cylindrical** with little muscle definition.

---

## 6. Print readiness

**The bar is met on the stated criterion: both components are watertight (0 boundary edges).**

**But there is one genuine print blocker, and it is inherited, not introduced.**
`marin-head-canonical.glb` is a **hollow shell**. This is now proved three ways, not asserted:

- signed volume 0.026356 in a 0.628 bbox = **4.20 % fill**, surface area 10.97 (the whole *body* is
  2.36) → implied wall 2V/A = 0.00481 native;
- ray-parity from four points along the skull's centre axis returns **6 crossings at every one of
  them — even, i.e. the interior of the cranium is void, not material**;
- at the delivered scale that wall is **0.20 mm**, below a single 0.4 mm nozzle bead.

The body is genuinely solid (19.08 % bbox fill, 6.31 mm implied wall). Nothing in this assembly
caused the shell; it ships inside the approved head asset.

**Remedy, and it is one pass:** solidify `final/parts/part2.glb` on its own — Blender *Remesh
(Voxel, ~0.15 mm at print scale)* or *Solidify*, or a slicer's "make solid" — then re-merge with
`final/parts/part1.glb` using `final/merge.mjs`. I deliberately did **not** attempt a voxel remesh
here: it resamples the surface, and the one thing about this head that is approved is the face.

Other print notes:

- 4,892 non-manifold edges (body 2,378, head 2,508). The head's 2,508 are **identical to the source
  asset's** — inherited, not introduced. The body's 2,378 are up from the source body's 1,116; the
  increase comes from welding at 1e-5 and from the seal fans. Zero boundary edges means it slices;
  some engines still grumble at non-manifold edges.
- The two components **overlap** where the bust sinks into the body. That is intentional and correct
  for printing (the slicer unions them), but it means the *combined* file self-intersects. Validate
  the two parts separately — both are clean.
- No texture. The body was generated shape-only (the Hunyuan paint models are not staged) and the
  head's texture is dropped in the merge. Geometry-only, as every previous stage was.

---

## 7. Files

**Deliverables**

- `out/marin-FINAL.glb` — the figure. 955,416 tris, 2 watertight components.
- `out/marin-FINAL-comparison.png` — reference art | original fused body | final, front, labelled.
- `out/FINAL-REPORT.md` — this file.
- `final/parts/part1.glb`, `final/parts/part2.glb` — body and head as separate solids (head needs
  solidifying before slicing).

**Renders**

- `final/rnd/marin-final/r/marin-final.{front,right,back,iso}.png` — 900 px, clay.
- `final/rnd/marin-final/tt/_turntable-sheet.png` — 8-step turntable contact sheet.
- `final/rnd/marin-final/tt/marin-final.turn-{000..315}.png` — the individual turntable frames.
- `final/rnd/r5/hi/`, `r1/hi/`, `r2/hi/`, `r3/hi/`, `r4/hi/` — 2400 px renders and the 3× neck crops
  each round was judged on. `r2/hi/neck-front.png` is the crumpled regression.
- `final/rnd/r5/hi/torso.png` vs `final/rnd/r6b/hi/torso.png` — the bust-adjust decision.

**Tools written this session** (all under `ForgeGen`, all reusable)

- `final/pre-deform.mjs` — region-classified body pre-deform so the cut ring hides inside the bust.
- `final/seal.mjs` — centroid-fan closure of residual boundary loops; guaranteed watertight.
- `final/merge.mjs` — merge GLBs, one component per input, no cross-welding.
- `final/split-audit.mjs` — per-component printability audit incl. volume and implied wall.
- `final/blobs.mjs`, `final/coverage.mjs`, `final/covermap.mjs` — envelope comparison between two
  meshes at matched world heights; this is what located the seam.
- `final/stand.mjs` — centre of mass vs footprint.
- `final/diag-head.mjs` — the shell proof (volume/area + ray parity).
- `final/boundary-where.mjs` — locates boundary edges by Y band.
- `final/iou.ps1`, `final/compare.ps1`, `final/crop.ps1` — measurement and figure-building.
- `graft-head.mjs` — extended additively with `--body`, `--head`, `--outdir`, `--bodySeed`,
  `--headSeed`. Existing defaults are unchanged, so previous runs still reproduce.

**Candidates kept** — `final/cand/R1..R5.glb` (graft rounds), `R6.glb`/`R6b.glb` (rejected bust
variants), `final/graft/F1..F6/` (per-round graft reports and parts).

---

## 8. Bottom line

The final is **substantially, not marginally, better than the starting body** — a real face instead
of a blob, the correct winter uniform instead of the summer one, an upright figure that will
actually stand on a base instead of one whose centre of mass falls behind its feet, and a costume
that reads as blazer-over-vest-over-pleated-skirt instead of a smock.

It is **not** better than the bare `marin-body-canonical.glb` on front silhouette alone (0.625 vs
0.651) — the grafted head is bigger than the generated one. That is the one measurable thing the
graft costs, and it is stated here so it is not discovered later.

The single highest-value next step is **solidifying the head**, which is a one-pass job on
`final/parts/part2.glb` and is the only thing standing between this file and a slicer.

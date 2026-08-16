# Alya (Roshidere) — 3D figure in colour: assembly report

**Subject:** Alisa Mikhailovna Kujou, Seiren Academy winter uniform, single-view reconstruction.
**Deliverable:** `C:\Users\Daniel\ForgeGen\out\alya-FINAL.glb` (textured, 2 materials)
**Comparison image:** `C:\Users\Daniel\ForgeGen\out\alya-comparison.png`
**Render evidence:** `C:\Users\Daniel\ForgeGen\out\alya-FINAL-renders\` (10 sheets)
**Assembled by:** the assembler agent, 2026-08-05. CPU only — no GPU work, no regeneration.
**Licensing:** copyrighted character art. Personal-use print/display only. Never publish, never sell.

---

## 1. The headline call: the graft ships, and it is the right call

The generation agent delivered a body whose **colour and silhouette are good and whose face is a
failure** — a smooth beige mound with one ghost-blue eye and no left eye, no nose, no mouth. On a
character figure the face is the likeness; a blank face is a reference-accuracy failure of the
first order, not a cosmetic one.

The graft buys a real face. It costs a small amount of hair-mass fidelity at the crown, a slight
hair-colour shift, and one residual seam artifact visible only under magnification. **That trade is
clearly worth taking.** See `alya-FINAL-renders\03-bodyonly-vs-graft-fullfigure.png` — the two
figures side by side at matched height, front / right / back.

Where the body-only version genuinely wins: **the back**. Body-only has one continuous pale hair
mass down the back; the graft has the head's hair mass ending over the body's hair with a faint
horizontal break. If Daniel only ever looks at this figure from behind, body-only is better. From
any other angle the graft wins outright.

---

## 2. What was tried, and what lost

Nine grafts were built and rendered. Every one was judged on renders, not numbers.

| run | head donor | method | verdict |
|---|---|---|---|
| r1 | s2026 | measured scale 0.3654 | **LOST** — 3.88 heads tall. Bobblehead. The auto-anchor bailed (Alya's bust section *grows* with height, so the bisection's monotonic-band assumption is inverted) and the measured neck ratio was simply wrong for this pair. |
| r2 | s2026 | headCut −0.315, measured 0.2950 | **LOST** — 4.29 heads. Still oversized. |
| r3 | s2026 | scale forced to 0.2596 | Correct size (4.74 heads = the body's own). **LOST on colour** — slate-grey hair against the body's pale lilac, plus a red collar smear. |
| r4 | s1234 | headCut −0.32, scale 0.2578 | Best neck match of the set (ratio 1.021). **LOST on a shelf** — the head's rebuilt cut cap is a wide horizontal plate under the chin, clearly visible from below. `10-cut-variant-shelf-defect.png`. |
| r5 | s2026 | headCut −0.25 (to escape the collar smear) | **LOST** — neck ratio 0.771, a stepped joint, and the shelf returned. |
| r6 | s1234 | `--keepHair` (bust uncut) | Shelf under the chin gone; replaced by flat wings at the shoulders (the bust's own crop plane). |
| r7 | s1234 | `--keepHair --hairTaper 0.30` | Wings tucked in. Good front. **Still a dark flat plate across the back of the hair.** |
| r8 | s2026 | same settings as r7 | Head-donor A/B at matched settings. **LOST** — confirmed s1234's hair colour matches the body and s2026's does not. `04-head-donor-AB.png`. |
| **FINAL** | **s1234** | **r7 + `--dy −0.030`** | **WON.** Sinking the head 3 % of figure height buries the bust's bottom plane inside the body, kills the back plate, and lands the proportion at 5.35 heads — the closest of any run to the reference's ≈5.3. |

**The head-donor decision reversed the generation agent's pick, deliberately.** It shipped
`WINNER-head-alya-s2026-512.glb` because that head has genuinely carved eye sockets. Grafted, s2026
loses on two things that matter more at figure scale: its hair bakes slate-grey against the body's
pale silver-lilac (a visible two-tone break across the whole silhouette), and its collar region
bakes to a red/dark smear with shard artifacts. `RUNNERUP-head-alya-s1234-512.glb` has paler hair
that reads continuous with the body, a clean white collar, both red ribbons, and the cleanest
ahoge. Its faults — eyes that are a painted crease rather than a carved socket, and a faint vertical
"muzzle plate" seam down the nose — are invisible past about 2× zoom. Colour fidelity was a
first-class requirement here; s1234 serves it and s2026 does not.

Cost of that choice, stated plainly: **the final head carries 6,453 non-manifold edges against
s2026's 1,652.** The figure is still watertight (0 boundary edges) but this is the worse mesh.

---

## 3. Final recipe (reproducible)

```
node graft-head.mjs
  --body C:\Users\Daniel\ForgeGen\out\alya\FINAL\WINNER-body-alya-s99-512.glb
  --head C:\Users\Daniel\ForgeGen\out\alya\FINAL\RUNNERUP-head-alya-s1234-512.glb
  --outdir C:\Users\Daniel\ForgeGen\alya\work --name alya-final
  --bodyCut 0.29  --bodySeed -0.0068,0.0411  --cutZ 0.5
  --headCut -0.32 --headSeed -0.0081,0.0199
  --scale 0.2578 --keepHair --hairTaper 0.30 --dy -0.030
  --noAutoAnchor --bodyFill --uv
node final\seal.mjs   <part>  <out>  --tex <that part's ORIGINAL glb>      # per part, never combined
node final\merge.mjs  out\alya-FINAL.glb  sealed\body.glb sealed\head.glb  --tex <body>,<head>
```

Every parameter above was **re-measured on Alya**, none copied from Marin:

- `--bodyCut 0.29` / `--bodySeed` — read off `neck-scan.mjs --fill`. The neck is the compact
  centre-front blob (area 0.00315, eqDia 0.0633) sitting between the two hair-curtain blobs.
  Note: at y = 0.30 and 0.31 the neck blob is *cleaner* but its area falls below `pickNeck`'s
  0.002 floor and the script throws — 0.29 is the highest usable plane without editing the tool.
- `--cutZ 0.5` — past the body's max z. **Alya has no raised arm** (her left hand is on her hip at
  y ≈ 0.05, far below the neck plane), so the half-space hazard that forced Marin's 0.105 does not
  exist here. Verified: the `discarded generated head` component list is 1 component, 41,272 tris,
  nothing else.
- `--bodyFill` ON — `volcheck.js` puts the body at **3.92 % fill**, a hollow shell, so its neck
  section is an annulus. Confirmed, not assumed.
- `--scale 0.2578` **overridden** — the measured 0.2987 produces a 4.3-heads bobblehead. Overridden
  only after a render said so, per the skill's rule.
- `--capUV` left at (0,0) — **probed, not assumed**: the body atlas at (0,0) is `#B0A2AE` (pale
  hair lilac) and the head atlas is `#5E6673` (slate hair). Both are plausible hair colours and both
  caps are buried, so no override was needed. This was luck again, but measured luck.
- Stray components: the body cut dropped exactly one 384-tri component, bbox
  `(−0.048, 0.234, −0.121) .. (−0.005, 0.290, −0.067)` — a hair tip behind the left ear, buried
  under the head. **Not the ahoge and not a ribbon**; both of those live on the head part and are
  intact. Checked in `report-alya-final.json` before accepting, as the runbook demands.

---

## 4. Measured stats

**`out\alya-FINAL.glb`**

| | |
|---|---|
| triangles | 256,322 |
| welded vertices | 118,844 |
| components | 3 (body shell, one detached body hair strand, head shell) |
| **boundary edges** | **0 — watertight** |
| non-manifold edges | 7,234 (781 body + 6,453 head; all inherited, none introduced) |
| degenerate tris | 0 |
| attributes | POSITION, NORMAL, **TEXCOORD_0** |
| materials | **2**, both with a bound baseColorTexture — `tex-audit` reports `TEXTURED=YES` on both primitives |
| images / textures | 4 / 4 (EXT_texture_webp) |
| native bbox | 0.2990 × 0.9726 × 0.2929 |
| unitScale @ 200 mm | 205.63 |
| proportion | neck-to-crown 18.7 % → **5.35 heads tall** (reference art ≈ 5.3; body-only was 4.74) |
| volume | 24.3 cm³ at 200 mm |

**Hollowness (`volcheck.js`, per part, the number that actually matters):**

| part | fill % | implied wall 2V/A native | wall at 200 mm |
|---|---|---|---|
| body | 3.91 % | 0.00491 | **1.01 mm** |
| head | 3.78 % | 0.00115 | **0.24 mm** |

**Stability (`stand.mjs`):** centre of mass (−0.0087, 0.0582, 0.0296). At a 0.4 mm slice height the
contact patch is a single 7.3 mm² toe blob and the CoM is **outside** it in both X and Z. At 1.0 mm
both feet are down, patch 26.4 mm², CoM inside. **She is marginal — she will stand on a flat table
but she is one nudge from going over backwards.** A base is recommended.

---

## 5. Colour accuracy assessment

Judged against `refs\alya\front-1.png` and `refs\alya\uniform-sheet.jpg`.

| element | reference | model | verdict |
|---|---|---|---|
| hair | silver-white with a faint pink cast | silver-lilac, correct hue but **noticeably darker in value** | **acceptable** — hue is the strongest reason s99 was kept as the body; the darkness is real |
| blazer | cream / light greige | warm cream-taupe, **darker in value** | **acceptable**, a shade warmer and darker than reference |
| jumper + skirt | near-black charcoal | near-black charcoal | **correct** |
| skirt hem stripes | white double stripe | white double stripe, continues around the back | **correct** |
| bow | wine / crimson | crimson-rose with a white centre | **good** — measured within ~17/255 of the reference on every channel (see below) |
| blouse collar | white | white | **correct** |
| cuffs | white with a gold band | white with a gold band | **correct** |
| socks | white, thin gold band mid-thigh | white, gold band mid-thigh | **correct** |
| loafers | dark brown | dark brown | **correct** |
| hair ribbon | red, at the left temple | red, reads on both temples | **correct**, arguably over-delivered |
| eyes | bright blue | blue with a lash line and blush | **good** |
| skin | pale | pale, slightly muddy across the cheeks | **acceptable** |
| back of thighs | bare skin | **a pink/red flush painted across the backs of the thighs** | **wrong** — a bake artifact, inherited, unfixable without a repaint |

### Sampled, not eyeballed

Pixel values, reference art vs the final lit render (front view):

| | reference | model | note |
|---|---|---|---|
| bow | `#AF2B50` (175, 43, 80) | `#C02D5A` (192, 45, 90) | **very close.** The generation agent's "hot magenta" warning does not survive contact with the assembled render — the bow is a crimson-rose, marginally brighter and pinker than the reference wine. **No atlas hue-shift was needed and none was applied.** |
| hair | `#DFD7DB` (223, 215, 219) | `#8E839A` (142, 131, 154) | hue correct (lilac-silver), **value clearly darker** |
| blazer | `#B6AAAA` (182, 170, 170) | `#958280` (149, 130, 128) | hue correct (warm greige), **value darker** |

**Caveat, stated so the table is not read as worse than it is:** the reference is flat cel art with
no shading and the model is a lit render, so mid-tone darkening is expected and part of that gap is
lighting, not bake. The *hues* match. The *values* run dark, and that is a real and visible
characteristic of this figure: she reads as a slightly dusky Alya rather than the near-white one in
the official art.

There is **no colour seam at the graft join**, which was the main risk of grafting at all. That is
the direct payoff of switching donor from s2026 to s1234.

---

## 6. Sculpt-warning checklist — every item, checked on a render

Evidence: `09-sculpt-warning-checks.png` (clay) and `01-final-4view-plus-clay.png`.

| warning | result |
|---|---|
| **Ahoge survives?** | **YES.** A clean free-standing loop above the crown, present in all 8 turntable frames and in the clay render. It is the head donor's ahoge, not the body's — thinner and closer to the reference than the body's fat cone. |
| **Red ribbon tails?** | **YES**, on both temples, in colour. |
| **Hair volume correct?** | **NO — the single biggest miss.** Reference hair runs past the hips; the model stops around mid-back. True of the body-only version too and of all ten seeds the generation agent produced. Not fixable in assembly. |
| **No false thigh gap?** | **CORRECT.** Thighs meet, legs cross slightly, exactly as the reference. |
| **Sock band present?** | **YES**, as a painted gold band at the right mid-thigh height, over a soft compression step — not a hard ledge. Correct. |
| **Two-layer torso?** | **PARTIAL.** The open cropped blazer over the jumper bodice reads as a soft lapel edge in clay. **The double-breasted gold buttons and the lapel notch are not sculpted at all.** They exist at 1024 (see `GEOMETRY-body-alya-s1234-1024.glb`) but that mesh's texture is unusable. |
| **Hand-on-hip loop not webbed?** | **CORRECT.** Clear daylight between the left forearm and the torso in both clay and colour. |
| **Both loafers?** | **YES**, both present and shod; the far one is invented and holds up under the turntable. |
| **Mouth?** | Painted only, no geometry — as documented and as intended. |

---

## 7. Remaining flaws, honestly

1. **Hair length.** Reference: past the hips. Model: mid-back. This is the flaw that most costs the
   likeness and it is a generation-level limit — I cannot fix it without a GPU run, which is not
   mine to make.
2. **The skirt rides up at the back.** At matched figure height the *front* hem sits at very nearly
   the reference height — that part is right. But the back hem is shorter and exposes the buttocks,
   with the pink bake flush painted across them. It is the worst thing about the figure from behind.
   Cloth cannot be extended by a deformer, so this is generation-level.
   Related: the skirt is also **less flared and less pleated** than the reference's crisp box
   pleats. Those pleats exist on `GEOMETRY-body-alya-s1234-1024.glb`; they do not survive at 512.
3. **A residual seam plate at the back of the head.** Sinking the head by 0.030 removed the hard
   dark shelf, but a faint translucent horizontal band is still visible across the back hair at
   roughly 3× zoom (`08-residual-seam-zoom.png`). At normal viewing distance it reads as a shading
   change, not an artifact. It is the head bust's own bottom crop plane and no taper can remove it,
   because the taper weight is zero exactly at the mating plane.
4. **The join is an overlap, not a fusion.** With `--dy −0.030` the "neck match" number is
   meaningless (the VERIFY line reports ratio 1.847 because it is measuring the jaw, not the neck).
   The head and body are two interpenetrating watertight shells. That is fine for viewing and fine
   for slicing, but the figure is 3 components, not 1, and I want that on the record rather than
   dressed up as a matched-diameter joint.
5. **The neck is almost invisible.** The head sits low enough that the chin nearly meets the bow.
   The reference shows a short but visible neck and collar. This was the price of burying the seam.
6. **No sculpted facial geometry.** The clay render shows an essentially featureless face — the
   eyes, brows, blush and mouth are all texture. Expected and documented, but it means the figure
   only looks like Alya *in colour*. A clay/resin print of this mesh would have a blank face.
7. **7,234 non-manifold edges.** Watertight is necessary, not sufficient; some slicers reject
   non-manifold geometry. Both numbers are reported above rather than just the flattering one.
8. **Torso/limb proportion drift.** At matched height (see `alya-comparison.png`) the model's torso
   is shorter and narrower than the reference and the arms are longer and thinner, so she reads
   leggier and more slender overall. Some of the torso compression is my doing — sinking the head
   0.030 shortened the neck-and-shoulder run. The rest is generation.
9. **The back is invented.** No back or side reference exists for this character. Judged against
   `uniform-sheet.jpg` the blazer, skirt and hem stripes read correctly from behind; the hair length
   and the thigh flush do not.

## 8. Stages not dropped, and one that was not attempted

Every stage in the runbook ran: measure → graft → render → taper/sink (in place of pre-deform) →
seal per part → merge with per-part textures → audit the artifact → render again → turntable.

- **Pre-deform was deliberately not used.** The runbook says it is a fix for a seam you have
  actually seen. The seam I saw was a cap plate at the mating plane, which pre-deform does not
  address — it reshapes the body *below* the cut. `--keepHair` + `--hairTaper` + `--dy` addressed it
  directly. Marin needed 5 pre-deform rounds; Alya needed 0.
- **The seal ran per part and never on the combined file** — the documented trap that silently
  collapses 2 materials to 1 and garbles the face. Both parts reported `open directed edges
  before: 0` and `after: 0`; no fan triangles were needed, so `capUV` never became visible.
- **The atlas hue-shift the generation agent proposed for the s99 bow was not performed** and is not
  needed. In the assembled render the bow reads crimson, not magenta.

---

## 9. Print-readiness verdict

**Not print-ready as-is. Fine to view.**

- **Blocker:** the head is a **0.24 mm double-walled hollow shell** at 200 mm — well under the
  0.4 mm FDM nozzle. It passes every topology check and will still fail on the plate. A solidify /
  voxel remesh on the **head part alone**, done last, is mandatory. Re-render the face afterwards:
  a remesh resamples the exact surface that was approved here.
- **Body wall is 1.01 mm** — sliceable.
- **Thin features below the wall thickness:** the ahoge loop and the ribbon tails. Check them
  explicitly after any remesh; they are the first things a voxelizer eats.
- **Stability is marginal** (CoM outside the 0.4 mm contact patch). Print with a base.
- **Non-manifold edges** may be rejected by some slicers.

For Forge 3D / on-screen viewing, which is what this pipeline optimises for, it is ready now:
schema-v3 mesh part, `unitScale` 205.63 for a 200 mm figure, `bbox` and `triangles` as measured
above.

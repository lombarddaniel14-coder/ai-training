---
type: progress-archive
tags: [ai-training, forge3d, marin, image-to-3d]
created: 2026-08-05
---

# Marin 3D Model — Progress Archive

Every render of the Marin Kitagawa model, in the order it happened. Folders are
chronological; files inside them are numbered.

## 00 - References
The input images. Everything downstream is determined by these — see the face
experiment for how much.

## 01 - Primitive Era (2026-08-03)
Built entirely from math primitives (boxes, spheres, swept tubes). The hair went
through three render-and-critique rounds, which genuinely improved it. The
ceiling was never iteration — it was vocabulary: **you cannot make a human face
out of primitives.** Ended at 56 hand-placed parts. Honest verdict at the time:
a LEGO-minifig interpretation of Marin, not Marin.

## 02 - Generation Era (2026-08-05)
Forge 3D v1.4 added real mesh parts, and a local image-to-3D model was run on the
RTX 3080 for the first time. Reference art in → sculpted mesh out in 36 seconds.

Two failures worth keeping, because the diagnosis is the useful part:
- **03/04** — at 1024 resolution the model rendered black. Geometry was fine; the
  *texture bake* had collapsed into mud (file 04 is the actual atlas — that's the
  evidence).
- **05** — after fixing the material, 1024 was still dark, which is what proved
  the fault was the bake and not the material.
- **06** — 512 + the metallic fix. The breakthrough frame.

Generated GLBs come out `metallicFactor: 1` (fully metallic). Metal with no
environment to reflect renders black. `ForgeGen\fix-generated-glb.js` patches it.

Two engines were compared. Both produce watertight, printable meshes:
| | Hunyuan3D | TRELLIS |
|---|---|---|
| triangles | 505,714 | 146,188 |
| detail | far finer | coarser |
| texture | none (shape-only) | yes |
| time | ~140 s | ~36 s |
| best for | **printing** | **on-screen preview** |

## 03 - Face Experiment (2026-08-05)
Answering "why is the face bad?" — with a test rather than a theory.

Identical engine, identical settings, identical 512 grid. The **only** change was
the input image: the full-body character sheet swapped for the 440×440 face
close-up. Result: sculpted eye sockets with lash lines, a real nose, lips,
layered hair strands, ears, a choker.

**The bottleneck is resolution allocation, not model capability.** In a full-body
run the face gets a few thousand pixels of the input and roughly 60 voxels of
head height in the output grid. Generate the head alone and both jump by about an
order of magnitude, because the entire grid is spent on the head.

The path forward is therefore: generate body and head separately at full
resolution each, then graft the head on — which is exactly what the Carver is for.

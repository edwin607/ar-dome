# AR Dome — artwork augment

Point your phone camera at the painting and a low-poly gazebo in the same blue
as the artwork's linework anchors to it in AR — with a low-poly cat chasing a
mouse inside.

**Live:** https://edwin607.github.io/ar-dome/

## How it works
- [MindAR](https://github.com/hiukim/mind-ar) image tracking (three.js build, pinned to three@0.150.1) anchors the scene to the artwork.
- `artwork.mind.gz.*` — the compiled image target, gzipped and chunked; the page reassembles and decompresses it in-browser (`DecompressionStream`, pako fallback).
- Gazebo struts are tubes matching the painting's stroke weight, colored from the sampled linework blue (`#2a3670`); mustard junction accents.
- Camera + HTTPS required — open on a phone for AR. Camera denied (or desktop) → orbit-controls preview against the artwork.

## Repository layout
- `index.html` — the whole AR experience (single file, CDN dependencies)
- `artwork.mind.gz.aa–ad` + `artwork.mind.manifest.json` — chunked image target
- `artwork-target.jpg.aa–ac` — chunked artwork photo (preview fallback ground plane)
- `tools/` — the pipeline: compile a new target, chunk binaries, smoke-test the page
- `SKILL.md` — full pipeline documentation (ar-image-augment agent skill)

## Retargeting to a new image
1. Crop the new artwork photo tight to its edges, ≤ ~900px long side → `artwork-target.jpg`
2. `uv run --with playwright python tools/compile_target.py artwork-target.jpg --out artwork.mind`
3. `gzip -9 -c artwork.mind > artwork.mind.gz && python tools/chunk.py artwork.mind.gz artwork.mind.gz --manifest artwork.mind.manifest.json`
5. Rectify first if the photo has any perspective: warp the 4 canvas corners to the physical aspect ratio (current target: 900×600, a 3:2 / 6×4" canvas)
6. Update `TARGET_H` and the focal-point pixel in `ANCHOR_X` / `ANCHOR_Z` in `index.html`; `python tools/chunk.py artwork-target.jpg artwork-target.jpg --manifest artwork-target.manifest.json`
7. `uv run --with playwright python tools/smoke_test.py <url> --fake-video cam.y4m --shot out.png`, require `errors: none`; add `?debug` to the URL to see the tracked outline

## URL params
- `?debug` draws the tracked target outline (should hug the canvas edges)
- `?scale=0.5` scales the augment on-device (0.1-3), handy for tuning size without a redeploy

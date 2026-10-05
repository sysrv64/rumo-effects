# Rumo effects

Example repository for the **Effects** section of the Rumo shop. It carries the
`rumo-effects` topic, so Rumo finds it through the GitHub search API.

## What is here

| Path | What it is |
|---|---|
| `rumo-manifest.json` | what this repository offers: name, version, which Rumo it is for, where the files are |
| `rumo-verification.json` | the result of Rumo's own check that the content works |
| `effects/crt-scanlines.rumoeffect` | effect 1 |
| `effects/rgb-split.rumoeffect` | effect 2 |
| `previews/*.webp` | the small pictures shown in the shop list |

### CRT Scanlines

Dark bands between phosphor rows, a horizontal wobble that grows towards the
edges and drifts with time, and a vignette.

| Parameter | Range | Default |
|---|---|---|
| Lines | 20 – 600 | 220 |
| Depth | 0 – 1 | 0.5 |
| Wobble | 0 – 0.03 | 0.006 |
| Vignette | 0 – 1 | 0.45 |

**This is the one to try first if you want to know whether a custom effect runs
at all**: all three of its parts are visible on a flat colour, so a shape layer
shows them. RGB Split does not — chromatic aberration needs texture detail, and
a shape layer is one flat colour.

### RGB Split

Chromatic aberration. Red is sampled a little further from the frame centre and
blue a little closer, while green stays put; the displacement grows with distance
from the centre, so the middle stays sharp and the edges fringe. A second
parameter darkens the corners.

| Parameter | Range | Default |
|---|---|---|
| Split | 0 – 0.06 | 0.02 |
| Vignette | 0 – 1 | 0.35 |

One pass, no downscale, so it costs one full-resolution draw.

## An effect is a WGSL module

`.rumoeffect` is JSON with the effect under `effect`:

```json
{
  "kind": "rumo-effect",
  "formatVersion": 1,
  "name": "RGB Split",
  "effect": {
    "id": "rgb_split",
    "label": "RGB Split",
    "space": "display",
    "passes": [{ "entry": "fs_main", "shrink": 0 }],
    "params": [{ "key": "amount", "label": "Split", "kind": "float",
                 "min": 0.0, "max": 0.06, "default": [0.012] }],
    "source": "...WGSL..."
  }
}
```

The module declares exactly five bindings and no vertex stage:

```wgsl
@group(0) @binding(0) var input_tex: texture_2d<f32>;
@group(0) @binding(1) var input_smp: sampler;
@group(0) @binding(2) var<uniform> frame: Frame;      // size, time, pass_index, texels
@group(0) @binding(3) var origin_tex: texture_2d<f32>; // the untouched layer
@group(1) @binding(0) var<uniform> params: Params;
```

`struct Params` must list the declared parameters **in the same order**, as
`f32`, `vec2<f32>`, `vec3<f32>`, `vec4<f32>`, `mat3x3<f32>` or `mat4x4<f32>`.
The host supplies the vertex stage, so `uv` has a top-left origin and can be fed
straight into `textureSampleLevel`. Colour maths is in display space
(gamma-encoded 0–1) and layer textures are straight alpha.

## Publishing your own

1. Put a `rumo-manifest.json` and a `rumo-verification.json` in the repository
   root, plus the content file and a **webp or avif** preview for each item.
2. Add the `rumo-effects` topic to the repository.
3. Keep it to four effects per repository and two shop repositories per account.

Rumo shows a repository only when the verification file is present, every item in
it passed, the declared files exist, and the previews are webp or avif. Rumo also
re-runs the WGSL validator on install and refuses a module it cannot compile, so
a broken effect cannot reach the driver.

The verification file is produced by Rumo's own checker. The one in this
repository was produced on the host with the engine's own `naga` front end
(`validate_source`); Rumo re-checks on install.

## Licence

MIT — see [LICENSE](LICENSE). Use this module for anything, including
commercially.

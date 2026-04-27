# Henrietta Manual Align

A small browser tool for the **early days of using Henrietta** — when
on-sky operations don't yet have a polished alignment loop and you need
a quick, reliable way to get an acquisition star onto the slit.

The user takes an acquisition image, reads off the star's pixel position
and the current PA, and the tool reports either:

- **Telescope offset** (″) — `E/W` and `N/S`, with the cardinal direction
  baked into the value (no signed numbers to mis-read in the dark), or
- **Guider arrow presses** — using an empirical 2×2 calibration matrix,
  rotated to the current PA. Toggle with the *Guide box move* checkbox.

The slit (20″ wide) and surrounding FOV (26′ × 3.18′) are drawn so you
can see exactly where the star is relative to the slit and target
crosshair. A compass rose tracks the current PA.

## Use

Open `Henrietta_Manual_Align.html` in any modern browser. No build
step, no dependencies — it's a single file.

1. Pick the **target pixel** (slit center vs. center of curvature).
2. Enter the **star X / Y** read off the acquisition image.
3. Enter the current **PA** in degrees.
4. Read the offset (or, with the checkbox on, the press count).

## See also

- [`SPEC.md`](SPEC.md) — geometry, sign conventions, the calibration
  matrix, and the PA-rotation math.

## Calibration assumption

The guider matrix in the HTML was fit at PA = 0 (constant
`PA_CAL_DEG` near the top of the script). If your calibration was done
at a different PA, change that constant. If the guide camera rotates
*with* the science detector, remove the rotation step entirely.

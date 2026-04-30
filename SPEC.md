# Hen Slit Alignment Tool — Spec

## Purpose
Self-contained HTML page (`Henrietta_Manual_Align.html`). The user enters a star's pixel
position from an acquisition image plus the current PA. The tool reports
how to move the telescope (or, optionally, the guide-box) so that the
star ends up on a chosen target pixel.

## Geometry
- Detector: 2048 × 2048 pixels, origin top-left.
- Plate scale: 0.776 ″/pix.
- FOV (illuminated strip): 26′ × 3.18′ (X × Y), centered on the slit
  at (995, 1005).
  - Half-widths: 13′ ≈ 1004.6 px in X; 1.59′ ≈ 122.9 px in Y.
- Slit:
  - Centered at (995, 1005).
  - Length: full FOV in X (≈ 2010 px = 26′).
  - Width: 20″ ≈ 25.77 px in Y.

## Target pixel (dropdown)
The "Center pixel" selector chooses where the star should land:

| Option                          | Pixel       | Default |
| ------------------------------- | ----------- | ------- |
| Center of 20″ slit              | (995, 1005) | yes     |
| Center of curvature             | (995, 941)  |         |

The slit itself is always drawn at (995, 1005); only the target
crosshair / arrow / pixel-offset reference move with the dropdown.

## PA / sky convention
- Image axes: +X = right, +Y = down (top-left origin).
- **PA = 0** → sky North along +X (right).
- **PA = 90** → sky North points up (−Y).
- East = 90° CCW from North on the sky (so at PA = 0, E = up; at
  PA = 90, E = left).
- Unit vectors on the screen:
  - N̂ = ( cos PA, −sin PA )
  - Ê = ( −sin PA, −cos PA )

## Inputs
- `Star X` (pix) — placeholder shows the currently-selected center X.
- `Star Y` (pix) — placeholder shows the currently-selected center Y.
- `PA` (degrees).
- `Guide box move` (checkbox) — switches the offset panel from "Telescope
  offset (sky)" to "Guider presses".

## Output: telescope offset (default)
Both axes shown, rounded to 0.1″, with the cardinal direction baked in.
A negative value flips the direction label, so the user never reads a
signed offset:

- `E/W (″)` — e.g. `11.6 E`, `4.3 W`, `0.0`.
- `N/S (″)` — e.g. `7.2 N`, `2.8 S`, `0.0`.
- `|Δ| (″)` — total offset magnitude.

Math (Xs, Ys = star pixel; (cx, cy) = selected target; s = 0.776):

    dx_as = (Xs - cx) * s
    dy_as = (Ys - cy) * s
    ΔE   =  sin(PA)*dx_as - cos(PA)*dy_as
    ΔN   =  cos(PA)*dx_as + sin(PA)*dy_as

Sign convention before formatting: positive ΔE = slew east, positive
ΔN = slew north.

The sin signs above are the negation of what `V · Ê` / `V · N̂` give
under the on-screen compass convention. Empirically (LCO, 2026-04), the
LCO offset command takes PA with the opposite sign from the compass —
i.e., the offset rotation uses −PA while the compass and guider use +PA.
The HTML implements this by negating PA inside `compute()`.

## Output: guide-box presses (when checkbox is on)
Uses an empirical 2×2 calibration matrix that maps guider arrow presses
to *star pixel motion on the detector*:

    M = [[ 0.725, -0.10  ],
         [-0.0375, -0.65 ]]

Convention: `gx +1` = "right" press, `gy +1` = "up" press.

The matrix is treated as calibrated at `PA_CAL_DEG = 0` (constant near
the top of the script — change it if calibration was actually done at a
different PA). At any other PA the field has rotated relative to the
mechanically-fixed guider, so the desired star motion is rotated by
`-(PA − PA_CAL_DEG)` before applying `M⁻¹`:

    M_eff(PA)        = R(PA − PA_cal) · M
    presses(PA)      = M⁻¹ · R(−(PA − PA_cal)) · (target − star)
    output           = "<n> right|left, <n> up|down"  (rounded to ints)

Input convention to `pixelsToPresses` is *motion from the star to the
target* (i.e. `(cx − Xs, cy − Ys)`, the negative of the displayed pixel
offset), matching the user's reference Python function.

If the guide camera rotates *with* the science detector (on-rotator),
this PA term should be removed — set the matrix to be valid at all PAs.

## Visualization
Single canvas, "Slit region", 800 × 380 px:

- Background + 1-px border.
- FOV strip (light blue fill, blue outline).
- Slit (semi-transparent green fill, green outline).
- Target crosshair (green when target = slit; amber when target = center
  of curvature).
- Star marker (red dot) at the entered (Xs, Ys), with a dashed line
  from star to target.
- Compass rose (top-right): N (red) and E (blue) arrows rotated by PA;
  S, W in grey for reference.
- Corner labels: `(0,0) ↖` top-left (origin is off-canvas, up-left);
  `Y ↓` bottom-left; `X →` bottom-right.
- Footer note: "Y stretched ×N for visibility" (the Y axis is stretched
  relative to X so the slit is thick enough to read).

## Layout
- Single HTML file, inline CSS/JS, no external libraries.
- Two-column layout: controls + outputs on the left, canvas on the right.

## File
`Henrietta_Manual_Align.html` — open in any modern browser.

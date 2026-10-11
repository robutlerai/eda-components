# Generated 3D models

Generated on 2026-10-11 with KiCad's own parametric CadQuery generators. Nothing here is
hand-edited. Layout is the library's: `<Library>.3dshapes/<Name>.step` and `.wrl`.

## Toolchain

- Generator repository: https://gitlab.com/kicad/libraries/kicad-footprint-generator
- Commit: `a903a695bf4e69905e9e56548f44dd46cdc1130e` (2026-10-10, shallow clone)
- Python 3.12.11 (`/opt/homebrew/bin/python3.12`, in `../venv`)
- CadQuery 2.6.1, cadquery-ocp 7.8.1.1.post1 (OpenCascade 7.8.1), numpy 2.5.4, nlopt 2.11.0
  (the repository's `3d` extra: `pip install -e './kicad-footprint-generator[3d]'`)
- Run through `../kgen.py`, which calls the repository's `src/generators/generate.py`
  with `-j 1 --export-vrml`. The reason for the wrapper is at the top of that file: at
  this commit the generator's own VRML export crashes, and the wrapper repairs the call
  without changing the clone.

## Files

| File | Generator script | Parameters |
| --- | --- | --- |
| `Package_SO.3dshapes/SOIC-8_3.9x4.9mm_P1.27mm.step` (115,173 bytes), `.wrl` (79,176 bytes, 1,392 triangles) | `src/generators/package/gullwing/model.py` | upstream entry `SOIC-8_3.9x4.9mm_P1.27mm` in `data/package/gullwing/soic.yaml`, unchanged |
| `Package_BGA.3dshapes/BGA-64_8x8mm_Layout8x8_P0.8mm.step` (71,793 bytes), `.wrl` (1,689,367 bytes, 69,288 triangles) | `src/generators/package/grid_array/model.py` | our entry `BGA-64_8x8mm_Layout8x8_P0.8mm` in `../packages/package/grid_array/robutler.yaml` |

BGA parameters (from the EDA app's `PKG_DIMS.BGA`): body 8.0 x 8.0 mm, body height 1.2 mm,
gap to the board 0.36 mm (overall 1.56 mm), 8 x 8 balls (64), pitch 0.8 mm, ball diameter
0.5 mm.

Measured from the generated files:

- SOIC-8: 6.000 x 4.902 x 1.999 mm. The library's own file (kicad-packages3D 8.0.9) is
  6.000 x 4.902 x 1.750 mm with the same three materials. See the note below.
- BGA-64: body 8.000 x 8.000 mm, top at 1.562 mm, one fused solid, 64 balls on a 0.8 mm
  grid from -2.8 to +2.8 mm in both axes.

Note on the SOIC-8 height: today's upstream entry gives ranges (body height 1.35 to 1.75,
gap 0.10 to 0.25) and the generator takes the maximum of each, so the body sits at 0.25 mm
and tops out at 2.00 mm. The 8.0.9 library file was built with a 0.10 mm gap and a 1.75 mm
top. Outline, lead span and materials are identical.

Note on the ball size: the generator draws each ball 5 % oversize (0.525 mm) so it fuses
with the body, and lets it reach 0.0125 mm below the board plane. That is upstream
behaviour, the same in every library BGA.

The generator stamps KiCad's library licence text (CC-BY-SA 4.0 with the design-use
exception) into the header of every file it writes, these included.

## Reproduce

```sh
cd /Users/vs/dev/eda-components-tools
/opt/homebrew/bin/python3.12 -m venv venv
git clone --depth 1 https://gitlab.com/kicad/libraries/kicad-footprint-generator.git
git -C kicad-footprint-generator rev-parse HEAD   # a903a695... for these files
./venv/bin/python -m pip install -e './kicad-footprint-generator[3d]'

./venv/bin/python kgen.py -g package/gullwing   -p 'SOIC-8_3.9x4.9mm_P1.27mm'      -m out
./venv/bin/python kgen.py -g package/grid_array -p 'BGA-64_8x8mm_Layout8x8_P0.8mm' -m out

./venv/bin/python -I wrlstat.py out/Package_BGA.3dshapes/BGA-64_8x8mm_Layout8x8_P0.8mm.wrl
```

A later clone gets a later commit; to pin this one, fetch it by hash
(`git fetch --depth 1 origin a903a695bf4e69905e9e56548f44dd46cdc1130e && git checkout FETCH_HEAD`).

## Ceramic packages (2026-10-11)

Twenty ceramic prototype packages: footprint, closed 3D model (lid on) in STEP and WRL,
land pattern as plain data, and the cavity numbers for the app's open view. Same clone,
same commit, same environment as above.

Where things are:

- `Package_DIP|LCC|QFP|PGA|DFN_QFN.3dshapes/<Name>.step` and `.wrl`: the models.
- `Package_DIP|LCC|QFP|PGA|DFN_QFN.pretty/<Name>.kicad_mod`: the footprints. The DIP
  generator always writes its four variants (plain, `_LongPads`, `_Socket`,
  `_LongPads_Socket`); only the plain one has a model and a JSON file.
- `footprints/<Name>.json`: `{name, origin, pads: [{n, x, y, w, h, shape, drill?}],
  body: {w, h, cx, cy}, courtyard: {w, h, cx, cy}}`, mm, y down. Read back from the
  `.kicad_mod`, so it cannot differ from it. `origin` is `center` for surface mount and
  `pin1` for through-hole (then `cx, cy` say where the body centre is).
- `cavities.json`: per package, die attach pad, cavity opening and depth, bond finger
  ring, lid, seal ring width, body below the cavity floor, each with its basis.
- `ceramic_check.json`: triangles and bounding box of every WRL against the data.

All numbers live in `../scripts/ceramic_data.py`, each group marked sourced, derived or
typical. Kyocera's drawings are behind a member login and were not opened: body size,
body thickness, pitch and recommended chip size come from Kyocera's public standard-parts
tables; lead and pad forms come from one named maker's drawing per family (Analog
Devices D-40-1 and J-44, TI FK, Intersil G84.A). The die attach area is Kyocera's
recommended maximum chip plus 0.40 mm, a rule that holds on the four rows where
Kyocera's older PDF lists print both numbers. Cavity depth, floor thickness, the bond
finger ring, seal ring width and most lid sizes are typical, not sourced.

Made by:

| Family | Footprint | 3D model |
| --- | --- | --- |
| Side-brazed DIP | generator `package/DIP` (8 and 16 pins: upstream entries, unchanged; 24 to 48: `../packages/package/DIP/ceramic_sidebrazed.yaml`) | `../scripts/ceramic_models.py`, which calls the DIP generator's own `make_case` and `make_pins_tht` and adds our lid (the generator only reads its tracked `cq_parameters.yaml`, which we do not edit) |
| J-leaded carrier | generator `package/PLCC`, KiCad's J-lead generator (`../packages/package/PLCC/ceramic_jlcc.yaml`) | `../scripts/ceramic_models.py`, our own CadQuery |
| Leadless carrier | by hand, `../scripts/ceramic_outputs.py`, IPC-7351 nominal (toe 0.55, heel 0.15, side -0.05) | `../scripts/ceramic_models.py`, our own CadQuery |
| CQFP | generator `package/gullwing` (`../packages/package/gullwing/ceramic_cqfp.yaml`) | generator `package/gullwing` through `../scripts/ckgen.py`; at 100 and 144 leads the body and leads are swapped for flat-sided ones |
| Pin grid array | by hand, `../scripts/ceramic_outputs.py`, origin on pin A1 | `../scripts/ceramic_models.py`, our own CadQuery |
| Open-cavity QFN | generator `package/no_lead` (`../packages/package/no_lead/ceramic_qfn.yaml`) | generator `package/no_lead` through `../scripts/ckgen.py` |

Materials are the library's: `brown body` (dark alumina), `gold pins` (leads, pads, pins,
seal ring), `metal silver` (lid), and the generator's `light brown label` pin-1 mark.

Kept light: the WRL writer's angular step is 1.0 rad instead of OpenCascade's 0.1; J
leads, carrier pads and pins are flat-sided prisms (a pin is a hexagonal prism, 20
triangles); gull-wing bends are near-sharp (radius 0.03). That was not enough at 100 and
144 leads (9,708 and 13,932 triangles, because OpenCascade keeps a minimum number of
points on every arc and the generator notches the body once per lead), so those two get
an eight-corner lead profile and a plain body: 2,940 and 4,172.

Measured with `wrlstat.py` (triangles, size in mm, expected size, result):

```
CERDIP-8_W7.62mm_SideBrazed                                    586      7.87x13.21x6.78      7.87x13.21x6.78 pass
CERDIP-16_W7.62mm_SideBrazed                                  1114      7.87x20.32x6.78      7.87x20.32x6.78 pass
CERDIP-24_W15.24mm_SideBrazed                                 1642     15.49x30.48x6.78     15.49x30.48x6.78 pass
CERDIP-28_W15.24mm_SideBrazed                                 1906     15.49x35.56x6.78     15.49x35.56x6.78 pass
CERDIP-40_W15.24mm_SideBrazed                                 2698     15.49x50.80x6.78     15.49x50.80x6.78 pass
CERDIP-48_W15.24mm_SideBrazed                                 3226     15.49x60.96x6.78     15.49x60.96x6.78 pass
JLCC-44_16.5x16.5mm_P1.27mm                                   1656     17.65x17.65x2.97     17.65x17.65x2.97 pass
JLCC-68_24.1x24.1mm_P1.27mm                                   2520     25.27x25.27x3.35     25.27x25.27x3.35 pass
JLCC-84_29.2x29.2mm_P1.27mm                                   3096     30.35x30.35x2.97     30.35x30.35x2.97 pass
CLCC-28_11.43x11.43mm_P1.27mm                                  632     11.47x11.47x1.82     11.47x11.47x1.82 pass
CLCC-44_16.51x16.51mm_P1.27mm                                  952     16.55x16.55x1.95     16.55x16.55x1.95 pass
CQFP-44_10x10mm_P0.8mm                                        4396     13.20x13.20x1.75     13.20x13.20x1.75 pass
CQFP-64_10x10mm_P0.5mm                                        6316     13.00x13.00x1.75     13.00x13.00x1.75 pass
CQFP-100_13.97x13.97mm_P0.5mm                                 2940     17.00x17.00x1.45     17.00x17.00x1.45 pass
CQFP-144_19.2x19.2mm_P0.5mm                                   4172     22.20x22.20x1.60     22.20x22.20x1.60 pass
CPGA-84_27.94x27.94mm_Layout11x11_P2.54mm                     1732     27.94x27.94x6.90     27.94x27.94x6.90 pass
CPGA-144_40.01x40.01mm_Layout15x15_P2.54mm                    2932     40.01x40.01x6.90     40.01x40.01x6.90 pass
QFN-32-1EP_5x5mm_P0.5mm_EP3.1x3.1mm_CeramicCavity             2312       5.00x5.00x0.80       5.00x5.00x0.80 pass
QFN-48-1EP_7x7mm_P0.5mm_EP5.15x5.15mm_CeramicCavity           3400       7.00x7.00x0.90       7.00x7.00x0.90 pass
QFN-64-1EP_9x9mm_P0.5mm_EP7.15x7.15mm_CeramicCavity           4488       9.00x9.00x0.90       9.00x9.00x0.90 pass
```

The expected size is computed from the data: over the leads for DIP, J-lead and CQFP;
body plus 0.02 mm of pad metal on each side wall for the leadless carrier; body for the
grid array and the QFN; height from the lead tip or the board up to the top of the lid.

Reproduce:

```sh
cd /Users/vs/dev/eda-components-tools
./venv/bin/python scripts/ceramic_yaml.py
./venv/bin/python scripts/ckgen.py -g package/gullwing -p 'CQFP-*' -m out -f out
./venv/bin/python scripts/ckgen.py -g package/no_lead -p 'QFN-*_CeramicCavity' -m out -f out
./venv/bin/python kgen.py -g package/DIP -p 'CERDIP-8_JEDEC_MS-015-AA' 'CERDIP-16_JEDEC_MS-015-AC' 'CERDIP-*_SideBrazed_Kyocera' -f out
./venv/bin/python kgen.py -g package/PLCC -p 'JLCC-*' -f out
./venv/bin/python scripts/ceramic_models.py
./venv/bin/python scripts/ceramic_outputs.py footprints json cavities check
```

### CERDIP-8_W7.62mm_SideBrazed

- Family: dip; 8 pins; library `Package_DIP`; Kyocera standard part: KD-SB9F71
- Footprint: generator package/DIP (upstream entry CERDIP-8_JEDEC_MS-015-AA, unchanged)
- Model: scripts/ceramic_models.py calling the DIP generator's make_case and make_pins_tht, plus our lid
- Parameters (mm): body (7.49, 13.21), body_t 2.16, standoff 1.02, pitch 2.54, row_spacing 7.62, lead_w 0.46, lead_shoulder_w 1.27, lead_t 0.254, lid (6.6, 10.09), lid_t 0.25, seal_w 0.3, overall height lid on 3.48
- Pin 1: notch at the pin-1 end of the body; square pad 1
- Sources:
  - body length, width over leads, body thickness, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=dip (row KD-SB9F71)
  - ceramic width: kicad-footprint-generator/data/package/DIP/cerdip.yaml (cites JEDEC MS-015A, https://www.jedec.org/system/files/docs/Ms-015a.pdf): 0.295 in
  - lead width, shoulder, thickness, standoff, lead length: https://mds.analog.com/api/public/content/D_40_1.pdf (40-lead drawing, applied to the family)
  - lid thickness: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no (Fe-Ni-Co pedestal lids are 0.25 mm)
- Typical, not sourced: lid size; seal ring width; cavity depth and floor thickness; bond finger ring; standoff nominal (middle of the drawing's 0.51 to 1.52 range)

### CERDIP-16_W7.62mm_SideBrazed

- Family: dip; 16 pins; library `Package_DIP`; Kyocera standard part: KD-SB9A87
- Footprint: generator package/DIP (upstream entry CERDIP-16_JEDEC_MS-015-AC, unchanged)
- Model: scripts/ceramic_models.py calling the DIP generator's make_case and make_pins_tht, plus our lid
- Parameters (mm): body (7.49, 20.32), body_t 2.16, standoff 1.02, pitch 2.54, row_spacing 7.62, lead_w 0.46, lead_shoulder_w 1.27, lead_t 0.254, lid (6.6, 10.6), lid_t 0.25, seal_w 0.3, overall height lid on 3.48
- Pin 1: notch at the pin-1 end of the body; square pad 1
- Sources:
  - body length, width over leads, body thickness, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=dip (row KD-SB9A87)
  - ceramic width: kicad-footprint-generator/data/package/DIP/cerdip.yaml (cites JEDEC MS-015A, https://www.jedec.org/system/files/docs/Ms-015a.pdf): 0.295 in
  - lead width, shoulder, thickness, standoff, lead length: https://mds.analog.com/api/public/content/D_40_1.pdf (40-lead drawing, applied to the family)
  - lid thickness: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no (Fe-Ni-Co pedestal lids are 0.25 mm)
- Typical, not sourced: lid size; seal ring width; cavity depth and floor thickness; bond finger ring; standoff nominal (middle of the drawing's 0.51 to 1.52 range)

### CERDIP-24_W15.24mm_SideBrazed

- Family: dip; 24 pins; library `Package_DIP`; Kyocera standard part: KD-SB9F85
- Footprint: generator package/DIP (our entry)
- Model: scripts/ceramic_models.py calling the DIP generator's make_case and make_pins_tht, plus our lid
- Parameters (mm): body (14.99, 30.48), body_t 2.16, standoff 1.02, pitch 2.54, row_spacing 15.24, lead_w 0.46, lead_shoulder_w 1.27, lead_t 0.254, lid (12.37, 12.37), lid_t 0.25, seal_w 1.0, overall height lid on 3.48
- Pin 1: notch at the pin-1 end of the body; square pad 1
- Sources:
  - body length, width over leads, body thickness, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=dip (row KD-SB9F85)
  - ceramic width: https://mds.analog.com/api/public/content/D_40_1.pdf: 0.59 in (14.99) BSC end view
  - lead width, shoulder, thickness, standoff, lead length: https://mds.analog.com/api/public/content/D_40_1.pdf (40-lead drawing, applied to the family)
  - lid thickness: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no (Fe-Ni-Co pedestal lids are 0.25 mm)
- Typical, not sourced: lid size; seal ring width; cavity depth and floor thickness; bond finger ring; standoff nominal (middle of the drawing's 0.51 to 1.52 range)

### CERDIP-28_W15.24mm_SideBrazed

- Family: dip; 28 pins; library `Package_DIP`; Kyocera standard part: KD-SB9F91
- Footprint: generator package/DIP (our entry)
- Model: scripts/ceramic_models.py calling the DIP generator's make_case and make_pins_tht, plus our lid
- Parameters (mm): body (14.99, 35.56), body_t 2.16, standoff 1.02, pitch 2.54, row_spacing 15.24, lead_w 0.46, lead_shoulder_w 1.27, lead_t 0.254, lid (12.37, 12.37), lid_t 0.25, seal_w 1.0, overall height lid on 3.48
- Pin 1: notch at the pin-1 end of the body; square pad 1
- Sources:
  - body length, width over leads, body thickness, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=dip (row KD-SB9F91)
  - ceramic width: https://mds.analog.com/api/public/content/D_40_1.pdf: 0.59 in (14.99) BSC end view
  - lead width, shoulder, thickness, standoff, lead length: https://mds.analog.com/api/public/content/D_40_1.pdf (40-lead drawing, applied to the family)
  - lid thickness: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no (Fe-Ni-Co pedestal lids are 0.25 mm)
- Typical, not sourced: lid size; seal ring width; cavity depth and floor thickness; bond finger ring; standoff nominal (middle of the drawing's 0.51 to 1.52 range)

### CERDIP-40_W15.24mm_SideBrazed

- Family: dip; 40 pins; library `Package_DIP`; Kyocera standard part: KD-SB9F95
- Footprint: generator package/DIP (our entry)
- Model: scripts/ceramic_models.py calling the DIP generator's make_case and make_pins_tht, plus our lid
- Parameters (mm): body (14.99, 50.8), body_t 2.16, standoff 1.02, pitch 2.54, row_spacing 15.24, lead_w 0.46, lead_shoulder_w 1.27, lead_t 0.254, lid (12.37, 12.37), lid_t 0.25, seal_w 1.0, overall height lid on 3.48
- Pin 1: notch at the pin-1 end of the body; square pad 1
- Sources:
  - body length, width over leads, body thickness, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=dip (row KD-SB9F95)
  - ceramic width: https://mds.analog.com/api/public/content/D_40_1.pdf: 0.59 in (14.99) BSC end view
  - lead width, shoulder, thickness, standoff, lead length: https://mds.analog.com/api/public/content/D_40_1.pdf (40-lead drawing, applied to the family)
  - lid thickness: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no (Fe-Ni-Co pedestal lids are 0.25 mm)
- Typical, not sourced: lid size; seal ring width; cavity depth and floor thickness; bond finger ring; standoff nominal (middle of the drawing's 0.51 to 1.52 range)

### CERDIP-48_W15.24mm_SideBrazed

- Family: dip; 48 pins; library `Package_DIP`; Kyocera standard part: KD-SB9F99
- Footprint: generator package/DIP (our entry)
- Model: scripts/ceramic_models.py calling the DIP generator's make_case and make_pins_tht, plus our lid
- Parameters (mm): body (14.99, 60.96), body_t 2.16, standoff 1.02, pitch 2.54, row_spacing 15.24, lead_w 0.46, lead_shoulder_w 1.27, lead_t 0.254, lid (13.4, 14.66), lid_t 0.25, seal_w 0.75, overall height lid on 3.48
- Pin 1: notch at the pin-1 end of the body; square pad 1
- Sources:
  - body length, width over leads, body thickness, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=dip (row KD-SB9F99)
  - ceramic width: https://mds.analog.com/api/public/content/D_40_1.pdf: 0.59 in (14.99) BSC end view
  - lead width, shoulder, thickness, standoff, lead length: https://mds.analog.com/api/public/content/D_40_1.pdf (40-lead drawing, applied to the family)
  - lid thickness: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no (Fe-Ni-Co pedestal lids are 0.25 mm)
- Typical, not sourced: lid size; seal ring width; cavity depth and floor thickness; bond finger ring; standoff nominal (middle of the drawing's 0.51 to 1.52 range)

### JLCC-44_16.5x16.5mm_P1.27mm

- Family: jlcc; 44 pins; library `Package_LCC`; Kyocera standard part: PB-F86090-A
- Footprint: generator package/PLCC (KiCad's J-lead generator, our entry)
- Model: scripts/ceramic_models.py (our own CadQuery)
- Parameters (mm): body (16.51, 16.51), body_t 1.65, standoff 1.02, pitch 1.27, overall (17.65, 17.65), contact_span 16.0, lead_w 0.46, lead_shoulder_w 0.66, lead_t 0.2, lid (12.48, 12.48), lid_t 0.25, seal_w 1.0, overall height lid on 2.97
- Pin 1: pin 1 in the middle of the top side, gold dot beside it, the one small corner chamfer at top left
- Sources:
  - body size, body thickness, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=qfj (row PB-F86090-A)
  - size over leads, contact row distance, lead width, standoff, corner chamfers: https://mds.analog.com/api/public/content/j-44.pdf
  - lid: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no (PGMR-A9388, 12.48 x 12.48 x 0.25; pairing with this body is our reading, the J-44 drawing shows a 12.50 to 12.70 square seal area)
- Typical, not sourced: lead thickness (0.2); standoff nominal (from the J-44 overall height); seal ring width; cavity depth and floor thickness; bond finger ring

### JLCC-68_24.1x24.1mm_P1.27mm

- Family: jlcc; 68 pins; library `Package_LCC`; Kyocera standard part: PB-F86635
- Footprint: generator package/PLCC (KiCad's J-lead generator, our entry)
- Model: scripts/ceramic_models.py (our own CadQuery)
- Parameters (mm): body (24.13, 24.13), body_t 2.03, standoff 1.02, pitch 1.27, overall (25.27, 25.27), contact_span 23.62, lead_w 0.46, lead_shoulder_w 0.66, lead_t 0.2, lid (16.8, 16.8), lid_t 0.25, seal_w 1.0, overall height lid on 3.35
- Pin 1: pin 1 in the middle of the top side, gold dot beside it, the one small corner chamfer at top left
- Sources:
  - body size, body thickness, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=qfj (row PB-F86635)
  - size over leads, contact row distance, lead width, standoff, corner chamfers: https://mds.analog.com/api/public/content/j-44.pdf (44-lead drawing, lead form reused)
  - lid: typical, not sourced
- Typical, not sourced: lead thickness (0.2); standoff nominal (from the J-44 overall height); seal ring width; cavity depth and floor thickness; bond finger ring; lid size

### JLCC-84_29.2x29.2mm_P1.27mm

- Family: jlcc; 84 pins; library `Package_LCC`; Kyocera standard part: PB-F86074
- Footprint: generator package/PLCC (KiCad's J-lead generator, our entry)
- Model: scripts/ceramic_models.py (our own CadQuery)
- Parameters (mm): body (29.21, 29.21), body_t 1.65, standoff 1.02, pitch 1.27, overall (30.35, 30.35), contact_span 28.7, lead_w 0.46, lead_shoulder_w 0.66, lead_t 0.2, lid (12.48, 12.48), lid_t 0.25, seal_w 1.0, overall height lid on 2.97
- Pin 1: pin 1 in the middle of the top side, gold dot beside it, the one small corner chamfer at top left
- Sources:
  - body size, body thickness, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=qfj (row PB-F86074)
  - size over leads, contact row distance, lead width, standoff, corner chamfers: https://mds.analog.com/api/public/content/j-44.pdf (44-lead drawing, lead form reused)
  - lid: typical, not sourced
- Typical, not sourced: lead thickness (0.2); standoff nominal (from the J-44 overall height); seal ring width; cavity depth and floor thickness; bond finger ring; lid size

### CLCC-28_11.43x11.43mm_P1.27mm

- Family: clcc; 28 pins; library `Package_LCC`; Kyocera standard part: PB-45033-C
- Footprint: hand (scripts/ceramic_outputs.py), IPC-7351 nominal
- Model: scripts/ceramic_models.py (our own CadQuery)
- Parameters (mm): body (11.43, 11.43), body_t 1.52, standoff 0.0, pitch 1.27, pad_w 0.635, pad_l 1.27, lid (9.65, 9.65), lid_t 0.25, seal_w 0.8, overall height lid on 1.82
- Pin 1: pin 1 in the middle of the top side, gold dot beside it, the large corner chamfer at top left
- Sources:
  - body size, body thickness, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=qfn (row PB-45033-C)
  - pad width, pad length, pitch, body range, index corner: https://www.ti.com/lit/ml/mlcc006b/mlcc006b.pdf (JEDEC MS-004)
  - lid: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no (PGMR-A9389, 9.65 x 9.65 x 0.25; pairing with this body is our reading)
- Typical, not sourced: seal ring width; cavity depth and floor thickness; bond finger ring; small corner chamfer (0.5)

### CLCC-44_16.51x16.51mm_P1.27mm

- Family: clcc; 44 pins; library `Package_LCC`; Kyocera standard part: PB-81840-B-01
- Footprint: hand (scripts/ceramic_outputs.py), IPC-7351 nominal
- Model: scripts/ceramic_models.py (our own CadQuery)
- Parameters (mm): body (16.51, 16.51), body_t 1.65, standoff 0.0, pitch 1.27, pad_w 0.635, pad_l 1.27, lid (12.48, 12.48), lid_t 0.25, seal_w 0.8, overall height lid on 1.95
- Pin 1: pin 1 in the middle of the top side, gold dot beside it, the large corner chamfer at top left
- Sources:
  - body size, body thickness, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=qfn (row PB-81840-B-01)
  - pad width, pad length, pitch, body range, index corner: https://www.ti.com/lit/ml/mlcc006b/mlcc006b.pdf (JEDEC MS-004)
  - lid: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no (PGMR-A9388, 12.48 x 12.48 x 0.25; pairing with this body is our reading)
- Typical, not sourced: seal ring width; cavity depth and floor thickness; bond finger ring; small corner chamfer (0.5)

### CQFP-44_10x10mm_P0.8mm

- Family: cqfp; 44 pins; library `Package_QFP`; Kyocera standard part: none
- Footprint: generator package/gullwing (our entry)
- Model: generator package/gullwing through scripts/ckgen.py (ceramic colours, lid added)
- Parameters (mm): body (10.0, 10.0), body_t 1.25, standoff 0.25, pitch 0.8, overall (13.2, 13.2), lead_w 0.37, lead_t 0.15, lead_len 0.6, lid (8.0, 8.0), lid_t 0.25, seal_w 0.5, overall height lid on 1.75
- Pin 1: circle at the pin-1 corner of the body top (generator marker), pin 1 at the top of the left side
- Sources:
  - land pattern outline (body 10 x 10, span 13.2, lead 0.29..0.45): kicad-footprint-generator/data/package/gullwing/qfp_lqfp_tqfp_vqfp_wqfp.yaml entry PQFP-44_10x10mm_P0.8mm (cites JEDEC MS-022 AB): the PLASTIC 44-lead outline; no ceramic 44-lead 0.8 mm drawing was found
- Typical, not sourced: EVERYTHING ceramic about this one: body thickness, cavity, chip size, lid (copied from the 64-lead 10 x 10 CERQUAD); lead foot length (0.6); lead thickness (0.15); standoff (0.25); lid size; seal width; cavity depth and floor thickness; bond finger ring

### CQFP-64_10x10mm_P0.5mm

- Family: cqfp; 64 pins; library `Package_QFP`; Kyocera standard part: QC-064347-WZ
- Footprint: generator package/gullwing (our entry)
- Model: generator package/gullwing through scripts/ckgen.py (ceramic colours, lid added)
- Parameters (mm): body (10.0, 10.0), body_t 1.25, standoff 0.25, pitch 0.5, overall (13.0, 13.0), lead_w 0.2, lead_t 0.15, lead_len 0.6, lid (8.0, 8.0), lid_t 0.25, seal_w 0.5, overall height lid on 1.75
- Pin 1: circle at the pin-1 corner of the body top (generator marker), pin 1 at the top of the left side
- Sources:
  - body size, body thickness, pitch, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=cerquad (row QC-064347-WZ)
  - die cavity, lead width 0.20: https://global.kyocera.com/prdct/semicon/semi/std_pkg/pdf/kyocera-pkg-cerquad-e_r0165f.pdf (search excerpt of the list; the PDF itself now redirects)
  - lid thickness: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no
- Typical, not sourced: lead span (body + 3.0); lead foot length (0.6); lead thickness (0.15); standoff (0.25); lid size; seal width; cavity depth and floor thickness; bond finger ring

### CQFP-100_13.97x13.97mm_P0.5mm

- Family: cqfp; 100 pins; library `Package_QFP`; Kyocera standard part: QC-100364-WZ
- Footprint: generator package/gullwing (our entry)
- Model: generator package/gullwing through scripts/ckgen.py, with the body and leads swapped for our flat-sided ones (ckgen.simple_gullwing), ceramic colours, lid added
- Parameters (mm): body (13.97, 13.97), body_t 0.95, standoff 0.25, pitch 0.5, overall (17.0, 17.0), lead_w 0.2, lead_t 0.15, lead_len 0.6, lid (12.2, 12.2), lid_t 0.25, seal_w 0.5, overall height lid on 1.45
- Pin 1: circle at the pin-1 corner of the body top (generator marker), pin 1 at the top of the left side
- Sources:
  - body size, body thickness, pitch, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=cerquad (row QC-100364-WZ)
  - die cavity, lead width 0.20: https://global.kyocera.com/prdct/semicon/semi/std_pkg/pdf/kyocera-pkg-cerquad-e_r0165f.pdf (search excerpt of the list; the PDF itself now redirects)
  - lid thickness: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no
  - lead span 17.00: https://global.kyocera.com/prdct/semicon/semi/std_pkg/pdf/kyocera-pkg-cerquad-e_r0165f.pdf (a 17.00 x 17.00 column in the excerpt, read as the span over leads: column heading not seen)
- Typical, not sourced: lead foot length (0.6); lead thickness (0.15); standoff (0.25); lid size; seal width; cavity depth and floor thickness; bond finger ring

### CQFP-144_19.2x19.2mm_P0.5mm

- Family: cqfp; 144 pins; library `Package_QFP`; Kyocera standard part: QC-144398-WZ
- Footprint: generator package/gullwing (our entry)
- Model: generator package/gullwing through scripts/ckgen.py, with the body and leads swapped for our flat-sided ones (ckgen.simple_gullwing), ceramic colours, lid added
- Parameters (mm): body (19.2, 19.2), body_t 1.1, standoff 0.25, pitch 0.5, overall (22.2, 22.2), lead_w 0.2, lead_t 0.15, lead_len 0.6, lid (14.4, 14.4), lid_t 0.25, seal_w 0.5, overall height lid on 1.6
- Pin 1: circle at the pin-1 corner of the body top (generator marker), pin 1 at the top of the left side
- Sources:
  - body size, body thickness, pitch, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=cerquad (row QC-144398-WZ)
  - die cavity, lead width 0.20: https://global.kyocera.com/prdct/semicon/semi/std_pkg/pdf/kyocera-pkg-cerquad-e_r0165f.pdf (search excerpt of the list; the PDF itself now redirects)
  - lid thickness: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no
- Typical, not sourced: lead span (body + 3.0); lead foot length (0.6); lead thickness (0.15); standoff (0.25); lid size; seal width; cavity depth and floor thickness; bond finger ring

### CPGA-84_27.94x27.94mm_Layout11x11_P2.54mm

- Family: cpga; 84 pins; library `Package_PGA`; Kyocera standard part: KD-P85311-A
- Footprint: hand (scripts/ceramic_outputs.py), origin on pin A1
- Model: scripts/ceramic_models.py (our own CadQuery)
- Parameters (mm): body (27.94, 27.94), body_t 2.03, standoff 1.27, pitch 2.54, pin_d 0.46, pin_len 3.3, matrix 11, lid (16.8, 16.8), lid_t 0.25, seal_w 1.0, overall height lid on 3.6
- Pin 1: A1 at the top left, large corner chamfer there, square pad A1
- Sources:
  - body size, body thickness, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=pga (row KD-P85311-A)
  - pin diameter, pin length, standoff, grid: https://www.renesas.com/en-sg/package-image/pdf/outdrawing/g84.a.pdf
  - pin layout: typical, not sourced: two full outer rings (72) plus three pins centred on each side of the third ring (12)
  - lid thickness: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no
- Typical, not sourced: pin layout inside the 11 x 11 matrix; lid size; seal ring width; corner chamfers; cavity depth and floor thickness; bond finger ring

### CPGA-144_40.01x40.01mm_Layout15x15_P2.54mm

- Family: cpga; 144 pins; library `Package_PGA`; Kyocera standard part: KD-P86499-B
- Footprint: hand (scripts/ceramic_outputs.py), origin on pin A1
- Model: scripts/ceramic_models.py (our own CadQuery)
- Parameters (mm): body (40.01, 40.01), body_t 2.03, standoff 1.27, pitch 2.54, pin_d 0.46, pin_len 3.3, matrix 15, lid (22.7, 22.7), lid_t 0.25, seal_w 1.0, overall height lid on 3.6
- Pin 1: A1 at the top left, large corner chamfer there, square pad A1
- Sources:
  - body size, body thickness, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=pga (row KD-P86499-B)
  - pin diameter, pin length, standoff, grid: https://www.renesas.com/en-sg/package-image/pdf/outdrawing/g84.a.pdf (84-lead drawing, pin form reused)
  - pin layout: https://www.topline.tv/PGA.html (CPGA144-15x15: 1.57 in square, 15 x 15, 3 rows; seen in a search listing). Kyocera's own row is the 145-pin sibling; its extra orientation pin is left out
  - lid thickness: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no
- Typical, not sourced: lid size; seal ring width; corner chamfers; cavity depth and floor thickness; bond finger ring; body is Kyocera's 145-pin part used for 144 pins

### QFN-32-1EP_5x5mm_P0.5mm_EP3.1x3.1mm_CeramicCavity

- Family: qfn; 32 pins; library `Package_DFN_QFN`; Kyocera standard part: PB-EB5643
- Footprint: generator package/no_lead (our entry)
- Model: generator package/no_lead through scripts/ckgen.py (ceramic colours, lid added)
- Parameters (mm): body (5.0, 5.0), body_t 0.7, standoff 0.0, pitch 0.5, pad_w 0.25, pad_l 0.4, ep (3.1, 3.1), lid (4.5, 4.5), lid_t 0.1, seal_w 0.3, overall height lid on 0.8
- Pin 1: generator pin-1 mark on the body top, pin 1 at the top of the left side
- Sources:
  - body size, body thickness, pitch, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=qfn (row PB-EB5643)
  - land width 0.18..0.30, land length 0.30..0.50: kicad-footprint-generator/data/package/no_lead/qfn-*.yaml (cites JEDEC MO-220, https://www.jedec.org/system/files/docs/MO-220K01.pdf)
  - lid thickness 0.10: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no (flat Fe-Ni-Co lids)
- Typical, not sourced: exposed pad (3.1 square, the size of the library's plastic footprint; whether the Kyocera part has a bottom pad was not confirmed); lid size (body - 0.5); glue seal width; floor thickness (0.25) and cavity depth; bond fingers

### QFN-48-1EP_7x7mm_P0.5mm_EP5.15x5.15mm_CeramicCavity

- Family: qfn; 48 pins; library `Package_DFN_QFN`; Kyocera standard part: PB-EB5704
- Footprint: generator package/no_lead (our entry)
- Model: generator package/no_lead through scripts/ckgen.py (ceramic colours, lid added)
- Parameters (mm): body (7.0, 7.0), body_t 0.8, standoff 0.0, pitch 0.5, pad_w 0.25, pad_l 0.4, ep (5.15, 5.15), lid (6.5, 6.5), lid_t 0.1, seal_w 0.3, overall height lid on 0.9
- Pin 1: generator pin-1 mark on the body top, pin 1 at the top of the left side
- Sources:
  - body size, body thickness, pitch, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=qfn (row PB-EB5704)
  - land width 0.18..0.30, land length 0.30..0.50: kicad-footprint-generator/data/package/no_lead/qfn-*.yaml (cites JEDEC MO-220, https://www.jedec.org/system/files/docs/MO-220K01.pdf)
  - lid thickness 0.10: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no (flat Fe-Ni-Co lids)
- Typical, not sourced: exposed pad (5.15 square, the size of the library's plastic footprint; whether the Kyocera part has a bottom pad was not confirmed); lid size (body - 0.5); glue seal width; floor thickness (0.25) and cavity depth; bond fingers

### QFN-64-1EP_9x9mm_P0.5mm_EP7.15x7.15mm_CeramicCavity

- Family: qfn; 64 pins; library `Package_DFN_QFN`; Kyocera standard part: PB-EB5710
- Footprint: generator package/no_lead (our entry)
- Model: generator package/no_lead through scripts/ckgen.py (ceramic colours, lid added)
- Parameters (mm): body (9.0, 9.0), body_t 0.8, standoff 0.0, pitch 0.5, pad_w 0.25, pad_l 0.4, ep (7.15, 7.15), lid (8.5, 8.5), lid_t 0.1, seal_w 0.3, overall height lid on 0.9
- Pin 1: generator pin-1 mark on the body top, pin 1 at the top of the left side
- Sources:
  - body size, body thickness, pitch, chip size: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=package&productConfiguration1=qfn (row PB-EB5710)
  - land width 0.18..0.30, land length 0.30..0.50: kicad-footprint-generator/data/package/no_lead/qfn-*.yaml (cites JEDEC MO-220, https://www.jedec.org/system/files/docs/MO-220K01.pdf)
  - lid thickness 0.10: https://prdct-search.kyocera.co.jp/semiconductor/en/result?searchType=lid&tab=lid&sealMaterial=no (flat Fe-Ni-Co lids)
- Typical, not sourced: exposed pad (7.15 square, the size of the library's plastic footprint; whether the Kyocera part has a bottom pad was not confirmed); lid size (body - 0.5); glue seal width; floor thickness (0.25) and cavity depth; bond fingers


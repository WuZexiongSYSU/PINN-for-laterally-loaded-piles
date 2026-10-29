# Data provenance and dictionary

## Figure 9 FEM reference document

Source: Wu et al. (2026), *Physics-Informed Neural Network Analysis of Laterally Loaded Piles Based on Elastic Subgrade Reaction Approach*, DOI [10.1002/nag.70382](https://doi.org/10.1002/nag.70382), Figure 9 on PDF page 12.

`figure9_fem_reference.csv` contains **graphically extracted FEM reference values**. The supplied PDF preserves vector graphics, including the centers of the colored "+" markers identified as FEM in the legend. We extracted these centers directly from vector paths. We did not sample the PINN lines, calculate a beam solution, call a finite element solver, infer missing nodes, or interpolate additional FEM measurements.

### Files

| File | Purpose |
|---|---|
| `figure9_cases.csv` | All 20 parameter combinations, using published legend values |
| `figure9_fem_reference.csv` | 1360 reference values: 20 cases × 4 quantities × 17 markers |
| `figure9_calibration.json` | Source hash, RGB colors, panel bounds, ticks, and affine transforms |
| `training_history.csv.gz` | The separate archived PINN training history; not FEM data |

### Case schema

| Column | Meaning | Units |
|---|---|---|
| `case_id` | Group and one-based color index | text |
| `group` | `loads`, `EI`, `Lp`, `m`, or `n` | text |
| `order` | Blue, red, gray, teal mapped to 1, 2, 3, 4 | integer |
| `Vt_kN` | Head shear force | kN |
| `Mt_kNm` | Head moment | kN m |
| `EI_kNm2` | Pile flexural rigidity | kN m² |
| `Lp_m` | Published pile length | m |
| `m_kN_per_m_n_plus_2` | Coefficient in `k = m*x**n` | kN/m^(n+2) |
| `n` | Soil stiffness exponent | dimensionless |

In the `m` sweep, `n = 1`, so `m` has units kN/m³. In the `n` sweep, the numerical value of `m` remains 1000 while its units follow the changed exponent. These values follow the paper's convention.

### FEM reference schema

| Column | Meaning |
|---|---|
| `case_id` | Foreign key into `figure9_cases.csv` |
| `quantity` | `y` (m), `M` (kN m), `Fs` (kN), or `p` (kN/m) |
| `point_index` | Zero-based marker order, 0-16 |
| `x_m` | Depth from the figure's calibrated vertical axis, positive downward |
| `value` | Response from the calibrated horizontal axis, in the quantity's units |
| `pdf_u_pt`, `pdf_v_pt` | Original marker center in PDF points, with origin at the upper left |

Each quantity has its own extracted depth coordinates. No common-depth interpolation is imposed on the stored values. Small differences between corresponding depths in different panels are expected from graphical coordinate rounding.

### Extraction and calibration

1. Read vector paths from PDF page 12 using PyMuPDF. Identify the 20 white plotting rectangles in row-major order.
2. Identify the four reference colors from the original legend, preserving blue/red/gray/teal order. The exact RGB values are recorded in `figure9_calibration.json`.
3. Locate "+" path primitives consisting of horizontal and vertical strokes of approximately 3.04 PDF points. Some strokes have duplicate reversed segments; these are recognized as the same marker shape. Extract the crossing point. Ignore filled outlines of text, continuous curves, pastel overlaid paths, and legend symbols outside the plotting rectangles.
4. Identify the major-axis tick centers and transcribe their printed values. For every panel, fit separate affine maps from PDF coordinates to depth and response using all labeled major ticks:

   ```text
   response = value_scale * pdf_u_pt + value_offset
   depth_m  = depth_scale * pdf_v_pt + depth_offset
   ```

5. Preserve the 17 extracted markers per curve, including their raw coordinates. Write the long-format CSV. The calibration file records all tick coordinates and values, so conversions are independently auditable.

The calibration file preserves the tick positions, transforms, and source PDF hash for independent auditing. Training and plotting require neither the PDF nor a PDF-processing package.

### Precision and coverage

The PDF stores coordinates at finite precision, and the visible tick geometry also has rounding. Exporting nine significant figures preserves the extracted coordinate calculation; it does **not** imply that the FEM solution is accurate to nine figures. Treat the values as graphical reference data. A conservative coordinate perturbation of 0.02 PDF points corresponds to at most roughly 0.0051 m in depth and a panel-dependent response perturbation. This is a practical digitization scale, not a certified statistical uncertainty bound.

The original simulations use 50 elements according to Section 4.2. Most curves show markers at approximately `0, 0.06*Lp, ..., 0.96*Lp`, consistent with every third node of a 51-node discretization. The PDF supplies 17 plotted markers rather than all 51 simulation nodes. No marker is supplied at the exact pile end. Rotation is not plotted in Figure 9 and is not included in the reference CSV.

Small negative extracted head depths (around 0.0001 m) arise from axis rounding. They remain unchanged in the CSV and in plotted markers. For PINN evaluation only, the notebook clips them to zero. Accuracy integration covers the first-to-last displayed markers, without extrapolating the FEM reference to the pile end. The resulting overlap ratios are approximate.

### Published-label discrepancy: `Lp_1`

The blue pile-length curve is labeled **`Lp = 9 m`** in Figure 9. Its marker interval is approximately 0-8.16 m, with a spacing of approximately 0.51 m. Given the marker pattern above, these depths suggest an 8.5 m length. The visible curve also ends above 9 m. This is graphical evidence of a source inconsistency, not proof of the original FEM input.

This repository keeps `Lp_m = 9` in `figure9_cases.csv`, evaluates the PINN at 9 m, and preserves all extracted reference depths and values. We do not silently change the case to 8.5 m or rescale the FEM depths. The resulting comparison is weaker for this case. It must be resolved against the original simulation records before treating this case as a precise validation benchmark.

## Archived PINN history and model

The original research archive supplies `train_save/Fc/model` and `train_save/Fc/datas`. Only that archive is used here. Its weights have two independent branches with input width 3, four hidden layers of width 60, and output width 1.

The original history contains 656,000 rows in the order `gov`, `Mt`, `Vt`, `Mb`, `Vb`. It was exported to compressed CSV with one-based `epoch` values and columns `Loss_Gov`, `Loss_Mt`, `Loss_Vt`, `Loss_Mb`, `Loss_Vb`. All entries are finite and nonnegative. Values retain ten significant figures, sufficient to preserve the original float32 loss records.

The original primitive-container history was read once during preparation; runtime users read a CSV and use `torch.load(..., weights_only=True)` for the model. The archive does not contain optimizer state or a random seed. An independently verified checkpoint epoch counter was unavailable. The final weights are associated with this local archive, not certified as the epoch-200000 model.

Figure 4(b) is reconstructed from the first 200,000 entries, including the original initialization transient. Its relative curves divide each term by its own actual value at epoch 1000. Neither the exact random seed nor an identity between this trace and the printed Figure 4 is claimed.

`models/manifest.json` records original source hashes and hashes of the redistributed numerical assets. This release does not redistribute the paper PDF or original artwork. Please cite the source paper when using the digitized numerical facts.

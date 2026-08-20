# Validation — Sentinel-1 InSAR Amatrice 2016

## 1. Validation Objective

The purpose of this validation stage is to assess whether the final Sentinel-1 Line-of-Sight (LOS) displacement field is spatially plausible in relation to independent earthquake-source information for the 24 August 2016 Amatrice event.

The validation is intentionally framed as a **structural and geospatial consistency assessment**.

It is **not** a direct quantitative accuracy assessment because the external reference used in this project—the DISS Composite Seismogenic Source—is a modeled seismogenic-source geometry rather than an observed displacement raster or a set of geodetic displacement measurements.

The validation therefore asks the following question:

> Does the independently derived InSAR deformation pattern occur in a spatially reasonable relationship to the modeled Accumoli–Amatrice seismogenic-source geometry, the mainshock epicentral area, and the principal reference locations?

---

## 2. Product Being Validated

The validation was performed on the final reference-corrected and coherence-filtered LOS displacement product derived from the Sentinel-1A interferometric pair acquired on:

- **15 August 2016**
- **27 August 2016**

The final analysis raster represents relative LOS displacement after:

- interferometric processing;
- phase unwrapping;
- phase-to-displacement conversion;
- terrain correction;
- coherence-based quality masking;
- stable-reference correction;
- GeoTIFF export;
- final raster QA/QC.

The final coherence threshold was:

```text
γ ≥ 0.40
```

The final displacement product was projected to:

```text
WGS 84 / UTM Zone 33N
EPSG:32633
```

This projected coordinate system was used for final GIS inspection and cartographic validation.

---

## 3. Validation Reference Data

### 3.1 DISS 3.3.1

The principal independent structural reference was the **Database of Individual Seismogenic Sources (DISS), version 3.3.1**.

The selected Composite Seismogenic Source was:

```text
ITCS128 — Accumoli-Amatrice
```

The relevant source feature was isolated using the identifier:

```text
"idsource" = 'ITCS128'
```

The DISS geometry was used to provide an independent representation of the seismogenic-source area associated with the Amatrice–Accumoli region.

### 3.2 Reference Locations

The final validation map also included:

- **Amatrice**
- **Accumoli**
- **24 August 2016 earthquake epicentral location**

These reference locations provide geographic context for visual interpretation of the deformation field.

---

## 4. Important Interpretation of the DISS Geometry

The DISS ITCS128 polygon shown on the final map represents a **source-plane projection / modeled seismogenic-source geometry**.

It must not be interpreted as:

- a mapped surface-rupture trace;
- a direct observation of ground displacement;
- an InSAR-derived fault boundary;
- a pixel-level reference displacement dataset.

This distinction is important because a modeled source geometry and a satellite-derived LOS displacement raster describe different physical quantities.

The DISS layer is therefore used here as an **independent structural reference**, not as a numerical ground-truth raster.

---

## 5. Validation Strategy

The validation was performed in QGIS by overlaying the final LOS displacement raster with the independent reference information.

The comparison focused on four components:

1. **spatial relationship between the principal deformation field and ITCS128;**
2. **relationship between the deformation field and the mainshock epicentral area;**
3. **location of Amatrice relative to the deformation pattern;**
4. **location of Accumoli relative to the deformation pattern.**

The purpose was to determine whether the final interferometric result exhibits coherent spatial organization in the expected earthquake-affected region rather than isolated or geographically implausible patterns.

---

## 6. Validation Map

The final cartographic product is stored in:

```text
figures/final/amatrice_insar_los_displacement_validation.png
```

The map integrates:

- reference-corrected LOS displacement;
- coherence-based quality filtering;
- DISS ITCS128 source-plane projection;
- Amatrice;
- Accumoli;
- the 24 August 2016 epicentral location;
- north arrow;
- scale bar;
- CRS information;
- source information.

The map is intended to communicate both the deformation result and the independent structural comparison in a single figure.

---

## 7. Cartographic Displacement Representation

The displayed LOS displacement range was:

| Display parameter | Value |
|---|---:|
| Minimum | -0.18 m |
| Neutral value | 0.00 m |
| Maximum | +0.06 m |

White represents approximately:

```text
0 m LOS displacement
```

The cartographic display limits were selected for visualization and should not be confused with the complete numerical range of the final raster.

The full-raster statistics were:

| Statistic | Value |
|---|---:|
| Valid pixels | 3,033,433 |
| Minimum | -0.228492 m |
| Maximum | +0.064816 m |
| Mean | -0.041798 m |
| Standard deviation | 0.047592 m |

These raster extrema were not independently interpreted as representative peak physical deformation because local values may be influenced by residual decorrelation, unwrapping uncertainty, atmospheric effects, or other interferometric artifacts.

---

## 8. Raster Quality Context

Validation was carried out only after the principal internal quality-control stages had been completed.

### 8.1 Coherence Filtering

Two coherence thresholds were evaluated during processing:

```text
γ ≥ 0.30
γ ≥ 0.40
```

The final product used:

```text
γ ≥ 0.40
```

This threshold provided stronger suppression of low-confidence measurements while retaining the principal deformation pattern.

### 8.2 Stable-Reference Correction

A stable reference area outside the principal deformation lobes was used to remove the residual LOS displacement offset.

Before correction, the reference-area mean displacement was approximately:

```text
+0.0112 m
```

After correction, the reference-area statistics were:

| Statistic | Value |
|---|---:|
| Mean | -1.9926 × 10^-5 m |
| Standard deviation | ~0.0050 m |
| Median | -5.8089 × 10^-4 m |

The corrected mean is effectively zero, supporting the internal consistency of the relative referencing step.

This internal QA/QC does not replace external validation, but it reduces the likelihood that the structural comparison is dominated by an arbitrary raster-wide offset.

---

## 9. Spatial Validation Criteria

The final result was assessed using the following qualitative criteria.

### Criterion 1 — Spatial Coherence of the Deformation Field

The deformation signal should form a spatially organized pattern rather than a collection of random isolated values.

**Assessment:**  
The final raster shows a coherent regional LOS displacement pattern after coherence masking and stable-reference correction.

### Criterion 2 — Relationship to the DISS ITCS128 Source Geometry

The deformation field should exhibit a geographically meaningful relationship to the modeled Accumoli–Amatrice seismogenic-source region.

**Assessment:**  
The principal deformation corridor is spatially consistent with the modeled ITCS128 source geometry.

### Criterion 3 — Relationship to the Epicentral Area

The earthquake epicentral location should occur within the broader area relevant to the detected coseismic deformation.

**Assessment:**  
The epicentral area is spatially associated with the principal deformation zone shown in the final LOS displacement map.

### Criterion 4 — Relationship to Amatrice and Accumoli

The mapped deformation should be geographically meaningful relative to the two principal reference locations included in the analysis.

**Assessment:**  
Amatrice and Accumoli lie within the regional context of the mapped coseismic deformation field and the ITCS128 source geometry.

---

## 10. Validation Outcome

The final Sentinel-1 LOS displacement field demonstrates **qualitative spatial consistency** with:

- the principal coseismic deformation corridor;
- the DISS 3.3.1 ITCS128 Accumoli–Amatrice Composite Seismogenic Source;
- the 24 August 2016 mainshock epicentral area;
- the Amatrice–Accumoli geographic context.

This supports the **geospatial plausibility** of the final InSAR result.

The result should be described as:

> spatially consistent with independent seismogenic-source information

rather than:

> quantitatively validated against DISS displacement measurements

because DISS does not provide a directly comparable observed LOS displacement field for this analysis.

---

## 11. What This Validation Demonstrates

The validation supports the following conclusions:

- the final displacement field is not geographically random;
- the mapped deformation occupies a spatially meaningful earthquake-affected corridor;
- the result is compatible with the independent ITCS128 source geometry at the map scale used in this project;
- the epicentral and reference locations provide coherent geographic context for the deformation field;
- the independent comparison strengthens confidence in the structural plausibility of the processing result.

---

## 12. What This Validation Does Not Demonstrate

This validation does **not** establish:

- centimeter-level absolute accuracy;
- pixel-by-pixel agreement with ground observations;
- vertical displacement;
- three-dimensional displacement;
- exact fault-slip magnitude;
- exact fault geometry;
- a mapped surface-rupture trace;
- atmospheric-error removal;
- complete elimination of phase-unwrapping errors;
- GNSS-level geodetic accuracy.

These claims would require additional independent observations or a different validation design.

---

## 13. Validation Limitations

### 13.1 Structural Reference Instead of Displacement Ground Truth

DISS ITCS128 is a modeled seismogenic-source geometry.

It does not provide the same observable as the Sentinel-1 LOS raster.

Consequently, the comparison is spatial and structural rather than measurement-to-measurement.

### 13.2 Single Interferometric Pair

The analysis uses a single 12-day pre-event/post-event interferometric pair.

A single pair cannot isolate all atmospheric contributions through temporal averaging.

### 13.3 LOS Geometry

Sentinel-1 measures displacement projected along the radar Line of Sight.

The final map therefore cannot independently resolve the full east-west, north-south, and vertical displacement components.

### 13.4 Coherence Masking

The coherence threshold removes many low-confidence measurements, but:

- some valid lower-coherence pixels may be excluded;
- retained high-coherence pixels are not automatically error-free.

### 13.5 Phase Unwrapping

Local phase-unwrapping errors can propagate into the displacement field.

The validation therefore emphasizes broad spatial pattern consistency rather than isolated extreme pixels.

### 13.6 Stable-Reference Assumption

The final displacement values are relative to the selected stable reference polygon.

If that area contains unrecognized displacement, a residual reference bias could remain.

---

## 14. Recommended Language for Scientific Communication

To avoid overstating the validation, the project should use wording such as:

> The final Sentinel-1 LOS displacement field shows qualitative spatial consistency with the DISS 3.3.1 ITCS128 Accumoli–Amatrice Composite Seismogenic Source and the 24 August 2016 epicentral area.

or:

> DISS ITCS128 was used as an independent structural reference to assess the geospatial plausibility of the InSAR-derived coseismic deformation pattern.

Avoid wording such as:

> DISS proves the InSAR displacement is exact.

or:

> The DISS polygon represents the surface rupture measured by Sentinel-1.

Those statements would exceed what the validation dataset supports.

---

## 15. Opportunities for Stronger Future Validation

A more quantitative validation framework could include independent geodetic observations such as:

- GNSS coseismic displacement measurements;
- published displacement profiles;
- independent ascending Sentinel-1 interferograms;
- independent descending Sentinel-1 interferograms;
- multi-temporal InSAR products;
- published coseismic deformation models.

Potential extensions could compare:

- LOS displacement values at GNSS sites;
- deformation-profile shape and amplitude;
- ascending vs. descending displacement geometry;
- residuals between observed and modeled LOS displacement;
- spatial uncertainty as a function of coherence.

These additions would allow the project to progress from structural plausibility validation toward quantitative geodetic validation.

They were not part of the present repository workflow and are therefore not claimed in the current results.

---

## 16. Reproducibility of the Validation

The validation can be reproduced using:

1. the final reference-corrected LOS displacement GeoTIFF;
2. the DISS 3.3.1 dataset;
3. the ITCS128 source selection;
4. the Amatrice reference location;
5. the Accumoli reference location;
6. the 24 August 2016 epicentral location;
7. QGIS or equivalent GIS software.

The required source filter is:

```text
"idsource" = 'ITCS128'
```

The final validation map should preserve the same conceptual layers:

```text
LOS displacement
        +
DISS ITCS128 source geometry
        +
Epicentral location
        +
Amatrice
        +
Accumoli
```

The interpretation should remain qualitative unless an independent displacement dataset is added.

---

## 17. Validation Summary

The validation strategy combines internal InSAR quality control with an external structural comparison.

### Internal controls

- coherence assessment;
- γ ≥ 0.40 quality masking;
- stable-reference correction;
- raster QA/QC;
- spatial inspection of the final displacement field.

### External comparison

- DISS 3.3.1;
- ITCS128 Accumoli–Amatrice source geometry;
- 24 August 2016 epicentral area;
- Amatrice;
- Accumoli.

### Final assessment

The final LOS displacement field shows a coherent spatial pattern that is **qualitatively consistent with the principal earthquake deformation corridor and the independent Accumoli–Amatrice seismogenic-source geometry**.

This constitutes a defensible geospatial validation for the scope of the project while clearly distinguishing structural consistency from quantitative displacement accuracy.

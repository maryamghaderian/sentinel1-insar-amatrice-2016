# Methodology — Sentinel-1 InSAR Earthquake Deformation Mapping

## 1. Purpose and Analytical Design

This project uses Sentinel-1 Interferometric Synthetic Aperture Radar (InSAR) to map coseismic Line-of-Sight (LOS) surface displacement associated with the 24 August 2016 Amatrice earthquake in Central Italy.

The methodological design was structured around four objectives:

1. derive a geocoded coseismic LOS displacement field from a pre-event/post-event Sentinel-1 interferometric pair;
2. preserve interferometric data quality through explicit coregistration and coherence-based quality control;
3. remove the residual displacement offset using a stable reference area;
4. assess the spatial plausibility of the final deformation field using independent seismogenic-source information.

The workflow was implemented primarily in ESA SNAP, with SNAPHU used for phase unwrapping and QGIS used for final raster verification, cartographic visualization, and spatial validation.

This document explains the methodological rationale and quality-control logic of the analysis. The detailed operator-by-operator processing sequence is documented separately in [`workflow.md`](workflow.md).

---

## 2. Study Area and Event Context

The study area covers the Amatrice–Accumoli region of Central Italy.

The interferometric pair spans the 24 August 2016 Amatrice earthquake, allowing the phase difference between the two Sentinel-1 acquisitions to capture the dominant coseismic deformation signal occurring between the acquisition dates.

The analysis was spatially restricted to the relevant Sentinel-1 IW3 subswath and the Amatrice–Accumoli study extent to:

- reduce unnecessary processing volume,
- limit computational cost,
- focus phase unwrapping on the deformation area,
- and retain the principal earthquake-related displacement pattern.

---

## 3. Sentinel-1 Input Data

Two Sentinel-1A Interferometric Wide Swath Single Look Complex (IW SLC) acquisitions were used.

| Parameter | Pre-event acquisition | Post-event acquisition |
|---|---|---|
| Platform | Sentinel-1A | Sentinel-1A |
| Acquisition date | 15 August 2016 | 27 August 2016 |
| Product type | IW SLC | IW SLC |
| Polarization | VV | VV |
| Relative orbit / track | 117 | 117 |
| Orbit number | 12614 | 12789 |

The temporal baseline is 12 days.

The SLC product type was required because interferometric processing depends on the complex SAR signal, including both amplitude and phase.

Using acquisitions from the same relative orbit provides a consistent repeat-pass viewing geometry suitable for interferometric phase comparison.

---

## 4. Auxiliary Data

The following auxiliary datasets were incorporated into processing or validation:

- **Copernicus DEM GLO-30** for radar-geometry operations, topographic phase removal, and terrain correction;
- **Sentinel precise orbit information** for improved satellite-position accuracy;
- **DISS 3.3.1 Composite Seismogenic Source ITCS128 — Accumoli-Amatrice** for independent structural comparison;
- reference locations for **Amatrice** and **Accumoli**;
- the **24 August 2016 earthquake epicenter**.

The DISS source geometry was used as an independent structural reference. It was not interpreted as a directly mapped surface-rupture trace.

---

## 5. Software and Processing Environment

The analysis used the following software:

- **ESA SNAP** — Sentinel-1 preprocessing, coregistration, interferogram generation, phase processing, displacement conversion, terrain correction, raster collocation, and band mathematics;
- **SNAPHU 2.0.4** — external phase unwrapping;
- **QGIS** — raster inspection, GeoTIFF verification, band extraction, cartographic production, and spatial validation;
- **GDAL tools through QGIS** — final raster band extraction and GeoTIFF preparation.

The workflow combines graphical and command-line operations, with major processing parameters retained in the repository for reproducibility.

---

## 6. Sentinel-1 TOPS Preprocessing

### 6.1 TOPS Split

The Sentinel-1 IW SLC products were restricted to:

- **Subswath:** IW3
- **Polarization:** VV
- **Spatial focus:** Amatrice–Accumoli study area

TOPS Split was performed before the main interferometric processing chain.

Restricting processing to the relevant subswath and polarization reduced computational demand while retaining the earthquake-affected region.

### 6.2 Precise Orbit Correction

Precise orbit information was applied to both acquisitions before coregistration.

Processing parameters included:

- **Orbit type:** Sentinel Precise
- **Orbit source:** Auto Download
- **Polynomial degree:** 3

Precise orbit information improves the geometric description of the satellite trajectory and reduces orbit-related uncertainty in subsequent interferometric processing.

---

## 7. TOPS Coregistration Strategy

Accurate coregistration is essential for Sentinel-1 TOPS interferometry because small azimuth misregistrations can introduce strong phase ramps and discontinuities.

The project therefore used a two-stage TOPS coregistration strategy.

### 7.1 Back-Geocoding

The post-event acquisition was aligned to the pre-event acquisition using SNAP Back-Geocoding.

The operation used:

- **DEM:** Copernicus 30 m Global DEM
- **DEM resampling:** Bilinear interpolation
- **Image resampling:** BISINC 21-point interpolation
- **Mask areas without elevation:** Enabled

Back-Geocoding establishes the common radar geometry required for interferometric phase comparison.

### 7.2 Enhanced Spectral Diversity

Enhanced Spectral Diversity (ESD) was applied after Back-Geocoding.

ESD uses burst-overlap information to refine azimuth registration in Sentinel-1 TOPS data.

The estimated shift information was inspected as a short QA/QC step before interferogram formation.

This check was important because a technically successful operator execution does not, by itself, guarantee sufficiently accurate TOPS coregistration.

---

## 8. Interferogram and Coherence Estimation

The coregistered pair was used to generate:

- wrapped interferometric phase;
- interferometric coherence.

### 8.1 Interferogram Parameters

The main interferogram settings were:

- **Subtract flat-earth phase:** Enabled
- **Flat-earth polynomial degree:** 5
- **Number of estimation points:** 501
- **Orbit interpolation degree:** 3
- **Subtract topographic phase during interferogram formation:** Disabled
- **Coherence estimation:** Enabled
- **Square pixel:** Enabled

### 8.2 Coherence Window

The coherence estimation window was:

- **Range window size:** 10
- **Azimuth window size:** 2

Interferometric coherence was retained as the primary pixel-level quality indicator for later masking.

At this stage, interferometric phase may include contributions from:

- surface displacement;
- topography;
- atmospheric delay;
- residual orbital effects;
- decorrelation;
- measurement noise.

For this reason, the interferogram was not interpreted directly as displacement.

---

## 9. TOPS Deburst

The individual Sentinel-1 TOPS bursts were merged using TOPS Deburst.

This operation created spatially continuous interferometric phase and coherence products across burst boundaries.

Debursting is required before later spatial operations that assume a continuous raster representation.

---

## 10. Topographic Phase Removal

The topographic phase contribution was removed using Copernicus DEM GLO-30.

The workflow used:

- **DEM:** Copernicus 30 m Global DEM
- **Orbit interpolation degree:** 3

After removal of the modeled topographic contribution, the remaining phase is more directly related to surface displacement but can still include atmospheric, orbit-related, unwrapping, and noise components.

Accordingly, additional processing and quality control remained necessary.

---

## 11. Goldstein Phase Filtering

Goldstein phase filtering was applied before phase unwrapping.

The selected parameters were:

- **Adaptive filter exponent:** 1.0
- **FFT size:** 64
- **Window size:** 3
- **Coherence masking during filtering:** Disabled

The purpose of filtering was to suppress high-frequency phase noise and improve spatial phase continuity in coherent areas without intentionally redefining the broad deformation pattern.

---

## 12. Spatial Subsetting Before Unwrapping

A spatial subset covering the principal earthquake deformation area was created before export to SNAPHU.

The subset retained the required interferometric information, including:

- I component;
- Q component;
- intensity;
- wrapped phase;
- coherence.

Spatial subsetting reduced data volume and the computational cost of external phase unwrapping.

It also concentrated the unwrapping problem on the area of scientific interest.

---

## 13. Phase Unwrapping

### 13.1 SNAPHU Export

The filtered phase and coherence products were exported from SNAP for external unwrapping.

The coherence raster was explicitly used as the SNAPHU correlation input.

### 13.2 SNAPHU Processing

Phase unwrapping was performed using **SNAPHU 2.0.4**.

The wrapped interferometric phase is measured modulo $2\pi$. Phase unwrapping estimates a spatially continuous phase field by resolving the integer-cycle ambiguity between neighboring pixels.

The resulting unwrapped phase was inspected for spatial continuity before being imported back into SNAP.

### 13.3 SNAPHU Import

The unwrapped phase product was imported into SNAP and used as the input for phase-to-displacement conversion.

Phase unwrapping is one of the most sensitive stages in an InSAR workflow. Errors introduced during unwrapping can propagate directly into the displacement product, so continuity and plausibility checks are essential.

---

## 14. Phase-to-LOS Displacement Conversion

The unwrapped phase was converted to Line-of-Sight displacement using SNAP's Phase to Displacement operator.

Conceptually, interferometric phase and radar LOS displacement are related through the radar wavelength. The sign of the reported displacement depends on the processing convention and viewing geometry.

The final product therefore represents **motion projected along the satellite Line of Sight**, not pure vertical displacement and not a complete three-dimensional ground-motion vector.

This distinction was retained throughout interpretation.

---

## 15. Terrain Correction and Geocoding

### 15.1 Displacement Terrain Correction

The LOS displacement raster was geocoded using Range-Doppler Terrain Correction.

Parameters included:

- **DEM:** Copernicus 30 m Global DEM
- **DEM resampling:** Bilinear interpolation
- **Image resampling:** Bilinear interpolation
- **Pixel spacing:** 13.88912 m
- **Output CRS:** WGS 84 / UTM Zone 33N
- **EPSG:** 32633
- **Source band:** Displacement only

### 15.2 Coherence Terrain Correction

The coherence raster was terrain-corrected separately using the same geospatial framework:

- **DEM:** Copernicus 30 m Global DEM
- **Pixel spacing:** 13.88912 m
- **CRS:** WGS 84 / UTM Zone 33N
- **EPSG:** 32633

Using a common projected CRS and pixel spacing allowed the two rasters to be collocated for pixel-level quality filtering.

---

## 16. Raster Collocation

The terrain-corrected displacement and coherence rasters were collocated.

The configuration used:

- **Reference product:** Terrain-corrected displacement
- **Secondary product:** Terrain-corrected coherence
- **Resampling:** Nearest neighbour

The collocated product contained the displacement and coherence information needed to apply a coherence condition directly to each displacement pixel.

---

## 17. Coherence-Based Quality Control

Interferometric coherence was used to suppress low-confidence displacement measurements.

Two thresholds were evaluated:

```text
γ ≥ 0.30
γ ≥ 0.40
```

The 0.40 condition was implemented using the equivalent logic:

```text
coh_IW3_VV_15Aug2016_27Aug2016_S >= 0.40 ? displacement_VV_M : NaN
```

The final product used:

```text
γ ≥ 0.40
```

The 0.40 threshold provided stronger suppression of low-confidence and salt-and-pepper measurements while preserving the principal coseismic deformation pattern.

This value is a **project-specific quality-control decision**, not a universal coherence threshold for every InSAR application.

The resulting masked displacement band was:

```text
displacement_coh04
```

---

## 18. Stable Reference Area

InSAR displacement derived from a single interferogram is relative. A reference area was therefore selected to define a practical zero-displacement level.

The stable reference polygon was placed:

- outside the principal coseismic deformation lobes;
- west of the main deformation zone;
- away from raster boundaries.

### 18.1 Pre-Correction Displacement Statistics

Within the selected reference area:

| Statistic | Value |
|---|---:|
| Valid pixels | 129,293 |
| Minimum | -0.0277 m |
| Maximum | +0.0554 m |
| Mean | +0.0112 m |
| Standard deviation | 0.0050 m |
| Median | +0.0106 m |

### 18.2 Coherence Statistics in the Same Area

| Statistic | Value |
|---|---:|
| Pixels | 222,787 |
| Minimum | 0.0168 |
| Maximum | 0.9907 |
| Mean | 0.4564 |
| Standard deviation | 0.1940 |
| Median | 0.4454 |

The relatively small displacement variability, together with the position of the polygon outside the principal deformation lobes, supported its use as a practical reference region.

---

## 19. Stable-Reference Correction

The mean displacement measured inside the reference polygon was removed from the coherence-masked displacement raster.

The correction was:

```text
corrected_displacement = displacement_coh04 - 0.0112
```

The corrected output band was:

```text
displacement_coh04_ref
```

### 19.1 Post-Correction Reference Statistics

| Statistic | Value |
|---|---:|
| Valid pixels | 129,293 |
| Minimum | -0.0389 m |
| Maximum | +0.0442 m |
| Mean | -1.9926 × 10^-5 m |
| Standard deviation | 0.0050 m |
| Median | -5.8089 × 10^-4 m |

The corrected mean is effectively zero, confirming that the selected residual offset was removed from the reference area.

This operation establishes a **relative displacement reference**. It does not transform the result into an absolute geodetic displacement field.

---

## 20. GeoTIFF Production

The reference-corrected displacement product was exported from SNAP as GeoTIFF.

The initial export contained:

- an auxiliary constant band;
- the LOS displacement band.

The displacement band was extracted in QGIS using GDAL Translate with:

```text
-b 2
```

The final output configuration was:

- **Data type:** Float32
- **NoData:** -9999
- **CRS:** EPSG:32633

Final raster:

```text
Amatrice_LOS_Displacement_20160815_20160827_coh04_ref_UTM33N_singleband.tif
```

---

## 21. Final Raster QA/QC

The final single-band GeoTIFF was verified in QGIS.

### 21.1 Raster Characteristics

| Property | Value |
|---|---|
| Band count | 1 |
| Data type | Float32 |
| NoData | -9999 |
| CRS | EPSG:32633 |
| Pixel size | 13.88912 m |
| Dimensions | 3009 × 2852 pixels |

### 21.2 Full-Raster Statistics

| Statistic | Value |
|---|---:|
| Valid pixels | 3,033,433 |
| Minimum | -0.228492 m |
| Maximum | +0.064816 m |
| Mean | -0.041798 m |
| Standard deviation | 0.047592 m |

The raster extrema were treated as statistical values rather than automatically interpreted as representative peak physical deformation.

Local extrema can be influenced by:

- residual decorrelation;
- phase-unwrapping errors;
- atmospheric contributions;
- residual orbit effects;
- edge effects;
- local low-quality measurements.

Physical interpretation therefore relies on spatial continuity, coherence, and geological context rather than raster extrema alone.

---

## 22. Cartographic Visualization

The final LOS displacement raster was visualized in QGIS using a diverging color ramp centered near zero displacement.

The cartographic display range was:

- **Minimum:** -0.18 m
- **Neutral:** 0.00 m
- **Maximum:** +0.06 m

White was used to represent approximately zero LOS displacement.

The final map included:

- reference-corrected LOS displacement;
- DISS ITCS128 source-plane projection;
- Amatrice;
- Accumoli;
- the 24 August 2016 epicentral location;
- scale bar;
- north arrow;
- CRS information;
- data-source information.

The visualization range is a cartographic decision and is distinct from the complete numerical range of the underlying raster.

---

## 23. Independent Spatial Validation

The final displacement field was compared with the DISS 3.3.1 Composite Seismogenic Source:

```text
ITCS128 — Accumoli-Amatrice
```

Spatial comparison considered:

- the modeled ITCS128 source geometry;
- the principal LOS deformation corridor;
- the 24 August 2016 epicentral area;
- Amatrice;
- Accumoli.

The final deformation pattern showed spatial consistency with the principal coseismic corridor and the modeled Accumoli-Amatrice seismogenic-source geometry.

This comparison provides an **independent structural plausibility check**.

It should not be interpreted as a direct quantitative accuracy assessment because DISS source geometry is not an observed displacement raster.

A dedicated description of the validation logic is provided in [`validation.md`](validation.md).

---

## 24. Interpretation Framework

The final product is a relative Sentinel-1 LOS displacement raster for the 15–27 August 2016 interferometric interval.

Interpretation follows the constraints below:

1. **LOS measurement**  
   The result represents displacement projected onto the radar Line of Sight.

2. **No direct vertical interpretation**  
   LOS values should not automatically be described as pure uplift or subsidence.

3. **Relative reference**  
   The displacement field is referenced to the selected stable area.

4. **Single interferometric pair**  
   Atmospheric contributions cannot be averaged through a multi-temporal time series.

5. **Coherence filtering**  
   The γ ≥ 0.40 mask suppresses many low-confidence pixels but does not guarantee that every retained pixel is error-free.

6. **Phase-unwrapping sensitivity**  
   Unwrapping errors can create locally incorrect displacement estimates.

7. **DEM and orbit residuals**  
   Residual topographic and orbital effects may remain after correction.

8. **Display range vs. numerical range**  
   The map display limits are not identical to the complete raster statistical range.

9. **Structural validation**  
   DISS comparison provides geological context and spatial plausibility, not direct measurement-to-measurement validation.

---

## 25. Methodological Limitations

The project intentionally focuses on a single pre-event/post-event Sentinel-1 interferometric pair.

Therefore:

- no ascending/descending decomposition was performed;
- no 2-D or 3-D displacement inversion was attempted;
- no multi-temporal atmospheric correction was available;
- no GNSS-based quantitative accuracy assessment was included;
- the coherence threshold can remove valid lower-coherence measurements;
- the stable-reference correction depends on the assumption that the selected polygon is sufficiently stable;
- the final raster represents relative LOS displacement rather than absolute three-dimensional ground motion.

These limitations define the scope of the analysis and should accompany scientific interpretation of the result.

---

## 26. Reproducibility

The repository records the major methodological decisions required to understand and reproduce the analysis, including:

- Sentinel-1 acquisition dates;
- product type;
- polarization;
- relative orbit;
- processing subswath;
- study extent;
- precise-orbit strategy;
- coregistration approach;
- DEM selection;
- interferogram parameters;
- coherence window;
- Goldstein filter parameters;
- SNAPHU phase-unwrapping stage;
- terrain-correction parameters;
- output CRS and pixel spacing;
- coherence thresholds;
- stable-reference statistics;
- reference correction;
- GeoTIFF output configuration;
- final raster QA/QC;
- structural validation approach.

Large Sentinel-1 SLC products and intermediate SAR files are intentionally excluded from version control because of their size.

Detailed operational instructions are available in:

- [`workflow.md`](workflow.md) — complete processing sequence;
- [`validation.md`](validation.md) — validation strategy and interpretation.

---

## 27. Methodological Summary

The final analytical chain can be summarized as:

```text
Sentinel-1A IW SLC pair
        ↓
TOPS preprocessing
        ↓
Precise orbit correction
        ↓
Back-Geocoding + ESD
        ↓
Interferogram + coherence
        ↓
Deburst
        ↓
Topographic phase removal
        ↓
Goldstein phase filtering
        ↓
SNAPHU phase unwrapping
        ↓
Phase-to-LOS displacement
        ↓
Terrain correction
        ↓
Displacement/coherence collocation
        ↓
Coherence masking (γ ≥ 0.40)
        ↓
Stable-reference correction
        ↓
Single-band GeoTIFF
        ↓
Raster QA/QC
        ↓
QGIS visualization
        ↓
DISS-based spatial validation
```

The methodology emphasizes not only generation of an InSAR displacement raster, but also explicit quality control, transparent reference correction, conservative interpretation, and independent geospatial validation.

# Sentinel-1 InSAR Processing Workflow

## Overview

This document describes the complete Sentinel-1 InSAR processing workflow used to derive coseismic Line-of-Sight (LOS) displacement associated with the 24 August 2016 Amatrice earthquake in Central Italy.

The processing chain was implemented primarily in ESA SNAP, with SNAPHU used for phase unwrapping and QGIS used for post-processing, raster verification, cartographic visualization, and independent spatial validation.

The interferometric pair consisted of two Sentinel-1A IW SLC acquisitions acquired on 15 August 2016 and 27 August 2016.

---

## 1. Input Sentinel-1 Data

Two Sentinel-1A Interferometric Wide Swath Single Look Complex products were used.

### Acquisition 1

- Platform: Sentinel-1A
- Acquisition date: 15 August 2016
- Product type: IW SLC
- Relative orbit / track: 117
- Orbit number: 12614
- Polarization: VV

### Acquisition 2

- Platform: Sentinel-1A
- Acquisition date: 27 August 2016
- Product type: IW SLC
- Relative orbit / track: 117
- Orbit number: 12789
- Polarization: VV

The 12-day interferometric interval captures the coseismic deformation associated with the 24 August 2016 earthquake.

---

## 2. Sentinel-1 TOPS Split

The Sentinel-1 IW SLC products were subset to the relevant subswath, polarization, and study extent.

### Processing Parameters

- Subswath: IW3
- Polarization: VV
- Spatial extent: Amatrice–Accumoli study area

Restricting the processing to the relevant subswath and polarization reduced computational requirements while retaining the earthquake-affected region.

---

## 3. Apply Precise Orbit File

Precise orbit information was applied to both Sentinel-1 acquisitions before coregistration.

### Parameters

- Orbit type: Sentinel Precise
- Orbit source: Auto Download
- Polynomial degree: 3

Precise orbit information improves satellite position accuracy and provides a more reliable geometric basis for interferometric processing.

---

## 4. Back-Geocoding

The two TOPS SLC acquisitions were coregistered using the SNAP Back-Geocoding operator.

### Digital Elevation Model

- DEM: Copernicus 30 m Global DEM
- DEM source: Auto Download

### Resampling Parameters

- DEM resampling: Bilinear interpolation
- Image resampling: BISINC 21-point interpolation
- Mask areas without elevation: Enabled

The 27 August 2016 acquisition was geometrically aligned with the 15 August 2016 acquisition to establish a common radar geometry for interferometric processing.

---

## 5. Enhanced Spectral Diversity

Enhanced Spectral Diversity (ESD) was applied following back-geocoding.

ESD provides fine azimuth coregistration correction across Sentinel-1 TOPS burst overlaps and reduces phase discontinuities caused by residual azimuth misregistration.

The estimated azimuth shifts were inspected in the SNAP metadata as an additional coregistration quality-control step.

---

## 6. Interferogram Formation

The coregistered Sentinel-1 pair was used to generate the interferogram and coherence layer.

### Interferogram Parameters

- Subtract flat-earth phase: Enabled
- Flat-earth polynomial degree: 5
- Number of estimation points: 501
- Orbit interpolation degree: 3
- Subtract topographic phase during interferogram formation: Disabled
- Coherence estimation: Enabled
- Square pixel: Enabled

### Coherence Window

- Range window size: 10
- Azimuth window size: 2

The interferometric phase at this stage contains contributions from surface displacement, topography, atmospheric effects, residual orbital errors, and measurement noise.

---

## 7. TOPS Deburst

The individual Sentinel-1 TOPS bursts were merged using the TOPS Deburst operator.

This generated spatially continuous interferometric phase and coherence products across burst boundaries.

---

## 8. Topographic Phase Removal

The topographic phase contribution was removed using the Copernicus 30 m Global DEM.

### Parameters

- DEM: Copernicus 30 m Global DEM
- Orbit interpolation degree: 3

After this step, the interferometric phase primarily contains contributions from surface displacement together with residual atmospheric, orbital, and noise components.

---

## 9. Goldstein Phase Filtering

Goldstein phase filtering was applied to reduce interferometric phase noise before phase unwrapping.

### Parameters

- Adaptive filter exponent: 1.0
- FFT size: 64
- Window size: 3
- Coherence masking: Disabled

The filter improved phase continuity in coherent areas while preserving the broad spatial characteristics of the deformation signal.

---

## 10. Spatial Subsetting

A spatial subset covering the principal earthquake deformation area was created before SNAPHU export.

The subset retained the following bands:

- I component
- Q component
- Intensity
- Wrapped phase
- Coherence

Spatial subsetting reduced data volume and the computational cost of external phase unwrapping.

---

## 11. SNAPHU Export

The filtered interferometric phase and coherence products were exported from SNAP for external phase unwrapping using SNAPHU.

### Exported Phase

```text
Phase_ifg_VV_15Aug2016_27Aug2016.snaphu.img
```

### Exported Coherence

```text
coh_IW3_VV_15Aug2016_27Aug2016.snaphu.img
```

The SNAPHU configuration file was checked before execution.

The coherence input was explicitly defined as:

```text
CORRFILE coh_IW3_VV_15Aug2016_27Aug2016.snaphu.img
CORRFILEFORMAT FLOAT_DATA
```

---

## 12. SNAPHU Phase Unwrapping

Phase unwrapping was performed externally using SNAPHU 2.0.4.

The unwrapping command was executed using:

```text
snaphu.exe -f snaphu.conf Phase_ifg_VV_15Aug2016_27Aug2016.snaphu.img 10754
```

The operation completed successfully and produced:

```text
UnwPhase_ifg_VV_15Aug2016_27Aug2016.snaphu.img
```

The unwrapped phase exhibited a spatially coherent deformation field across the principal earthquake-affected area.

---

## 13. SNAPHU Import

The externally generated unwrapped phase was imported back into ESA SNAP.

The resulting product contained:

- I component
- Q component
- Intensity
- Wrapped phase
- Coherence
- Unwrapped phase

The principal unwrapped-phase band was:

```text
Unw_Phase_ifg_15Aug2016_27Aug2016
```

---

## 14. Phase-to-Displacement Conversion

The unwrapped phase was converted into Line-of-Sight displacement using the SNAP Phase to Displacement operator.

The resulting displacement band was used as the primary geophysical product for subsequent terrain correction, masking, referencing, and validation.

---

## 15. Range-Doppler Terrain Correction

The displacement raster was geocoded using Range-Doppler Terrain Correction.

### Parameters

- DEM: Copernicus 30 m Global DEM
- DEM resampling: Bilinear interpolation
- Image resampling: Bilinear interpolation
- Pixel spacing: 13.88912 m
- Output CRS: WGS 84 / UTM Zone 33N
- EPSG: 32633
- Source band: Displacement only

The resulting terrain-corrected displacement band was:

```text
displacement_VV
```

---

## 16. Coherence Terrain Correction

The coherence band was terrain-corrected separately using the same geocoding framework.

### Parameters

- DEM: Copernicus 30 m Global DEM
- Pixel spacing: 13.88912 m
- CRS: WGS 84 / UTM Zone 33N
- EPSG: 32633

This produced a geocoded coherence raster spatially compatible with the displacement product.

---

## 17. Raster Collocation

The terrain-corrected displacement raster and terrain-corrected coherence raster were collocated.

### Collocation Configuration

- Reference product: Terrain-corrected displacement
- Secondary product: Terrain-corrected coherence
- Resampling: Nearest neighbour

The resulting collocated product contained:

```text
displacement_VV_M
coh_IW3_VV_15Aug2016_27Aug2016_S
collocationFlags
```

---

## 18. Coherence-Based Quality Masking

Coherence thresholds were evaluated to suppress unreliable displacement measurements.

Two coherence masks were tested.

### Threshold 0.30

```text
coh_IW3_VV_15Aug2016_27Aug2016_S >= 0.30 ? displacement_VV_M : NaN
```

Output band:

```text
displacement_coh03
```

### Threshold 0.40

```text
coh_IW3_VV_15Aug2016_27Aug2016_S >= 0.40 ? displacement_VV_M : NaN
```

Output band:

```text
displacement_coh04
```

The 0.40 coherence threshold was selected for the final product because it provided improved suppression of low-confidence and salt-and-pepper measurements while preserving the principal coseismic deformation pattern.

---

## 19. Stable Reference Area Selection

A stable reference polygon was manually selected outside the principal coseismic deformation lobes.

The reference area was positioned west of the main deformation zone and away from raster boundaries.

### Displacement Statistics Within the Reference Area

- Number of valid pixels: 129,293
- Minimum: -0.0277 m
- Maximum: +0.0554 m
- Mean: +0.0112 m
- Standard deviation: 0.0050 m
- Median: +0.0106 m

### Coherence Statistics Within the Same Area

- Number of pixels: 222,787
- Minimum: 0.0168
- Maximum: 0.9907
- Mean: 0.4564
- Standard deviation: 0.1940
- Median: 0.4454

The relatively low displacement variability and the coherence characteristics supported use of this area as a practical stable reference for residual offset correction.

---

## 20. Stable-Reference Correction

The mean displacement of the stable reference area was removed using SNAP Band Maths.

### Expression

```text
displacement_coh04 - 0.0112
```

### Output Band

```text
displacement_coh04_ref
```

### Unit

```text
m
```

### Post-Correction Statistics Within the Stable Reference Area

- Number of valid pixels: 129,293
- Minimum: -0.0389 m
- Maximum: +0.0442 m
- Mean: -1.9926 × 10^-5 m
- Standard deviation: 0.0050 m
- Median: -5.8089 × 10^-4 m

The corrected mean displacement was effectively zero, confirming successful removal of the residual reference offset.

---

## 21. GeoTIFF Export

The reference-corrected displacement product was exported from SNAP as GeoTIFF.

The initial export contained two bands:

- Band 1: constant auxiliary band
- Band 2: LOS displacement

Band 2 was therefore extracted in QGIS using GDAL Translate.

### GDAL Band Extraction

Additional command-line parameter:

```text
-b 2
```

### Output Configuration

- Output data type: Float32
- NoData value: -9999
- Output CRS: EPSG:32633

Final output:

```text
Amatrice_LOS_Displacement_20160815_20160827_coh04_ref_UTM33N_singleband.tif
```

---

## 22. Final Raster Verification

The final single-band GeoTIFF was verified in QGIS.

### Raster Characteristics

- Band count: 1
- Data type: Float32
- NoData value: -9999
- CRS: EPSG:32633
- Pixel size: 13.88912 m
- Raster dimensions: 3009 × 2852 pixels

### Full-Raster Statistics

- Valid pixel count: 3,033,433
- Minimum: -0.228492 m
- Maximum: +0.064816 m
- Mean: -0.041798 m
- Standard deviation: 0.047592 m

The raster extrema were retained as statistical values and were not independently interpreted as representative peak physical deformation without consideration of local coherence and spatial context.

---

## 23. Cartographic Visualization

The final LOS displacement raster was visualized in QGIS using a diverging color scheme.

### Display Range

- Minimum: -0.18 m
- Neutral value: 0.00 m
- Maximum: +0.06 m

A three-point color interpretation was used so that the neutral color corresponded explicitly to approximately zero LOS displacement.

The final map also included:

- DISS ITCS128 source-plane projection
- Amatrice
- Accumoli
- 24 August 2016 earthquake epicenter
- Scale bar
- North arrow
- CRS information
- Data-source information

The cartographic visualization was designed for scientific communication, technical portfolio presentation, and independent validation.

---

## 24. Independent Spatial Validation

The final displacement field was compared with the DISS 3.3.1 seismogenic-source database.

The following Composite Seismogenic Source was selected:

```text
ITCS128 — Accumoli-Amatrice
```

The source-plane projection was filtered using:

```text
"idsource" = 'ITCS128'
```

The final LOS displacement field was spatially compared with:

- the ITCS128 modeled source geometry,
- the 24 August 2016 mainshock epicentral area,
- Amatrice,
- Accumoli.

The deformation field showed spatial consistency with the principal coseismic deformation corridor and the modeled Accumoli-Amatrice seismogenic source geometry.

The DISS Composite Seismogenic Source was used as an independent structural reference and was not interpreted as a mapped surface rupture trace.

---

## 25. Final Processing Chain

The complete processing sequence implemented in this project was:

```text
Sentinel-1A IW SLC
        ↓
TOPS Split
        ↓
Precise Orbit Correction
        ↓
Back-Geocoding
        ↓
Enhanced Spectral Diversity
        ↓
Interferogram + Coherence
        ↓
TOPS Deburst
        ↓
Topographic Phase Removal
        ↓
Goldstein Phase Filtering
        ↓
Spatial Subset
        ↓
SNAPHU Export
        ↓
SNAPHU Phase Unwrapping
        ↓
SNAPHU Import
        ↓
Phase-to-Displacement
        ↓
Range-Doppler Terrain Correction
        ↓
Coherence Terrain Correction
        ↓
Raster Collocation
        ↓
Coherence Masking (γ ≥ 0.40)
        ↓
Stable-Reference Correction
        ↓
GeoTIFF Export
        ↓
Single-Band Raster Extraction
        ↓
Final Raster QA/QC
        ↓
QGIS Cartographic Visualization
        ↓
DISS-Based Spatial Validation
```

---

## Software and Technologies

- ESA SNAP
- SNAPHU 2.0.4
- QGIS
- GDAL
- Sentinel-1 SAR
- Interferometric Synthetic Aperture Radar (InSAR)
- Copernicus DEM GLO-30
- DISS 3.3.1
- GeoTIFF
- Geospatial QA/QC
- Earth Observation
- Remote Sensing

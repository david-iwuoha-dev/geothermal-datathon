# SPE Africa Geothermal Datathon 2026 — Methodology & Progress Log

**Team Repo:** geothermal-datathon  
**Language:** Python  
**Deadline:** 4 June 2026, 23:59 EAT  
**Submission portal:** https://forms.gle/trogXPPhgyvjSzGE7

---

## 1. Project Overview

The challenge requires assessing whether a geothermal resource from four wells in the
Netherlands (Utrecht region) can supply a mixed urban district with:
- At least **10 MWth heating demand**
- At least **5 MWth cooling demand**

Target geological formation: **Rotliegend sandstone**

Grading split:
- Challenge 1 — Geothermal Assessment: **60%**
- Challenge 2 — System Design: **40%**
- Bonus — AI-assisted workflow: extra marks

---

## 2. Data Inventory & Known Issues

### 2a. LAS Files (4 wells)
| Well | Depth Range | Unit | Notes |
|------|-------------|------|-------|
| BLT-01 | 0–2124 m | meters | Most data-rich; 20 curves; no missing data |
| EVD-01 | 0–2197 m | meters | Minimal curves: GR, DT, RHOB, DRHO |
| PKP-01 | 0–2751 m | meters | Deepest well |
| JUT-01 | 0–11220 ft | **FEET** | Deviated well — requires unit + TVD conversion |

### 2b. target_lithologies.csv
- 3,455 rows × 13 columns
- `depth_tvd_m`: **100% missing** — caused by JUT-01 deviated well (measured depth ≠ TVD)
- `porosity_pct`: **30% missing** (1,036 rows)
- `bulk_density_gcc`: **7.4% missing** (256 rows)
- `flag_reason`: mostly "AH depth — deviated well needs TVD conversion"

### 2c. Lithostratigraphic_Data.xlsx
- 4 sheets (one per well)
- Contains formation tops and bases
- Primary target: **Slochteren Formation** (Rotliegend) at BLT-01 (~1924–2053 m)
- Zechstein evaporites act as seal above Rotliegend in some wells

### 2d. ThermoGIS_Data.xlsx
- 4 sheets (one per well)
- Probabilistic reservoir properties at P90 / P50 / P10
- Most important dataset for Challenge 1

| Well | Perm P50 (mD) | Thickness P50 (m) | Porosity (%) | Temp (°C) | Flow P50 (m³/h) | Power P50 (MW) | Verdict |
|------|--------------|-------------------|--------------|-----------|-----------------|----------------|---------|
| BLT-01 | 82 | 130 | 17 | 77 | 105 | 5.1 | **Best candidate** |
| JUT-01 | 40 | 125 | 11 | 72 | 55 | 2.3 | Good secondary |
| EVD-01 | 6 | 76 | 9 | 72 | 0 | 0.0 | Poor |
| PKP-01 | 1 | 60 | 9 | 88 | 0 | 0.0 | High temp, near-zero flow |

### 2e. Well_Path_Data.xlsx
- Directional survey for each well: MD, Inclination, Azimuth, TVD, X-offset, Y-offset
- Used to convert JUT-01 measured depths to TVD

### 2f. LCOE.xlsx
- Excel model for Levelized Cost of Energy
- To be adapted in Notebook 07

---

## 3. Physical Constants & Assumptions Locked In

| Parameter | Value | Justification |
|-----------|-------|---------------|
| `RHO_W` | 1000.0 kg/m³ | Water density |
| `C_W` | 4182.0 J/kg·K | Specific heat of water |
| `MU_W` | 3×10⁻⁴ Pa·s | Viscosity of water at ~75°C |
| `T_REJ` | 30°C | Rejection / reinjection temperature |
| `T_SURF` | 10°C | Mean surface temperature, Netherlands |
| `COP_HP` | 4.0 | Heat pump coefficient of performance |
| `T_MIN_ABS` | 70°C | Minimum brine temp for absorption chiller |
| `COP_CHILLER` | 0.7 | Absorption chiller thermal COP |
| `RECOVERY_FACTOR` | 0.25 | Volumetric heat recovery factor (25%) |
| `PROJECT_LIFE_YR` | 30 | Project lifetime in years |
| `WELL_AREA_M2` | 5×10⁶ m² | Drained area per doublet (5 km²) |
| `UPLIFT_FACTOR` | COP/(COP-1) = 1.333 | Heat pump thermal uplift at COP 4 |

---

## 4. Notebook Progress Log

### Notebook 01 — Data Loading and QC ✅
- Loaded all 4 LAS files using `lasio`
- Loaded all xlsx files using `pandas` / `openpyxl`
- Loaded `target_lithologies.csv`
- Documented all data quality issues:
  - JUT-01 depths in feet → must multiply by 0.3048
  - JUT-01 is deviated → measured depth ≠ TVD
  - `depth_tvd_m` 100% missing in target_lithologies
  - 30% porosity missing, 7.4% bulk density missing

---

### Notebook 02 — Well Log Analysis ✅
- Multi-track log displays plotted for all 4 wells (GR, RHOB, DT, NPHI where available)
- Rotliegend formation intervals shaded using Lithostratigraphic data
- Lateral variability between wells identified
- Rotliegend sandstone identified by: low GR, low density, elevated porosity

---

### Notebook 03 — Missing Data and TVD Conversion ✅
- **TVD conversion:** Used Well Path Data directional survey to interpolate TVD for JUT-01
- **Unit conversion:** JUT-01 depths multiplied by 0.3048 (feet → meters)
- **Porosity imputation:** Missing values filled using GR and RHOB via regression
  (density-porosity equation: φ = (ρma - ρb) / (ρma - ρfl), ρma = 2.65 g/cc for sandstone)
- **Density imputation:** Missing bulk_density_gcc filled using sonic-density relationship or KNN
- Cleaned dataset saved to `outputs/processed_data/`

---

### Notebook 04 — Reservoir Characterization ✅
- Log-derived porosity computed from density logs
- Log-derived values compared to ThermoGIS — differences reconciled
- Permeability estimated using porosity-permeability transforms (Kozeny-Carman / empirical)
- Net pay thickness computed using GR cutoff to separate clean sand from shale
- Reservoir properties mapped spatially using easting/northing coordinates
- Output saved to `outputs/processed_data/reservoir_properties.csv`

**Reservoir properties summary (from log analysis):**
| Well | Net Pay (m) | Log Porosity (%) | Log Perm (mD) | Temp P50 (°C) | Power P50 (MW) |
|------|------------|-----------------|---------------|---------------|----------------|
| BLT-01 | 138.2 | 14.0 | 40.4 | 77 | 5.1 |
| EVD-01 | 130.5 | 10.6 | 12.3 | 72 | 0.0 |
| PKP-01 | 70.1 | 5.6 | 2.3 | 88 | 0.0 |
| JUT-01 | 840.8 | 4.6 | 0.0 | 72 | 2.3 |

---

### Notebook 05 — Geothermal Potential Assessment (Challenge 1) ✅

#### Bug Found and Fixed
- **Issue:** `get_thermo()` function used exact string match `'Flow rate'` (lowercase r)
  but ThermoGIS data contains `'Flow Rate'` (uppercase R)
- **Effect:** Flow rates returned NaN for all wells → cooling calculation silently returned 0.0 MW
- **Fix:** Made string comparison case-insensitive using `.str.lower()` on both sides of mask

```python
# FIXED version of get_thermo()
mask = (df_thermo['Well'] == well) & \
       (df_thermo['Property'].str.strip().str.lower() == property_name.lower())
```

#### Scenario Analysis Results

**Heating (target: 10 MWth)**
| Configuration | P90 (MW) | P50 (MW) | P10 (MW) | Meets target at P50? |
|---------------|----------|----------|----------|----------------------|
| BLT-01 only | 0.60 | 5.10 | 23.70 | NO |
| BLT-01 + JUT-01 | 1.60 | 7.40 | 28.50 | NO |
| All four wells | 1.60 | 7.40 | 28.50 | NO |

**With heat pump uplift (COP=4, uplift factor=1.333):**
| Configuration | Geo P50 (MW) | Boosted P50 (MW) | Elec needed (MW) | Meets target? |
|---------------|-------------|-----------------|-----------------|---------------|
| BLT-01 only | 5.10 | 6.80 | 1.70 | NO |
| BLT-01 + JUT-01 | 7.40 | 9.87 | 2.47 | NO (MARGINAL) |

**Final heating result: 9.9 MWth at P50 — MARGINAL**

**Cooling (target: 5 MWth)**
- Absorption chiller from BLT-01 at P50: **4.0 MW**
- Gap: **1.0 MW** to be covered by compression chiller in Notebook 06

#### Volumetric Heat in Place
| Well | HIIP (PJ) | Recoverable (PJ) | Avg Power (MW) |
|------|-----------|-----------------|----------------|
| BLT-01 | 27.11 | 6.78 | 7.16 |
| EVD-01 | 17.93 | 4.48 | 4.74 |
| PKP-01 | 6.40 | 1.60 | 1.69 |
| JUT-01 | 50.14 | 12.54 | 13.24 |

#### Challenge 1 Conclusion
1. Develop BLT-01 as primary production well (doublet configuration)
2. Develop JUT-01 as secondary well — adds meaningful flow at P50
3. Install heat pump with COP ≥ 4 to bridge gap to 10 MWth at P50
4. At P10 (optimistic), BLT-01 alone exceeds the heating target
5. EVD-01 and PKP-01 are not viable production wells at P50
6. PKP-01 (88°C) serves as temperature calibration reference only

---

## 5. Remaining Notebooks — To Do

| Notebook | Topic | Status | Priority |
|----------|-------|--------|----------|
| 06 | System Design (doublet, heat pump, chiller, storage) | ⏳ Next | HIGH — 40% of grade |
| 07 | LCOE Analysis | ⏳ Pending | HIGH |
| 08 | Bonus AI Workflow | ⏳ Pending | Medium |

### Key inputs going into Notebook 06
- Heating gap: **0.1 MW** — heat pump needs small safety margin above 9.9 MW
- Cooling gap: **1.0 MW** — compression chiller to top up absorption chiller output
- Production wells: **BLT-01** (primary) + **JUT-01** (secondary)
- Doublet config: one production well + one injection well per location
- Heat pump target supply temperature: **~70°C** for district heating

---

## 6. Figures Generated

| File | Notebook | Description |
|------|----------|-------------|
| `05_power_by_well_scenarios.png` | 05 | Thermal power by well — P90/P50/P10 with target lines |

---

*Last updated: Notebook 05 complete. Next: Notebook 06 — System Design.*
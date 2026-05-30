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

**Final Result: Both targets met.**

| Target | Required | Delivered |
|--------|----------|-----------|
| Heating | 10 MWth | **10.39 MWth** ✓ |
| Cooling | 5 MWth | **5.00 MWth** ✓ |
| LCOE | competitive | **€54.5/MWh** vs Dutch gas €60–80/MWh ✓ |

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
- Adapted and reproduced in Python in Notebook 07

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
| `COP_CHILLER` | 0.75 | Absorption chiller thermal COP |
| `COP_COMPRESSION` | 3.5 | Compression chiller COP |
| `RECOVERY_FACTOR` | 0.25 | Volumetric heat recovery factor (25%) |
| `PROJECT_LIFE_YR` | 30 | Project lifetime in years |
| `WELL_AREA_M2` | 5×10⁶ m² | Drained area per doublet (5 km²) |
| `UPLIFT_FACTOR` | COP/(COP-1) = 1.333 | Heat pump thermal uplift at COP 4 |
| `DISCOUNT_RATE` | 6% | Project financing rate |
| `HEATING_SEASON_H` | 5,040 h/yr | 6 full months + 2 shoulder months at 50% |
| `COOLING_SEASON_H` | 2,880 h/yr | 4 summer months |

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
- QC issues register saved to `outputs/processed_data/qc_issues_register.csv`

**Key outputs:**
- `outputs/figures/01_well_location_map.png`
- `outputs/figures/01_well_trajectories.png`
- `outputs/figures/01_well_depth_coverage.png`
- `outputs/figures/01_litho_distributions.png`
- `outputs/figures/01_thermo_power_comparison.png`

---

### Notebook 02 — Well Log Analysis ✅
- Multi-track log displays plotted for all 4 wells (GR, RHOB, DT, NPHI where available)
- Rotliegend formation intervals shaded using Lithostratigraphic data
- Lateral variability between wells identified
- Rotliegend sandstone identified by: low GR, low density, elevated porosity
- BLT-01 confirmed as highest quality reservoir visually

**Key outputs:**
- `outputs/figures/02_well_logs_BLT-01.png`
- `outputs/figures/02_well_logs_EVD-01.png`
- `outputs/figures/02_well_logs_JUT-01.png`
- `outputs/figures/02_well_logs_PKP-01.png`
- `outputs/figures/02_rotliegend_thickness.png`
- `outputs/figures/02_gr_rotliegend_comparison.png`
- `outputs/figures/02_rhob_rotliegend_comparison.png`
- `outputs/figures/02_blt01_rotliegend_detail.png`

---

### Notebook 03 — Missing Data and TVD Conversion ✅
- **TVD conversion:** Used Well Path Data directional survey to interpolate TVD for JUT-01
- **Unit conversion:** JUT-01 depths multiplied by 0.3048 (feet → meters)
- **Porosity imputation:** Missing values filled using GR and RHOB via regression
  (density-porosity equation: φ = (ρma - ρb) / (ρma - ρfl), ρma = 2.65 g/cc for sandstone)
- **Density imputation:** Missing bulk_density_gcc filled using sonic-density relationship or KNN
- Cleaned dataset saved to `outputs/processed_data/target_lithologies_clean.csv`

**Key outputs:**
- `outputs/processed_data/target_lithologies_clean.csv`
- `outputs/figures/03_tvd_conversion_check.png`
- `outputs/figures/03_porosity_validation.png`
- `outputs/figures/03_imputation_results.png`

---

### Notebook 04 — Reservoir Characterization ✅
- Log-derived porosity computed from density logs using:
  φ = (ρma - ρb) / (ρma - ρfl) where ρma = 2.65 g/cc (sandstone matrix)
- Permeability estimated using Kozeny-Carman equation
- Net pay thickness computed using GR cutoff to separate clean sand from shale
- Log-derived values compared to ThermoGIS — differences reconciled
- Reservoir properties mapped spatially using easting/northing coordinates

**Bug fixed in this notebook:**
- `get_thermo()` called with `'Flow rate'` (lowercase r) — fixed to `'Flow Rate'`
  matching the exact string in ThermoGIS data. This caused `Thermo_Flow_m3h_P50`
  to be NaN in the output CSV until corrected.

**Note on JUT-01 permeability:**
Log-derived permeability for JUT-01 returns 0.0 mD because NPHI and RHOB curves
are sparse in this well, limiting the Kozeny-Carman transform. The ThermoGIS P50
value of 40 mD is used as the primary permeability estimate for JUT-01 in all
downstream calculations.

**Reservoir properties summary:**
| Well | Net Pay (m) | Log Porosity (%) | Log Perm (mD) | Thermo Flow P50 (m³/h) | Temp P50 (°C) | Power P50 (MW) |
|------|------------|-----------------|---------------|------------------------|---------------|----------------|
| BLT-01 | 138.2 | 14.0 | 40.4 | 105 | 77 | 5.1 |
| EVD-01 | 130.5 | 10.6 | 12.3 | 0 | 72 | 0.0 |
| PKP-01 | 70.1 | 5.6 | 2.3 | 0 | 88 | 0.0 |
| JUT-01 | 840.8 | 4.6 | 0.0* | 55 | 72 | 2.3 |

*JUT-01 log perm = 0.0 mD due to sparse curves — ThermoGIS P50 (40 mD) used instead.

**Key outputs:**
- `outputs/processed_data/reservoir_properties.csv`
- `outputs/processed_data/log_derived_properties.csv`
- `outputs/figures/04_well_ranking.png`
- `outputs/figures/04_phi_perm_crossplot.png`
- `outputs/figures/04_log_vs_thermo_comparison.png`
- `outputs/figures/04_porosity_perm_profiles.png`

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
| BLT-01 + JUT-01 | 7.40 | 9.87 | 2.47 | MARGINAL |

**Final heating result: 9.9 MWth at P50 — MARGINAL**

> Note: Raw HP uplift gives 9.9 MW at P50. Full system design (Notebook 06)
> achieves 10.39 MWth after heat exchanger sizing and thermal storage contribution.

**Cooling (target: 5 MWth)**
- Absorption chiller from BLT-01 at P50: **4.0 MW**
- Gap of 1.0 MW closed by compression chiller in Notebook 06

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

**Key outputs:**
- `outputs/processed_data/thermal_power_summary.csv`
- `outputs/processed_data/scenario_results.csv`
- `outputs/processed_data/hp_uplift_results.csv`
- `outputs/processed_data/heat_in_place.csv`
- `outputs/figures/05_power_by_well_scenarios.png`
- `outputs/figures/05_energy_balance.png`
- `outputs/figures/05_config_vs_target.png`
- `outputs/figures/05_temp_vs_flow.png`

---

### Notebook 06 — System Design (Challenge 2) ✅

Designed the full integrated district energy system using a doublet configuration
for both production wells. All components sized for the P50 base case.

#### System Components

| Component | Capacity | Role |
|-----------|----------|------|
| BLT-01 Production Well (doublet) | 105 m³/h @ 77°C | Primary geothermal heat source |
| JUT-01 Production Well (doublet) | 55 m³/h @ 72°C | Secondary geothermal heat source |
| Primary Heat Exchanger | 10.4 MWth | Transfer geothermal heat to district network |
| Heat Pump (geothermal-source) | 2.99 MWth / 0.748 MWe | COP 4.0 — bridges heating gap |
| Absorption Chiller (single-effect) | 1.15 MWth cooling | Geothermal-driven cooling at COP 0.75 |
| Compression Chiller (top-up) | 3.85 MWth / 1.10 MWe | Cooling top-up at COP 3.5 |
| Thermal Storage Tank | 1,342 m³ / 31.2 MWh | Peak demand buffer; enables steady well operation |

#### Design Decisions
- Doublet configuration for both BLT-01 and JUT-01 — one production + one injection well each
- Injection wells return cooled brine to maintain reservoir pressure and sustainability
- Thermal storage sized for 3-hour peak shaving — prevents flow rate cycling
- Absorption chiller preferred for base cooling load — uses waste brine heat directly
- Compression chiller retained as top-up only (1.0 MW) to minimise grid electricity dependency
- Heat pump supply temperature target: 70°C for district heating network

#### Challenge 2 Result
- **Heating delivered: 10.39 MWth** (target 10 MWth — MET ✓)
- **Cooling delivered: 5.00 MWth** (target 5 MWth — MET ✓)

**Key outputs:**
- `outputs/processed_data/system_design_summary.csv`
- `outputs/processed_data/annual_energy_balance.csv`
- `outputs/figures/06_system_schematic.png`
- `outputs/figures/06_energy_balance.png`

---

### Notebook 07 — LCOE Analysis ✅

Reproduced the LCOE Excel model in Python for full reproducibility.

**LCOE = (CAPEX × CRF + OPEX_annual) / E_annual**

Where CRF = i(1+i)^n / ((1+i)^n - 1)

#### Bug Fixed
- **Issue:** P10 scenario used a hard cap of 10.5 MW on heat output — P10 delivered
  the same annual energy as P50 → LCOE P50 = €63.7 and P10 = €63.2 (suspicious €0.50 gap)
- **Fix:** Removed the 10.5 MW cap from the scenario loop so P10 reflects full
  deliverable output. P10 now correctly shows €24.6/MWh.

#### CAPEX Note
CAPEX includes drilling and completion for both production AND injection wells:
- BLT-01 doublet = 2 wells | JUT-01 doublet = 2 wells | Total = 4 wells in CAPEX

#### Economic Results

| Metric | Value |
|--------|-------|
| Total CAPEX | €30.34 M |
| Annual OPEX | €1,433.6 k/yr |
| Annual energy delivered | 66,759 MWh/yr |
| Annual electricity consumed | 6,938 MWh/yr |
| Discount rate | 6% |
| Project lifetime | 30 years |
| Capital Recovery Factor | 0.0726 |
| **LCOE base case** | **€54.5/MWh** |
| LCOE P90 (pessimistic) | €125.1/MWh |
| LCOE P50 (base) | €63.7/MWh |
| LCOE P10 (optimistic) | €24.6/MWh |

**Key outputs:**
- `outputs/processed_data/lcoe_results.csv`
- `outputs/processed_data/capex_breakdown.csv`
- `outputs/figures/07_lcoe_scenarios.png`
- `outputs/figures/07_lcoe_vs_discount_rate.png`
- `outputs/figures/07_tornado_chart.png`
- `outputs/figures/07_cost_breakdown.png`

---

### Notebook 08 — Bonus AI Workflow ✅

Used the Groq API (LLaMA 3.3 70B) to automate two geothermal interpretation tasks.
API key loaded from `.env` file — never hardcoded.

**Model:** `llama-3.3-70b-versatile` | **Temperature:** 0.3 | **Provider:** Groq (free tier)

#### Task 1 — Automated Reservoir Description
- **Input:** `outputs/processed_data/reservoir_properties.csv`
- **Output:** `outputs/processed_data/ai_reservoir_description.txt`
- BLT-01: Good — viable at P50 | EVD-01: Poor | PKP-01: Poor | JUT-01: Fair

#### Task 2 — Automated Risk Register
- **Input:** `lcoe_results.csv` + `system_design_summary.csv`
- **Output:** `outputs/processed_data/ai_risk_assessment.txt`
- Covers: Technical · Economic · Operational · Regulatory/Environmental risks
- Overall project risk rating: **MEDIUM**

**Key outputs:**
- `outputs/processed_data/ai_reservoir_description.txt`
- `outputs/processed_data/ai_risk_assessment.txt`

---

## 5. All Bugs Found and Fixed

| # | Bug | Location | Effect | Fix |
|---|-----|----------|--------|-----|
| 1 | `get_thermo()` string match `'Flow rate'` vs `'Flow Rate'` | NB 04 & 05 | Flow rates NaN → cooling showed 0.0 MW | `.str.lower()` on both sides |
| 2 | `Thermo_Flow_m3h_P50` NaN in reservoir CSV | NB 04 | Column empty in output file | Fixed via Bug 1 above |
| 3 | LCOE scenario loop capped heat at 10.5 MW | NB 07 | P10 LCOE = €63.2 (should be ~€24) | Removed cap — P10 now €24.6/MWh |

---

## 6. Submission Checklist

| Deliverable | Filename | Status |
|-------------|----------|--------|
| Code repository (zip) | `Team_Apex_Code_V1.zip` | ready |
| Technical presentation | `Team_Apex_PPT_V1.pdf` | ready |
| 5-minute video | `Team_Apex_Vid_V1` | ready |

**Zip command:**
```bash
zip -r Team_Apex_Code_V1.zip . --exclude ".env" --exclude ".git/*" --exclude "*.pyc" --exclude "__pycache__/*"
```

**Submit at:** https://forms.gle/trogXPPhgyvjSzGE7
**Deadline:** 4 June 2026, 23:59 EAT

---


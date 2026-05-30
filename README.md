# SPE Africa Geothermal Datathon 2026

**Team:** Team Apex
**Competition:** SPE Africa DSEATS Geothermal Datathon 2026  
**Deadline:** 4 June 2026, 23:59 EAT  
**Submission portal:** https://forms.gle/trogXPPhgyvjSzGE7  
**Language:** Python  

---

## What This Project Does

This project assesses whether a geothermal resource from four wells in the Netherlands (Utrecht region) can reliably supply a mixed urban district with **10 MWth heating** and **5 MWth cooling** from the Rotliegend sandstone formation.

**Result: Both targets are met.**

| Target | Required | Delivered |
|--------|----------|-----------|
| Heating | 10 MWth | **10.39 MWth** ✓ |
| Cooling | 5 MWth | **5.00 MWth** ✓ |
| LCOE | competitive | **€54.5/MWh** (vs Dutch gas €60–80/MWh) ✓ |

---

## Grading Breakdown

| Challenge | Weight | Notebooks |
|-----------|--------|-----------|
| Challenge 1 — Geothermal Assessment | 60% | 01, 02, 03, 04, 05 |
| Challenge 2 — System Design + LCOE | 40% | 06, 07 |
| Bonus — AI-assisted workflow | Extra | 08 |

---

## Repository Structure

```
geothermal-datathon/
│
├── README.md                         ← You are here
├── requirements.txt                  ← All Python dependencies
│
├── data/
│   ├── raw/                          ← Original LAS well logs — DO NOT MODIFY
│   │   ├── BLT-01.las
│   │   ├── EVD-01.las
│   │   ├── PKP-01.las
│   │   └── JUT-01.las
│   ├── target_lithologies.csv        ← Pre-processed Rotliegend intervals
│   ├── ThermoGIS_Data.xlsx           ← Probabilistic reservoir properties (P90/P50/P10)
│   ├── Lithostratigraphic_Data.xlsx  ← Formation tops & bases per well
│   ├── Well_Path_Data.xlsx           ← Directional survey for TVD conversion
│   └── LCOE.xlsx                     ← Excel economic model
│
├── notebooks/
│   ├── 01_data_loading_and_qc.ipynb
│   ├── 02_well_log_analysis.ipynb
│   ├── 03_missing_data_and_tvd.ipynb
│   ├── 04_reservoir_characterization.ipynb
│   ├── 05_geothermal_potential.ipynb
│   ├── 06_system_design.ipynb
│   ├── 07_lcoe_analysis.ipynb
│   └── 08_bonus_ai_workflow.ipynb
│
├── outputs/
│   ├── figures/                      ← All plots saved here (PNG)
│   └── processed_data/               ← Cleaned datasets and AI outputs (CSV, TXT)
│
├── src/
│   └── __init__.py                  ← reserved for shared utility functions; 
|                                      all analysis logic is self-contained within notebooks for reproducibility
│                                         
│
└── docs/
    └── methodology.md                ← Full technical decisions, assumptions, and progress log
```

---

## How to Run

### 1. Clone the repo
```bash
git clone https://github.com/david-iwuoha-dev/geothermal-datathon.git
cd geothermal-datathon
```

### 2. Set up the environment
```bash
python -m venv venv
source venv/bin/activate        # Linux/Mac
venv\Scripts\activate           # Windows

pip install -r requirements.txt
```

### 3. Set up the API key (for Notebook 08 only)
Create a `.env` file in the project root:
```
GROQ_API_KEY="your_groq_api_key_here"
```
Get a free key at https://console.groq.com

### 4. Run notebooks in order
Open Jupyter and run each notebook top-to-bottom in sequence:
```bash
jupyter notebook
```
Run: `01` → `02` → `03` → `04` → `05` → `06` → `07` → `08`

> **Important:** Each notebook depends on outputs from the previous one.  
> Do not skip notebooks or run them out of order.

---

## What Each Notebook Does

### Notebook 01 — Data Loading and QC
Loads all 4 LAS files, all Excel files, and the target lithologies CSV. Prints data shapes, curve names, depth ranges. Documents all data quality issues found.

**Key outputs:**
- Console summary of all data issues
- `outputs/figures/01_well_location_map.png`
- `outputs/figures/01_well_trajectories.png`
- `outputs/figures/01_well_depth_coverage.png`

---

### Notebook 02 — Well Log Analysis
Plots multi-track log displays for all 4 wells (GR, RHOB, DT, NPHI). Shades Rotliegend formation intervals. Compares logs between wells.

**Key outputs:**
- `outputs/figures/02_well_logs_BLT-01.png` (and EVD-01, PKP-01, JUT-01)
- `outputs/figures/02_rotliegend_thickness.png`
- `outputs/figures/02_gr_rotliegend_comparison.png`

---

### Notebook 03 — Missing Data and TVD Conversion
Fixes three data problems:
1. **JUT-01 unit conversion** — depths in feet multiplied by 0.3048
2. **JUT-01 TVD conversion** — directional survey used to compute true vertical depth
3. **Porosity imputation** — 30% missing values filled using GR + RHOB regression
4. **Density imputation** — 7.4% missing RHOB filled using DT relationship

**Key outputs:**
- `outputs/processed_data/target_lithologies_clean.csv`
- `outputs/figures/03_tvd_conversion_check.png`
- `outputs/figures/03_porosity_validation.png`

---

### Notebook 04 — Reservoir Characterization
Computes log-derived porosity and permeability. Compares against ThermoGIS values. Computes net pay thickness. Maps reservoir properties spatially.

**Key outputs:**
- `outputs/processed_data/reservoir_properties.csv` ← used by all subsequent notebooks
- `outputs/figures/04_well_ranking.png`
- `outputs/figures/04_phi_perm_crossplot.png`
- `outputs/figures/04_log_vs_thermo_comparison.png`

**Reservoir summary:**
| Well | Net Pay (m) | Porosity (%) | Perm (mD) | Temp (°C) | Verdict |
|------|------------|--------------|-----------|-----------|---------|
| BLT-01 | 138.2 | 14.0 | 40.4 | 77 | PRIMARY — best candidate |
| EVD-01 | 130.5 | 10.6 | 12.3 | 72 | Non-viable at P50 |
| PKP-01 | 70.1 | 5.6 | 2.3 | 88 | Non-viable — zero flow |
| JUT-01 | 840.8 | 4.6 | 0.0 | 72 | SECONDARY — adds 2.3 MW |

---

### Notebook 05 — Geothermal Potential (Challenge 1 core — 60% of grade)
Calculates thermal power across P90/P50/P10 scenarios. Compares configurations against the 10 MWth target. Assesses cooling feasibility.

**Key outputs:**
- `outputs/processed_data/thermal_power_summary.csv`
- `outputs/processed_data/scenario_results.csv`
- `outputs/figures/05_power_by_well_scenarios.png`
- `outputs/figures/05_energy_balance.png`

**Challenge 1 conclusion:**
- BLT-01 + JUT-01 at P50 = 7.4 MW (raw)
- With heat pump COP 4.0 = **9.9 MW** (marginal, within engineering tolerance)
- Absorption chiller from BLT-01 = **4.0 MW** cooling (gap of 1.0 MW closed in Notebook 06)
- At P10, BLT-01 alone = 23.7 MW — comfortably exceeds target

---

### Notebook 06 — System Design (Challenge 2 core — 40% of grade)
Designs the full integrated energy system. Sizes heat pumps, chillers, and thermal storage. Produces energy balance diagram.

**System components:**
| Component | Capacity | Role |
|-----------|----------|------|
| BLT-01 Production Well (doublet) | 105 m³/h @ 77°C | Primary heat source |
| JUT-01 Production Well (doublet) | 55 m³/h @ 72°C | Secondary heat source |
| Primary Heat Exchanger | 10.4 MWth | Geo → district transfer |
| Heat Pump | 2.99 MWth / 0.748 MWe | COP 4.0 — heating bridge |
| Absorption Chiller | 1.15 MWth cooling | Geo-driven cooling |
| Compression Chiller | 3.85 MWth / 1.10 MWe | Cooling top-up |
| Thermal Storage Tank | 1,342 m³ / 31.2 MWh | Peak demand buffer |

**Key outputs:**
- `outputs/processed_data/system_design_summary.csv`
- `outputs/figures/06_system_schematic.png`
- `outputs/figures/06_energy_balance.png`

---

### Notebook 07 — LCOE Analysis
Adapts the provided Excel LCOE model in Python. Calculates Levelized Cost of Energy. Runs sensitivity analysis across P90/P50/P10 scenarios.

**Key outputs:**
- `outputs/processed_data/lcoe_results.csv`
- `outputs/processed_data/capex_breakdown.csv`
- `outputs/figures/07_lcoe_scenarios.png`
- `outputs/figures/07_tornado_chart.png`

**Economic results:**
| Metric | Value |
|--------|-------|
| Total CAPEX | €30.34 M |
| Annual OPEX | €1,433.6 k/yr |
| LCOE base case | **€54.5/MWh** |
| Dutch gas benchmark | €60–80/MWh |
| Project lifetime | 30 years |
| Discount rate | 6% |

---

### Notebook 08 — Bonus AI Workflow
Uses the Groq API (LLaMA 3.3 70B) to automate two interpretation tasks:

1. **Automated reservoir description** — feeds `reservoir_properties.csv` to the LLM, receives a professional well-by-well written interpretation
2. **Automated risk register** — feeds `lcoe_results.csv` + `system_design_summary.csv`, receives a structured risk register across technical, economic, operational, and regulatory categories

**Key outputs:**
- `outputs/processed_data/ai_reservoir_description.txt`
- `outputs/processed_data/ai_risk_assessment.txt`

> Requires `GROQ_API_KEY` in `.env` file. Free tier available at https://console.groq.com

---

## For Teammates — What we need for a complete submission

### Pitch Deck has been prepared proper

### Video (Team_Apex_Vid_V1)
- 3–5 minutes long
- At least one person on camera or clear voice narration
- 720p minimum resolution
- Walk through: problem → data → key findings → system design → LCOE → AI workflow → conclusion
- Use the slide deck as your visual aid

---

### Code Zip (Team_Apex_Code_V1.zip)
Before zipping:
- Make sure all notebooks run top-to-bottom without errors
- Remove `.env` from the zip (contains your API key)
- Confirm `README.md` and `requirements.txt` are present

```bash
# From the project root — exclude .env and .git
zip -r Team_Apex_Code_V1.zip . --exclude ".env" --exclude ".git/*"
```

---

## Known Issues & Fixes Applied

| Issue | Location | Fix Applied |
|-------|----------|-------------|
| JUT-01 depths in feet | LAS file | Multiplied by 0.3048 in Notebook 03 |
| JUT-01 deviated well — MD ≠ TVD | target_lithologies.csv | Minimum curvature interpolation in Notebook 03 |
| `depth_tvd_m` 100% missing | target_lithologies.csv | Computed from directional survey in Notebook 03 |
| 30% missing porosity | target_lithologies.csv | Imputed via density-porosity equation in Notebook 03 |
| `get_thermo()` case mismatch bug | Notebook 05 | Fixed `.str.lower()` on both sides of string comparison |
| Cooling showed 0.0 MW (caused by above bug) | Notebook 05 | Fixed — now correctly shows 4.0 MW from absorption chiller |

---

## Dependencies

All dependencies are in `requirements.txt`. Key libraries:

| Library | Purpose |
|---------|---------|
| `lasio` | Read LAS well log files |
| `pandas` / `numpy` | Data wrangling |
| `matplotlib` / `plotly` | Visualisation |
| `scikit-learn` | ML-based imputation |
| `scipy` | TVD interpolation |
| `openpyxl` / `xlrd` | Read Excel files |
| `groq` | Groq API client (Notebook 08) |
| `python-dotenv` | Load `.env` API key |

---

## Technical Documentation

For full methodology, physical constants, assumptions, and a detailed notebook-by-notebook progress log, see:

**`docs/methodology.md`**

---

*SPE Africa DSEATS Geothermal Datathon 2026 — geothermal-datathon repo*
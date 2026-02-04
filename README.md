[![Java 11-21](https://img.shields.io/badge/Java-11--21-orange?logo=openjdk)](https://adoptium.net/) [![Python 3.7+](https://img.shields.io/badge/Python-3.7+-blue?logo=python&logoColor=white)](https://python.org) [![Synthea](https://img.shields.io/badge/Synthea-Synthetic%20Health-green?logo=github)](https://github.com/synthetichealth/synthea) [![FHIR R4](https://img.shields.io/badge/FHIR-R4-red?logo=fire)](https://hl7.org/fhir/R4/) [![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE) [![BC Health](https://img.shields.io/badge/Region-Fraser%20Health%20BC-purple)](https://www.fraserhealth.ca/)

# bc-synthetic-health

A Jupyter Notebook here has everything to use Synthea as a state machine to simulate protocol-based, longitudinal data of the Canadian health system, specifically targeting Fraser Health (British Columbia).

## Overview

This repository contains two Jupyter notebooks for generating and validating synthetic health data for Fraser Health:

1. **`fraser_health_onboarding.ipynb`** - Interactive controller for generating synthetic health data using the Synthea simulator
2. **`fraser_health_validation_tests.ipynb`** - Comprehensive automated testing and clinical validation suite

### Key Features

- **Uses Existing Assets**: Leverages geography and provider data from synthea-international (no external APIs)
- **Fraser Health Focus**: Targets British Columbia cities (Surrey, Burnaby, New Westminster, Coquitlam)
- **Interactive Workflow**: Step-by-step notebook with data inspection, filtering, simulation, and validation
- **Comprehensive Validation**: Automated testing suite for demographic parity, geographic integrity, clinical timelines, and FHIR R4 compliance

## Prerequisites

Before running the notebook, ensure you have:

1. **Java 11-21 recommended** - Required to run Synthea
   - ⚠️ **Java 22+ may cause issues**: Java 25+ triggers Gradle build errors ("Unsupported class file major version 69")
   - The notebook auto-detects Java 17 at `/usr/lib/jvm/java-17-openjdk-amd64` if available
   - Download from [Adoptium](https://adoptium.net/)
   - Verify installation: `java -version`

2. **Python 3.7+** with the following packages:
   - pandas
   - matplotlib
   - numpy
   - ipywidgets
   - jupyter
   - haversine (for validation suite)
   - fhir.resources (for FHIR validation)
   - scipy (for validation suite)

3. **Git** - For cloning repositories

## Quick Start

1. **Clone this repository**:
   ```bash
   git clone https://github.com/AndrewMichael2020/bc-synthetic-health.git
   cd bc-synthetic-health
   ```

2. **Launch Jupyter Notebook**:
   ```bash
   jupyter notebook fraser_health_onboarding.ipynb
   ```

3. **Run the notebook cells sequentially**:
   - The notebook will automatically clone synthea and synthea-international repos
   - It will discover and display Canadian geography/provider data schemas
   - Filter data for Fraser Health cities
   - Run the simulation
   - Validate and visualize results

## Notebook Structure

### Section 1: Environment Setup & Repo Inspection
- Checks Java installation
- Clones synthea and synthea-international repositories
- Discovers Canada assets in `ca/src/main/resources/`
- Displays schemas for:
  - `zipcodes_ca.csv` (geography data)
  - `demographics_ca.csv` (population demographics)
  - `hospitals_ca.csv` (healthcare facilities)

### Section 2: Configuration & Filtering
- Configuration parameters:
  - `TARGET_REGION`: "British Columbia"
  - `TARGET_CITIES`: ['Surrey', 'Burnaby', 'New Westminster', 'Coquitlam']
  - `POPULATION_SIZE`: 100 (adjustable)
  - `RANDOM_SEED`: 12345 (for reproducibility)
- Filters existing data for target region and cities
- Saves prepared CSVs to `config/` directory

### Section 3: Simulation Execution
- Creates non-destructive Canadian configuration pointing to synthea-international data
- Runs Synthea simulation with population-weighted distribution across all Fraser Health cities:
  ```bash
  java -jar synthea-with-dependencies.jar -c config/fraser_health.properties -p 10000 -s 12345 "British Columbia"
  ```
- **BC/CIHI Taxonomy Mapping**: Automatically applies Canadian healthcare terminology mapping (see [Taxonomy Mapping](#bccihi-taxonomy-mapping) section)
- Previews all generated CSV files with summary statistics

### Section 4: Validation
- Loads generated `patients.csv` and `encounters.csv`
- Verifies location data matches target cities
- Creates comprehensive visualization dashboard:
  - Patient age distribution
  - Encounter type distribution
  - Age vs. encounter type analysis
  - Summary statistics

## Configuration

You can customize the simulation by modifying the configuration cell:

```python
TARGET_REGION = "British Columbia"
TARGET_STATE_CODE = "BC"
TARGET_CITIES = ['Surrey', 'Burnaby', 'New Westminster', 'Coquitlam']
POPULATION_SIZE = 100  # Adjust as needed
RANDOM_SEED = 12345    # For reproducibility
```

## Output

Generated synthetic data is saved to `synthea/output/` including:
- `csv/` directory:
  - `patients.csv` - Patient demographics
  - `encounters.csv` - Healthcare encounters
  - `conditions.csv` - Diagnosed conditions
  - `medications.csv` - Prescribed medications
  - And more...
- `fhir/` directory:
  - FHIR R4 JSON bundles for each patient

Example of 100K patients at Fraser Health service area:

<img width="1384" height="911" alt="image" src="https://github.com/user-attachments/assets/b717065e-9bca-416d-9465-ff2507ab93a2" />


## Validation Suite

After generating data, run the **`fraser_health_validation_tests.ipynb`** notebook to perform comprehensive validation:

### Validation Categories

#### 1. Statistical Demographic Parity
- **Age Distribution**: Compares median age against BC 2021 Census baseline (42.8 years)
- **Ethnicity Distribution**: Verifies representation of South Asian and East Asian populations expected in Fraser Health cities

#### 2. Geographic Integrity & Referral Routing
- **Fraser Health Boundary Check**: Ensures patients reside in target cities (Surrey, Burnaby, New Westminster, Coquitlam)
- **Provider-Patient Location Matching**: Validates providers are within Fraser Health boundaries
- **Haversine Distance Check**: Calculates distances between patient homes and providers, flags "teleporting" patients (>100km)

#### 3. Clinical Timeline & FHIR R4 Compliance
- **Encounter Sequence Validation**: Checks logical ordering of encounters (e.g., inpatient preceded by ambulatory/emergency)
- **FHIR Bundle Validation**: Validates JSON outputs conform to FHIR R4 standards
- **BC Address Verification**: Ensures all FHIR resources contain "BC" as the state value

#### 4. Fraser Health Specific Stress Test
- **Top Conditions Analysis**: Identifies most common diagnoses and verifies expected chronic conditions
- **Multi-Seed Consistency**: Template for running simulations with different seeds to test stability

### Running Validation

1. **Generate data first**:
   ```bash
   jupyter notebook fraser_health_onboarding.ipynb
   ```
   Run all cells to generate synthetic data.

2. **Run validation**:
   ```bash
   jupyter notebook fraser_health_validation_tests.ipynb
   ```
   Run all cells to validate the generated data.

3. **Review results**:
   The notebook generates:
   - `validation_report.md` - Human-readable Markdown report
   - `validation_results.json` - Detailed JSON results
   - `validation_results.csv` - Tabular test results
   - Console output with clear PASS/FAIL/WARN indicators

### Interpreting Validation Results

- ✓ **PASS**: Test met all criteria
- ⚠ **WARN**: Test passed but with minor concerns or missing optional data
- ✗ **FAIL**: Test did not meet criteria - requires attention

## Data Sources

The notebook uses data from:
- **Synthea**: [github.com/synthetichealth/synthea](https://github.com/synthetichealth/synthea)
- **Synthea-International**: [github.com/synthetichealth/synthea-international](https://github.com/synthetichealth/synthea-international)
  - Canadian geography data from `ca/src/main/resources/geography/`
  - Canadian provider data from `ca/src/main/resources/providers/`

## Troubleshooting

### Java not found
Ensure Java 11+ is installed and in your PATH:
```bash
java -version
```

### Java 25+ Gradle Compatibility Error
If you see `Unsupported class file major version 69`, your Java version is too new for Gradle. The notebook auto-detects and uses Java 17 if available:
```bash
# Install Java 17 (Ubuntu/Debian)
sudo apt install openjdk-17-jdk

# The notebook will automatically use /usr/lib/jvm/java-17-openjdk-amd64
```

### Synthea build fails
If the JAR build fails, try building manually with Java 17:
```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
cd synthea
./gradlew uberJar
```

### Cities not found in data
The notebook will show available cities if target cities aren't found. You can modify `TARGET_CITIES` based on the output.

### Payer/Insurance Errors
Canadian simulations require custom payer files for BC's single-payer system (MSP). The notebook automatically creates these in `config/payers/`. If you see payer-related errors, ensure:
- `insurance_companies_ca.csv` has `Ownership` = `Government` (not `GOV`)
- Column headers match Synthea's exact format (including the typo `State Headquarterd`)

## Technical Notes

### Non-Destructive Configuration
The notebook uses a **non-destructive approach** - it does not modify Synthea's built-in US configuration files. Instead:
1. Creates a custom `fraser_health.properties` in the `config/` directory
2. Uses the `-c` flag to pass this config to Synthea
3. Points to Canadian data files in `synthea-international/ca/`
4. Creates minimal BC MSP payer files for Canada's single-payer system

This means you can switch between US and Canadian simulations by simply including or omitting the `-c` flag.

### Population-Weighted Multi-City Distribution
When `city=None` is passed to the simulation function, Synthea distributes patients across **all cities** in the demographics file based on population weights. This eliminates the need for multiple simulation runs and provides a more realistic geographic distribution.

### BC/CIHI Taxonomy Integration
The `BC_CLASS` column is added to `encounters.csv` immediately after simulation, before any preview or validation cells run. This ensures:
- All CSV previews show Canadian terminology
- Validation dashboards use FHA-aligned categories
- Downstream consumers don't need separate translation logic

## Canadian Healthcare Alignment

### What Aligns Well
- ✅ **Single-Payer System**: BC MSP configured with $0 out-of-pocket costs
- ✅ **Geography**: Uses Canadian postal codes (V4A format), correct province names
- ✅ **Demographics**: Based on Canadian Census data from synthea-international
- ✅ **Providers**: Canadian hospitals and primary care facilities
- ✅ **BC/CIHI Taxonomy**: Encounter types mapped to CIHI/BC MoH standards (see below)
- ✅ **Multi-City Distribution**: Patients distributed across all Fraser Health cities by population weight

### BC/CIHI Taxonomy Mapping

Synthea generates encounter types using US healthcare terminology. The notebook automatically maps these to **Canadian Institute for Health Information (CIHI)** and **BC Ministry of Health** standards, which is critical for Fraser Health Authority reporting and data governance.

| Synthea (US) | BC/CIHI Alias | Data Source / Context |
|--------------|---------------|----------------------|
| `ambulatory` | `GP_Office` | MSP Fee-for-Service Claims |
| `wellness` | `Annual_Exam` | Preventative Care |
| `outpatient` | `Hosp_Clinic` | NACRS Level 1/2 Ambulatory Care |
| `urgentcare` | `UPCC` | Urgent Primary Care Centre (BC strategic priority) |
| `emergency` | `ER_Dept` | NACRS Level 3 Emergency |
| `inpatient` | `Acute_Inpatient` | DAD (Discharge Abstract Database) |
| `virtual` | `Telehealth` | Virtual Health Services |
| `snf` | `LongTerm_Care` | Residential/Long-Term Care |
| `hospice` | `Hospice` | Palliative Care |
| `home` | `Home_Health` | Community Care Services |

**FHA Budgeting Context:**
- **MSP Fee-for-Service**: `GP_Office`, `Annual_Exam`, `Telehealth` – billed to Medical Services Plan
- **Health Authority Global Budget**: `Hosp_Clinic`, `ER_Dept`, `Acute_Inpatient`, `UPCC` – funded through FHA operating budget

The mapping is applied immediately after simulation, adding a `BC_CLASS` column to `encounters.csv` so all downstream analysis uses Canadian terminology.

### Encounter Volume Analysis

#### Is the Volume Ratio Correct?

**Yes, the shape is correct, but the density is likely low.**

With a typical simulation of 10,000 patients, you may observe:
- ~35,000 Ambulatory + ~15,000 Wellness encounters
- ~3,000 Emergency encounters
- **Ratio**: ~16:1 Primary Care to Acute Care

This aligns with the **preventative care models** Fraser Health champions, where most patient interactions occur in community settings rather than acute care.

#### Density Audit

| Metric | Simulated | Real-World BC Benchmark |
|--------|-----------|------------------------|
| Encounters per patient-year | ~1.3 | 3–5 visits/year |
| Primary Care : Acute ratio | ~16:1 | ~10-15:1 |

**Interpretation:**
- The simulation **under-represents minor acute episodes** (colds, minor injuries, follow-ups) that drive high-frequency GP visits
- The "Wellness" module follows a strictly annual/bi-annual cadence without enough intervening "sick" visits
- For production dashboards, consider scaling encounter counts by ~3x to match BC utilization patterns

#### Recommendations for Realistic Density
1. Increase `POPULATION_SIZE` to generate a larger encounter base
2. Consider post-processing to inject additional GP visits based on BC utilization rates
3. Use multiple simulation seeds and aggregate for statistical stability

### Systems Architecture: Data Governance

From a platform perspective, using raw Synthea categories for an FHA dashboard creates **"Data Governance Debt"** – misaligned terminology that requires constant translation downstream.

| Layer | Issue | Solution in This Notebook |
|-------|-------|---------------------------|
| **ETL/Bronze** | Raw Synthea `ENCOUNTERCLASS` values | Preserved in original column |
| **Transform/Silver** | Need BC-aligned categories | `BC_CLASS` column added automatically |
| **Reporting/Gold** | FHA dashboards expect CIHI terms | Use `BC_CLASS` for all visualizations |

The notebook addresses this at the **Silver layer** by applying the `BC_TAXONOMY_MAP` immediately after data generation, ensuring all downstream consumers (dashboards, validation, exports) work with Canadian terminology.

### Known Limitations
- ⚠️ **Clinical Modules**: Disease prevalence models are US-based; BC-specific epidemiology may differ
- ⚠️ **Medications**: Uses US drug names/NDCs; Canada uses DIN (Drug Identification Numbers)
- ⚠️ **Wait Times**: Does not model Canadian healthcare wait times for specialists/procedures
- ⚠️ **Encounter Density**: Simulated visit frequency (~1.3/year) is lower than real BC utilization (~3-5/year)

### Recommendations for Production Use
1. Validate generated conditions against BC CDC prevalence data
2. Map medication codes to Health Canada DIN where FHIR export is required
3. Consider adding BC-specific provider types to the providers CSV
4. For realism, run multiple simulations with different seeds and aggregate

## Next Steps

### Planned Enhancements
1. ~~**Multi-City Generation**~~: ✅ Implemented - patients now distributed across all Fraser Health cities by population weight
2. ~~**BC Taxonomy Mapping**~~: ✅ Implemented - encounter types mapped to CIHI/BC MoH standards
3. **BC-Specific Modules**: Develop Synthea modules for BC-specific conditions (e.g., higher rates of certain cancers, regional health patterns)
4. **DIN Medication Mapping**: Create a lookup table to convert US NDC codes to Canadian DIN codes
5. **FHIR CA Core Profile**: Validate outputs against Canadian FHIR profiles (CA Core)
6. **Wait Time Modeling**: Add realistic wait times for specialist referrals based on BC health data
7. **Encounter Density Calibration**: Scale visit frequencies to match BC utilization rates (~3-5 visits/year)

### Community Contributions Welcome
- Additional BC/Canadian provider data
- Validation against real (anonymized) BC health statistics
- French language support for Quebec adaptation
- Other provincial health authority configurations (e.g., Vancouver Coastal Health, Interior Health)

## License

This project uses:
- Synthea: Apache License 2.0
- Synthea-International: Apache License 2.0

## Acknowledgments

- [SyntheaTM](https://synthetichealth.github.io/synthea/) by MITRE Corporation
- synthea-international repository contributors

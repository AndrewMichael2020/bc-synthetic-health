# bc-synthetic-health

A Jupyter Notebook that uses Synthea as a state machine to simulate a portion of the Canadian health system, specifically targeting Fraser Health (British Columbia).

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

1. **Java 11 or higher** - Required to run Synthea
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
- Deploys filtered data to Synthea resources directory
- Runs Synthea simulation with command:
  ```bash
  java -jar synthea-with-dependencies.jar -p 100 -s 12345 "British Columbia" "Surrey"
  ```

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

### Synthea build fails
If the JAR build fails, try building manually:
```bash
cd synthea
./gradlew uberJar
```

### Cities not found in data
The notebook will show available cities if target cities aren't found. You can modify `TARGET_CITIES` based on the output.

## License

This project uses:
- Synthea: Apache License 2.0
- Synthea-International: Apache License 2.0

## Acknowledgments

- [SyntheaTM](https://synthetichealth.github.io/synthea/) by MITRE Corporation
- synthea-international repository contributors

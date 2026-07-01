# Analysis Workflow: Internship Arthur

⚠️ **IMPORTANT**: This notebook is designed for **analyzing pre-processed data only**. It requires data that has already been processed, synchronized, and aligned. It cannot be used directly with raw experimental data.

## Overview

This analysis pipeline performs statistical analysis and visualization on pre-processed behavioral data from visual motor experiments on mice, with comparative analysis. The workflow focuses on data analysis, statistical testing, and visualization of key behavioral metrics.

**This is NOT a raw data processing pipeline.** Input data must have already been:
- Processed and cleaned
- Synchronized across multiple data streams
- Aligned to behavioral events (halt events)
- Organized into the required directory structure

## Data Structure

The analysis works with two experimental conditions:
- **VF0 (0x VF)**
- **VF2 (2x VF)**

Each condition contains processed data from multiple mice and experimental sessions.

## Key Components

### 1. Data Loading & Path Configuration
- Automatically discovers and loads processed data from classified data folders
- Extracts and organizes session information (mouse ID, experimental day, timestamp)
- Maintains a cached dictionary of aligned data in pickle format for efficient reuse

### 2. Data Processing
- **Temporal alignment**: Aligns data relative to halt events (motion markers)
- **Binning**: Creates temporal bins around halt events (16s total duration, 2s windows)
- **Indexing**: Assigns halt indices and bin indices for organized analysis

### 3. Behavioral Metrics Analyzed
- **Velocity (0Y, 0X)**: Head velocity in horizontal and vertical directions
- **Acceleration (0Y)**: Rate of velocity change
- **Pupil Diameter**: Eye pupil size (indicator of arousal/attention)
- **Activity**: Overall motion activity and activity peaks
- **Motor Velocity**: Motorized treadmill/stimulus velocity
- **Halt Events**: Discrete stopping events in the behavioral sequence
- **Eye Position**: Ellipse center coordinates (eye tracking)

### 4. Statistical Analyses
The notebook implements multiple statistical methods:
- **Robust regression** (Huber Regressor): Estimates visual motor gain from baseline periods
- **Bayesian Mixed-Effects GLM**: Tests for significant differences with random effects
- **One-way ANOVA**: Compares metrics across conditions (before/after CNO)
- **Tukey's HSD**: Post-hoc pairwise comparisons
- **Kolmogorov-Smirnov test**: Distribution comparisons

### 5. Visualization Outputs
- **2-second time bins**: Pre/post-halt response profiles for all metrics
- **Small time bins**: Fine-grained analysis around halt events (±0.5-1s windows)
- **Directional change histogram**: Frequency of direction changes
- **Trial evolution plots**: How behavioral metrics change across trials
- **Trace plots**: Raw data traces for motor (stimulus) vs. no-motor conditions

## Prerequisites: Data Preparation Pipeline

**Before using this notebook, your data must be processed through the following stages:**

1. **Data Acquisition & Storage**
   - Raw experimental data collected from behavioral rigs
   - Multi-stream data (position tracking, pupil tracking, motor commands, etc.)

2. **Synchronization**
   - Align all data streams to a common temporal reference
   - Handle timing drift across different sensors/recording systems
   - Create synchronized CSV files

3. **Classification & Processing**
   - Classify behavioral events (e.g., halt/motion episodes)
   - Process eye tracking data (pupil diameter extraction, ellipse fitting)
   - Calculate derived metrics (velocity, acceleration)
   - Clean and downsample data as needed

4. **Alignment to Events**
   - Align data relative to key behavioral events (halt onsets)
   - Create time-centered windows around events
   - Generate the `*_downsampled_data_Apply halt_ 2s_aligned.csv` files
   - Organize into `*_processedData/aligned_data/` directory structure

**This notebook expects the output of this entire preprocessing pipeline.**

## How to Use

### Software Prerequisites
```
- Python 3.7+
- Jupyter/JupyterLab
- Mamba/Conda environment (if using mamba shell hook)
```

### Required Python Libraries
- pandas, numpy, matplotlib, scipy
- scikit-learn (HuberRegressor, AUC metrics)
- statsmodels (Bayesian GLM, formula API)
- tabulate (formatted table output)

### Setup Instructions

**Prerequisite**: Ensure all data has been processed, synchronized, and aligned following the pipeline described above.

1. **Configure Data Paths** (Cell 2):
   - Update `paths_VF0` and `paths_VF2` with your data directory locations
   - These should point to folders containing `*_processedData/aligned_data/` subdirectories

2. **Set Analysis Parameters** (Cell 4):
   - `mm_type`: Choose 'visual' or 'vestibular' stimulus type
   - `method_to_use`: Select 'delta' or 'ratio' for metric calculations
   - `list_mice_to_plot`: Specify which mouse sessions to analyze (by ID_year_month_day format)
   - `mice_z_560` / `mice_z_470`: Specify z-wavelength used for each mouse

3. **Configure Output Settings** (Cell 4):
   - `path_pickle`: Location to cache processed data
   - `path_save_fig`: Directory for saving figures
   - `format`: Output format (pdf, png, svg)

4. **Select Metrics to Analyze** (Cell 4):
   - `parameters_to_analyse_2s_bins`: Metrics for 2-second window analysis
   - `parameters_to_analyse_small_bins`: Metrics for fine-grained analysis
   - `metrics_MM_noMM`: Metrics for motor vs. no-motor comparisons

5. **Run the Analysis**:
   - Execute cells sequentially from top to bottom
   - The notebook will load/process data, run statistics, and generate figures

## Output Files

The notebook generates:
- **Pickle files**: Cached data dictionaries for fast reloading
- **Figure files**: Publication-quality plots in your chosen format
- **Console output**: Summary statistics and test results in tabular format

## Key Analysis Sections

| Cell | Purpose |
|------|---------|
| 1 | Import libraries and initialize environment |
| 2 | Configure data paths |
| 4 | Discover sessions and parameters |
| 4 | Set analysis parameters and output settings |
| 3 | Load/cache aligned data |
| 5+ | Statistical testing and visualization |

## Expected Data Format

Input CSV files should contain:
- **Time (s)**: Temporal offset from halt event
- **Halt Time**: Identifier for halt events
- **Motor_Velocity**: Stimulus/motor velocity
- **Velocity_0Y, Velocity_0X**: Head velocity components
- **Acceleration_0Y**: Head acceleration
- **Pupil.Diameter_eye1**: Pupil diameter
- **activity**: Behavioral activity measure
- **Ellipse.Center.X_eye1, Ellipse.Center.Y_eye1**: Eye position

## Important Notes

- ⚠️ **This notebook requires pre-processed, synchronized, and aligned data.** It will not work with raw experimental data.
- ⚠️ When including activity metrics, use "activity" or "activity_peak" in parameter lists, not other variations
- The notebook respects Z-wavelength information for proper metric normalization
- Baseline periods (Time < 0) are used for gain calculations and normalization
- The robust regression approach (Huber Regressor) minimizes influence of outliers
- Data must be organized in the expected directory structure: `*_processedData/aligned_data/` folders
- Input CSV files must contain all required columns (see "Expected Data Format" section)

---

*Last Updated: 02/07/2026*
*Author: Arthur*

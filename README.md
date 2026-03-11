# Myocardial Perfusion Quantification Tool
README – Myocardial Perfusion Quantification Notebook

### Overview

This contains a Python tool for quantifying myocardial blood flow (MBF) from dual-bolus cardiac perfusion MRI data.
It uses DICOM perfusion images, extracts signal curves from the left ventricle (AIF) and myocardium, applies dual-bolus scaling, and estimates MBF using several deconvolution methods.

Implemented methods: Fermi, Fermi with AIF delay, and Model-free deconvolution

### Data Requirements
The code uses two perfusion MRI series:
1. Main Perfusion Scan for the myocardial signal and saturated LV signal
2. Dilute Bolus Scan for the unsaturated LV signal
The code is set up such that both series are in the same DICOM directory.

The tool assumes:
- Images are motion corrected and normalized
- Dilute and saturated runs contain the same number of timepoints

#### Required Python Packages
Install the required packages:
`pip install numpy matplotlib scipy pydicom scikit-image scikit-learn osqp`

Additional libraries used in the notebook include `pandas` and `seaborn`

### General Workflow
The perfusion analysis follows these steps:
1. Load DICOM perfusion series
2. Select slice and crop images around the heart
4. Select ROIs for LV and myocardium
5. Extract signal curves
6. Align dilute and saturated AIF signals
7. Estimate dual-bolus scaling factor
8. Generate AIF
9. Run selected deconvolution model
10. Estimate MBF

### How to Run

#### Step 1 – Set DICOM folder
  Edit the path in the notebook:
  `dicom_dir = "PATH_TO_DICOM_FOLDER"`

#### Step 2 – Select Slice
- Choose which myocardial slice to analyze:
  - `slice_index = 0`  # basal
  - `slice_index = 1`  # mid
  - `slice_index = 2`  # apical
    
#### Step 3 – Crop the Images
- Manually adjust cropping coordinates around the heart:
  - `y_min, y_max = ...`
  - `x_min, x_max = ...`
- Separate cropping is used for:
  - main perfusion images
  - dilute AIF series
    
#### Step 4 – Select ROIs
The notebook will open interactive windows to select:
- LV center (main perfusion)
- Myocardium circular ROI
- LV center for dilute AIF

Controls for myocardium ROI:
- Left click – move center
- Scroll wheel – change radius
- Arrow keys – fine radius adjustment
- Close window when finished

LV saturated and dilute ROI radii default to 8 pixels but may need adjustment.

#### Step 5 – Dual Bolus Scaling
The pipeline automatically:
- shifts the dilute LV AIF to align with the saturated run
- identifies the first-pass window
- estimates the dual-bolus scaling factor

Alternatively, a fixed scale factor can be specified:
- `estimate_scale_factor = False`
- `scale_factor = 4.0`

Optional: Exclude saturated AIF points during scale factor estimation
- `exclude_saturation = True`
- `exclude_n = 3`
- Removes points around AIF peak and displays a plot with excluded region

#### Step 6 – Choose Quantification Method
Set the desired deconvolution method: `METHOD = "model_free"`
- Available options: "fermi", "fermi_delay", "model_free"

Optional fixed delay (Fermi delay model): `FIXED_DELAY = None`

### Outputs
The notebook produces:
- Signal Plots
  - LV AIF
  - myocardial signal curve
  - scaled dual-bolus AIF
- Deconvolution Results
  - tissue transfer function or R(t)
  - model fit vs measured myocardium signal
  - convolution relationship plots
- Quantitative Metrics
  - MBF Myocardial blood flow (mL/min/g)
  - R² Fit goodness

MBF Calculation is computed from the inital value of the TTF:
- `MBF = R(0) * 60 / 1.05`
  - R(0) = initial residue value
  - 1.05 g/mL = myocardial density
- Units: mL/min/g

### Notes
- ROI selection is currently manual
- Cropping coordinates may need adjustment for new datasets
- The model-free method enforces:
  - non-negative residue function
  - monotonic decay constraint

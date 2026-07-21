# BIRAFFE3_eyetracking_analysis

Python notebook for advanced analysis of Tobii eye-tracking data, including
**time synchronization with XDF/LSL markers, fixation-based drift estimation,
trial-wise drift correction**, and generation of **corrected gaze heatmaps**
over visual stimuli (e.g., faces).

# Eye-Tracking Heatmap Analysis

Python pipeline for analyzing **Tobii eye-tracking data** and generating **heatmaps of gaze fixations on visual stimuli (e.g. faces)**.

The script synchronizes Tobii gaze data with experimental markers from **LSL/XDF streams**, extracts stimulus presentation windows, and produces heatmaps overlaid on the original stimuli.

---

# Features

* Import Tobii gaze data (`.csv`)
* Load experiment markers from **XDF / Lab Streaming Layer**
* Synchronize timestamps between streams
* Parse gaze coordinates from both eyes
* Filter invalid gaze samples
* Detect gaze points outside the screen
* Extract stimulus presentation windows
* Generate:

  * heatmaps for each stimulus
  * heatmap for the whole session
* Overlay gaze heatmaps directly on stimulus images

---

# Example Output

For each stimulus the pipeline produces:

* fixation heatmap
* gaze points overlay
* saved `.png` images

Example structure:

```
results/
  heatmap_SUB2812_168_m_f_s_b.png
  heatmap_SUB2812_004_o_m_h_a.png
  heatmap_all_SUB2812.png
```

---

# Project Structure

```
project/
│
├── eyetracking_heatmap_analysis.ipynb
│
├── data/
│   └── 2812/
│       ├── SUB2812.csv
│       ├── SUB2812.xdf
│       └── SUB2812-Procedure_info.csv
│
├── stimuli/
│   ├── 168_m_f_s_b.jpg
│   ├── 004_o_m_h_a.jpg
│   └── ...
│
└── results/
    └── generated heatmaps
```

---

# Installation

Install required Python libraries:

```bash
pip install pandas numpy matplotlib seaborn pyxdf scipy pillow
```

Required packages:

* pandas
* numpy
* matplotlib
* seaborn
* pyxdf
* scipy
* pillow

---

# Input Data

The pipeline expects three files per participant:

### 1. Tobii gaze data

```
SUBXXXX.csv
```

Contains:

* gaze coordinates
* timestamps
* gaze validity

---

### 2. Experiment markers

```
SUBXXXX.xdf
```

Contains LSL streams such as:

* Events
* Procedure

The **Procedure stream** includes JSON markers describing stimulus presentation.

Example marker:

```json
{
  "STIM-TYPE": "285_f_m_a_w",
  "STIM-SET": "faces",
  "STIM-COND": "conditionA"
}
```

---

### 3. Participant metadata

```
SUBXXXX-Procedure_info.csv
```

Contains:

* ID
* age
* sex
* timestamp

---

# How It Works


Pipeline steps:

1. Load Tobii gaze data
2. Parse and average gaze coordinates (left/right eye)
3. Filter invalid gaze samples
4. Load XDF experiment streams (LSL)
5. Extract stimulus markers from the *Procedure* stream
6. Synchronize Tobii timestamps with experiment markers
7. Extract fixation periods from each trial
8. Estimate gaze drift relative to the screen center
9. Identify and exclude drift outliers
10. Apply trial-wise drift correction
11. Perform quality control using fixation heatmaps
12. Extract gaze data for each stimulus
13. Generate corrected gaze heatmaps:
    - per stimulus
    - for the whole session

---

# Heatmap Generation

Heatmaps are created using:

* `numpy.histogram2d`
* `scipy.ndimage.gaussian_filter`

The gaze coordinates are normalized (0–1) and then scaled to the pixel size of the stimulus image. 
All stimulus heatmaps are generated **after drift correction** to ensure
spatial consistency and comparability across trials.



---

# Drift Estimation and Correction

The pipeline includes a **fixation-based drift estimation and correction procedure**
to compensate for systematic gaze offsets over time.

## Fixation-based Drift Estimation

For each trial, gaze samples from the **first 3 seconds of stimulus presentation**
(when a fixation cross was displayed) are extracted.

For each fixation period:

1. A gaze heatmap is computed.
2. The maximum of the heatmap is identified.
3. A drift vector is defined as the offset between the heatmap maximum and
   the screen center **(0.5, 0.5)** in normalized coordinates.

## Outlier Detection

Trials with excessively large drift vectors (e.g. top 5% of drift magnitude)
are automatically identified as outliers and excluded from correction,
under the assumption that the participant did not fixate on the cross.

## Drift Correction

For all remaining trials, a **trial-specific drift vector** is applied to
all gaze samples belonging to that trial.

Gaze coordinates are corrected and clipped to the valid screen range (0–1)
before further analysis.

## Quality Control

To verify correction effectiveness, fixation heatmaps are generated:

- **before drift correction**
- **after drift correction**

showing the alignment of gaze around the fixation cross.


---

# Example Use Case

Typical applications include:

* face perception research
* emotion recognition studies
* gaze behavior analysis
* visual attention experiments

---

# Author

Created for experimental eye-tracking data analysis using **Tobii eye trackers** and **LSL/XDF pipelines**.

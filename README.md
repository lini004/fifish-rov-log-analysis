# FIFISH E-GO ROV Log Analysis

Python notebook to analyse the **ROV Log** files of a QYSEA **FIFISH E-GO** underwater drone (tethered ROV):
dive profiles, water temperature against depth, orientation/stability and a comparison of several dives.

Developed during an internship at **Eawag**, Department of Surface Waters (SURF), Remote Sensing group, to test
how an inexpensive underwater drone can complement remote sensing of lakes (e.g. as in-situ reference for satellite products).

## What the notebook does

For one selected log it produces:

| Output | Content |
|---|---|
| `1_dive_profile.png` | depth over time, coloured by phase (on shore / surface / diving), gaps and photo events marked |
| `2_temperature_time.png` | ROV temperature sensor over time, with depth |
| `3_temperature_profile.png` | temperature against depth (in water) |
| `4_attitude.png`, `5_heading_rose.png` | pitch, roll, heading |
| `6_speed_stability.png` | vertical speed and stability (a proxy for how steady the video is) |
| `7_time_per_depth.png` | time spent in each depth layer |
| `dive_table.csv`, `temperature_per_depth_bin.csv` | one row per dive; temperature statistics per metre of depth |
| `dive_summary.json`, `rov_log_clean.csv` | key numbers and the cleaned time series |

With `COMPARE_ALL = True` it also overlays all logs in `data/raw/` in `outputs/compare/`.

### Example (log of 20 October 2026, Lake Zurich)

![Dive profile](outputs/log_1020111815/1_dive_profile.png)
![Temperature profile](outputs/log_1020111815/3_temperature_profile.png)

## Data

The logs are exported from the FIFISH app:
**Help → ROV Log → switch on *Log Collection* → *Connect* and dive → *Export log* (format JSON)**
(FIFISH E-GO user manual V1.6, pp. 58-60). Turn the log collection on **before** the dive; the app logs about 2 samples per second.

A log is a list of records with `type`, `timestamp` (UTC, milliseconds) and `payload`:

| Record type | Fields | Note |
|---|---|---|
| `depth` | `depth` (m), `temperature` (°C) | temperature is measured inside the ROV |
| `attitude` | `yaw`, `pitch`, `roll` (°) | `yaw` = heading, 0° = north |
| `water_sensor` | `ph`, `salt`, `ntu`, `doValue` | all 0 unless the water-quality sensor accessory is attached |
| `photo` | `photo_type` | button press, useful to synchronise with the video |

The raw logs are **not included** in this repository (see `.gitignore`). Copy your own `ROV-Log*.json` files into `data/raw/`.

## Repository structure

```
fifish-rov-log-analysis/
├── README.md
├── .gitignore
├── environment.yml          # conda environment with all required packages
├── data/
│   └── raw/                 # put the ROV-Log*.json files here (not tracked)
├── notebooks/
│   └── fifish_log_analysis.ipynb
└── outputs/                 # figures and tables written by the notebook
    ├── log_<ID>/            # one folder per analysed log
    └── compare/             # comparison of all logs
```

## Setup

You need [conda](https://docs.anaconda.com/miniconda/) (or Miniconda) and, to run the notebook, VS Code with the
*Python* and *Jupyter* extensions (or JupyterLab).

```bash
git clone https://github.com/<your-username>/fifish-rov-log-analysis.git
cd fifish-rov-log-analysis
conda env create -f environment.yml
conda activate fifish-env
```

## How to run

1. Copy your log files into `data/raw/`.
2. Open `notebooks/fifish_log_analysis.ipynb` (VS Code: select the kernel **fifish-env**).
3. In section 1.2 set `LOG_ID` to a unique part of the file name, e.g. the number at the end (`"1020111815"`).
4. Run all cells (*Restart & Run All*). Results appear in `outputs/log_<ID>/`.

To analyse another log, change `LOG_ID` and run again from the top.

## Limitations

- The temperature sensor is inside the ROV, uncalibrated and slow to respond (minutes). Compare it with an independent
  reference (CTD, lake model, satellite surface temperature) before drawing conclusions.
- Without the optional U-QPS GPS module the logs contain **no coordinates**.
- Water entry is detected automatically (temperature drop or first dive); it can be set by hand with `WATER_ENTRY_S`.

## Author

Alina Bauer, Eawag SURF (Remote Sensing group). Questions and suggestions are welcome via GitHub issues.

# Data

This describes how the raw sensor recordings are pulled and processed by [notebook.ipynb](../notebook.ipynb).

## Source

Recordings are stored in a shared Google Drive folder and pulled locally with [gdown](https://github.com/wkentaro/gdown) — no manual download needed. Everything under `data/` is gitignored, so each collaborator fetches their own local copy by running the notebook.

## Pipeline

1. **Download** (`downloads/` → `data/downloads/`)
   The first code cell downloads the shared Drive folder into `downloads/road-checker-data/` via `gdown.download_folder`, moves its contents into `data/downloads/`, then removes the now-empty `downloads/` directory. Skip re-running this cell once the data is present locally.

2. **Unzip** (`data/downloads/*.zip` → `data/extracted/<recording>/`)
   Each `Raw*.zip` file in `data/downloads/` is one recording session. It's extracted into its own folder under `data/extracted/`, named after the zip file (e.g. `Raw012_Bumpy/`).

3. **Per-sensor CSVs**
   Each extracted recording folder contains one CSV per sensor stream, e.g.:
   - `Accelerometer.csv`
   - `AccelerometerUncalibrated.csv`
   - `Gravity.csv`
   - `Gyroscope.csv`
   - `GyroscopeUncalibrated.csv`
   - `TotalAcceleration.csv`, `Orientation.csv`, `Annotation.csv`, `Metadata.csv`

## Naming convention

Recording folders/zips follow `Raw<number>_<label>`, e.g. `Raw012_Bumpy`, `Raw003_Smooth`. `<label>` is the road-surface class for that recording (`Bumpy` or `Smooth`), so the folder name doubles as the ground-truth label. Some recordings are instead prefixed `HC-...` (e.g. `HC-bumpy-2min-...`, `HC-annotations-19min-...`) — these are ad-hoc recordings/annotation exports used to sanity-check the sensor setup rather than part of the numbered `Raw###` dataset.

## Sampling frequency check

The notebook estimates each sensor's sampling frequency per recording from its `seconds_elapsed` column (`1 / mean(diff(seconds_elapsed))`), then plots frequency vs. recording (one subplot per sensor) to spot recordings with inconsistent or unexpectedly low sampling rates before they're used for modeling.

## Local layout after running the notebook

```
data/
├── downloads/          # zipped raw recordings (gitignored)
│   ├── Raw001_Smooth.zip
│   ├── Raw012_Bumpy.zip
│   └── ...
└── extracted/           # unzipped, one folder per recording (gitignored)
    ├── Raw001_Smooth/
    │   ├── Accelerometer.csv
    │   ├── Gyroscope.csv
    │   └── ...
    └── ...
```

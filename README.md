# FIELD//ZERO data pipeline (v2, corrected)

**Read this first: this ZIP contains code, not data.** `raw/` and `processed/` are empty. No NASA/USGS server was reachable
from where this was built, so nothing was downloaded. Whether any dataset exists is decided only by the generated files
`reports/status_earth.md`, `status_mars.md`, `status_moon.md` (state `VALIDATED` = file on disk + all checks passed).

## Run on Windows
1. Install Python 3.10+ (python.org, tick "Add python.exe to PATH"). Create a free account at https://urs.earthdata.nasa.gov (needed for NASADEM only).
2. Unzip, open Command Prompt in this folder, run `run_all.bat`. It creates `.venv`, installs `requirements.txt`, then runs:
   unit tests -> synthetic selftest -> link check -> download -> process -> validate.
   The first NASADEM download asks for your Earthdata login in the console (never typed into any file by this project).
3. Open `reports\status_index.md`. If the Moon step says range requests failed: `python scripts\01_download.py --only moon --moon-full` (~8 GB).

## What changed vs. the previous ZIP
| # | Fix |
|---|---|
| 1 | Mars: old MOLA 463 m guess removed; `config.DEM_SOURCES` uses the Jezero CTX DEM 20 m URL from the inventory. Whole 87 MB file downloaded (resumable). |
| 2 | POWER: community no longer hard-coded. Each variable is tried with AG, RE, SB; the accepted community and API-reported units are recorded per variable. The parameter-list endpoint is queried as advice only (`reports/power_parameter_availability.json`); its URL pattern is unconfirmed. |
| 3 | LOLA: int16 x 0.5 = metres, applied exactly once when the raster is read (`checks.read_dem_metres`). The raw window file keeps the unscaled values. |
| 4 | NASADEM: earthaccess search/download -> unzip -> check .hgt is 25,934,402 B -> GeoTIFF per tile -> `rasterio.merge` crop to bbox. Manual route: put the zip in `raw/earth/nasadem_zip/` and re-run. |
| 5 | Coordinates: every raster is handled through its own CRS with pyproj; bbox -> native map units; output grid is a local transverse-Mercator in metres. Degrees are never mixed with metres. |
| 6 | Verification: size vs server Content-Length and vs log, sha256 vs log, TIFF header, CRS/transform, pixel size vs expected, bbox coverage, valid-pixel count/fraction, elevation plausibility, non-constant. |
| 7 | All three bodies are area-averaged onto 120 m and 360 m square-metre grids; kernels are defined in metres. |
| 8 | Status text is generated from files on disk; no file -> `NOT_PRESENT`. 02_process refuses any raw file that is not `VALIDATED`. |
| 9 | One status report per planet in `reports/`. |

## Limits you should know
- No official checksums were found for these files, so sha256 is a local integrity fingerprint, not proof against NASA's copy. The server's Content-Length is the external check.
- CTX DEM coverage vs the Mars bbox is unknown until the file is opened; the validator reports the covered fraction (warning if < 100 %).
- Area-averaging 20 m / 30 m / 118 m sources to 120 m smooths them by different amounts (Moon barely). The 360 m scale reduces this bias; compare the two scales.
- Not implemented: MEDA, Diviner, surface-characteristic layers (see `metadata/DATASET_INVENTORY.md`).
- Reanalysis (POWER) is modelled data, not station measurements.

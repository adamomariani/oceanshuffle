<a href="https://colab.research.google.com/github/adamomariani/oceanshuffle/blob/main/OceanShuffle.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

# OceanShuffle

Code for the MSc thesis *Analyzing Trade Discrepancy of Forest Products Across Countries*. OceanShuffle builds a `year x exporter x importer x HS4` panel of forest-product trade (68 headings, HS chapters 44–49, 1995–2024) from the [Harvard Growth Lab trade dataset, v5](https://doi.org/10.7910/DVN/5NGVOB). It then uses this panel to reproduce every figure and table of the thesis, with one function per output. All data fetching and reading is done through DuckDB, and plotting through Matplotlib.

## Dataset Locations

| File | Where | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;What&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; |
|---|---|---|
| `hs4_analysis_panel_ch44-49.parquet` (~180&nbsp;MB) | **Release [v1.3.10](https://github.com/adamomariani/oceanshuffle/releases/tag/v1.3.10)** → *Assets* | Panel used to produce all results |
| `chen2008_annex_volumes.csv` | Repository [root folder](https://github.com/adamomariani/oceanshuffle/blob/main/chen2008_annex_volumes.csv) | Only needed for Figure&nbsp;5.2 |
| HGL source CSVs (~13&nbsp;GB) | [Harvard Dataverse, doi:10.7910/DVN/5NGVOB](https://doi.org/10.7910/DVN/5NGVOB) | Only needed to rebuild the panel from scratch |

The <u>panel already contains all preprocessing</u> (HS6 → HS4 aggregation, filter for forest products, flags for reporting cases, and discrepancy measures `D`, `S`, `G`). **You do not need the raw HGL files** to reproduce the thesis.

## Quickest Run (Google Colab)

1. Download `hs4_analysis_panel_ch44-49.parquet` from release [v1.3.10](https://github.com/adamomariani/oceanshuffle/releases/tag/v1.3.10), and `chen2008_annex_volumes.csv` from [root folder](https://github.com/adamomariani/oceanshuffle/blob/main/chen2008_annex_volumes.csv) (optional, for Figure&nbsp;5.2)
2. Put both files in one folder on your Google Drive, e.g. `My Drive/oceanshuffle/`
3. Open the notebook in Colab: [OceanShuffle.ipynb](https://colab.research.google.com/github/adamomariani/oceanshuffle/blob/main/OceanShuffle.ipynb).
4. In *Setup → Environment Setup*, set `DB_DIR` to that folder:
   ```python
   DB_DIR = Path("/content/drive/My Drive/oceanshuffle")
   ```
5. *Runtime → Run all*. Allow Drive access when asked.

The notebook finds the panel and skips the database creation steps. It then writes every figure and table to `<DB_DIR>/Output/Thesis Results/Figures` and `.../Tables`. A full run takes about 3 minutes.

To produce a single output instead (a single figure or table), use the *RUN - One* cell and disable *RUN - All* (commenting or deleting the cell), e.g.:

```python
con = get_panel()
fig_4_2_annual_D(con, min_flow=0)           # Figure 4.2 for the full sample
display(tab_4_1_min_flow_sensitivity(con))  # Table 4.1
```

Table D.1 in the thesis (Appendix D) lists the functions and adjustable parameters for every figure and table. Documentation and text blocks help clarify specific functions for further customization.

## Running locally

Requires Python ≥ 3.10:

```bash
pip install duckdb pandas numpy scipy scikit-learn matplotlib networkx
```

Export [notebook](https://colab.research.google.com/github/adamomariani/oceanshuffle/blob/main/OceanShuffle.ipynb) as python file (e.g. `oceanshuffle.py`), remove the import statement `from google.colab import drive`, and its code counterpart `drive.mount(...)`. Then set `DB_DIR` to the local folder containing the panel (fastest run, \~3min), merged database (\~1hr), or source directory with split raw files (slowest run, \~2hrs).

```bash
python oceanshuffle.py
```

Used package versions: DuckDB&nbsp;1.3.2, pandas&nbsp;2.2.3, NumPy&nbsp;2.4.6, SciPy&nbsp;1.16.3, scikit-learn&nbsp;1.6.1.

## Rebuilding the panel from the raw data (optional)

Download all of the `country x country x product x year` split CSV files (HS92) from v5 of HGL on the [Harvard Dataverse](https://doi.org/10.7910/DVN/5NGVOB) into `<DB_DIR>/source/`, and make sure no panel file or pre-merged dataset is in `<DB_DIR>`. On *RUN - All*, the pipeline merges the CSVs into `full_db.parquet`, and builds the HS4 panel in `<PANEL_PATH>`. It then continues with the analysis. This needs substantially more memory and time than the quick run (\~2hrs).

## Configuration

The *LEGEND* block describes the expected folder structure and provides the paths to the main directories and files used. The main analysis configuration settings are in *Analysis Setup*:

| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Setting&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Default&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Meaning |
|---|---|---|
| `STUDY_YEARS` | `(1995, 2024)` | Study period start and end years |
| `MIN_FLOW` | `10_000` | Threshold used on the reconciled value F (nominal USD) for the analysis-sample |
| `SAVE_OUTPUTS` | `True` | (Over)write figures/tables to drive |
| `REAL_USD`, `PRICE_BASE_YEAR` | `True`, `2024` | Values in constant 2024 USD |

# CH-TAN ley 2024-25 flux product

Produced by Fabio Turco and Lukas Hörtnagl

Processing code and documentation for the PI dataset of the **cropland station CH-TAN (Tänikon, Switzerland; FLUXNET CH-Tnk)** during the **grass-legume ley, from 22 August 2024 to 4 June 2025**. The research station CH-TAN was part of [Swiss FluxNet](https://www.swissfluxnet.ethz.ch/), operated by the [Grassland Sciences Group, ETH Zurich](https://gl.ethz.ch/). Group leader: [Prof. Nina Buchmann](https://gl.ethz.ch/people/person-detail.nina.html).

[Site info CH-TAN](https://www.swissfluxnet.ethz.ch/index.php/sites/site-info-ch-tan/)

The dataset contains ecosystem fluxes measured by eddy covariance, meteorological data and detailed management information. It starts with the last tillage and the digestate application before sowing of the grass-legume ley (22 Aug 2024; sowing on 23 Aug) and covers establishment, mulching, winter, slurry application, spring growth, the first cut (13 May 2025) and the regrowth until 4 Jun 2025. It follows the [CH-TAN wheat 2023-24 flux product](https://github.com/fabioturc/dataset_ch-tan_wheat_2023-24_flux_product) (winter wheat, 17 Oct 2023 to 22 Aug 2024); 22 Aug 2024 is included in both products.

> **Data availability:** the raw input data and the final flux product will be archived on Zenodo,
> **DOI: [10.5281/zenodo.XXXXXXX](https://doi.org/10.5281/zenodo.XXXXXXX)** *(to be added after upload)*.
> This repository contains the code only; see [Reproducing the dataset](#reproducing-the-dataset).

---

## Site and dataset

| | |
|---|---|
| Station | CH-TAN, Tänikon (TG), Switzerland, 47.480620 °N, 8.911868 °E |
| Crop | Grass-legume ley (UFA 323, UFA Samen), sown 23 Aug 2024 after tillage and digestate application (22 Aug 2024); mulched 1 Nov 2024, slurry 6 Mar 2025, first cut 13 May 2025; field managed by Swiss Future Farm |
| Design | Field split into two parcels: **A (GL)** grass-legume mixture, **B (GL+PL)** (towards the houses) the same mixture with 20% plantain (*Plantago lanceolata* cv. Ecotain, PL) in the seed mix; same management and N inputs in both parcels. Fluxes are attributed to a parcel by wind direction (dividing line 84.6° / 264.6°) |
| Period | 22 Aug 2024 00:00 – 5 Jun 2025 00:00 (full days 22 Aug 2024 to 4 Jun 2025) |
| Time zone | CET (UTC+1), no daylight saving time |
| Time resolution | 30 min |

The period is defined once in [`src/config.py`](src/config.py) and applied when the raw data are read in the first notebooks (steps 12, 21, 31, 32 and the canopy data in 8x.1). The two R scripts contain the same dates written out.

### Variables

Eddy covariance fluxes:

- **NEE**: net ecosystem exchange of CO₂, partitioned into **GPP** and **RECO** (nighttime and daytime methods, REddyProc)
- **LE**: latent heat flux
- **H**: sensible heat flux
- **FN2O**: nitrous oxide flux
- **FCH4**: methane flux

Plus gap-filled meteorological variables, soil temperature and water content, canopy height and crop N, and a half-hourly management time series (fertilisation, tillage, sowing, mulching, mowing). In the management time series, `crop` is 2 for bare soil (22 Aug 2024) and 3 for the ley (from 23 Aug 2024); the codes continue those of the wheat product (1 = winter wheat, 2 = bare soil). Both parcels received the same fertilization, so, unlike in the wheat product, the fertilization variables (`fert`, `fert_min`, `fert_org`, `nitrogen_fert` and the derived `n_decay_*`, `timesince_*`, `*_timed`) are not split by parcel; notebook 21.0 stops with an error if the management file contains a parcel-specific fertilization in the period.

### Processing levels and naming

| Level / tag | Meaning |
|---|---|
| `L1` | EddyPro output (FLUXNET format), IRGA (CO₂/H₂O) and LGR (N₂O/CH₄) runs merged |
| `L3.1`, `L3.2` | Quality control with the [diive](https://github.com/holukas/diive) flux processing chain (`QCF`: quality-controlled flux incl. flag 1; `QCF0`: highest quality only) |
| `L3.3_CUT_16/50/84` | USTAR-filtered with the 16th/50th/84th percentile of the seasonal u* thresholds (REddyProc) |
| `footprint`, `parcelA`, `parcelB` | Flux of the whole footprint, or attributed to parcel A or B |
| `gfXGBoost`, `gfXG` | Gap-filled with XGBoost; `_ISFILLED` flags filled records; `_sigmaGF` gap-filling uncertainty; `_ens..` cross-validation ensemble members |
| `GPP_NT`, `RECO_NT`, `GPP_DT`, `RECO_DT` | Partitioned fluxes (nighttime / daytime method) |

---

## Repository structure

```
dataset_ch-tan_ley_2024-25_flux_product/
├── data/                          # raw input data from Zenodo (not tracked by git)
│   ├── FLUXES/IRGA/               # EddyPro FLUXNET output, CO2/H2O analyser
│   ├── FLUXES/LGR/                # EddyPro FLUXNET output, N2O/CH4 analyser
│   ├── METEO/                     # on-site meteo, gaps filled with MeteoSwiss station Tänikon (TAE) (output of step 12)
│   ├── MANAGEMENT/                # management log per parcel
│   └── CANOPY/                    # half-hourly canopy variables
├── notebooks/                     # processing steps, numbered in run order
│   ├── 10_METEO/                  # meteo gap-filling
│   ├── 20_MANAGEMENT/             # management log -> time series
│   ├── 30_MERGE_DATA/             # read and merge EddyPro L1 outputs
│   ├── 40_FLUX_PROCESSING_CHAIN/  # diive quality control (L3.1-L3.2) per flux
│   ├── 50_REDDYPROC/              # u* thresholds, MDS gap-filling, partitioning (R)
│   ├── 60_USTAR_FILTERING/        # u* filtering (L3.3)
│   ├── 70_SPLIT_TREATMENTS/       # attribution of fluxes to parcels A/B
│   ├── 80_GAP-FILLING/            # XGBoost gap-filling (NEE, FN2O, FCH4), NEE partitioning
│   └── 90_FINAL_MERGE/            # final product: fluxes + meteo + management
├── src/
│   ├── config.py                  # dataset period, site info, data paths
│   └── gapfilling_utils.py        # gap-filling and uncertainty functions
├── run_pipeline.py                # runs all steps in order (see below)
├── environment.yml                # Python environment
├── dataset_ch-tan_ley_2024-25_flux_product.Rproj
├── CITATION.cff
└── LICENSE
```

Each notebook writes its outputs next to itself, with the same number (e.g. `31.0_… → 31.1_…`); later notebooks read them via relative paths. Outputs are not tracked by git.

### Processing steps

| Step | Notebook / script | Input | Output |
|---|---|---|---|
| 11 | `10_METEO/11.0_DownloadMeteoScreenedVariables` *(internal, see note)* | Swiss FluxNet database | `11.1_CH-TAN_meteo_meteoscreening_diive.csv` |
| 12 | `10_METEO/12.0_GapFillingMeteoswiss` *(internal, see note)* | 11.1, `10_METEO/meteoswiss/` (MeteoSwiss station TAE) | `data/METEO/CH-TAN_meteo_gapfilled-meteoswiss_2024-25.csv` |
| 13 | `10_METEO/13.0_GapFillingMeteoXGBoost` | `data/METEO/` | `13.1_CH-TAN_meteo_gapfilled.parquet` |
| 21 | `20_MANAGEMENT/21.0_ConvertMgmtToTimeseries` | `data/MANAGEMENT/` | `21.1_mgmt_full_timestamp.parquet` |
| 31–33 | `30_MERGE_DATA/31.0`, `32.0`, `33.0` | `data/FLUXES/` | `33.1_CH-TAN_IRGA+LGR_Level-1_eddypro_fluxnet.parquet` |
| 41–45 | `40_FLUX_PROCESSING_CHAIN/4x.0_FluxProcessingChain_L3.2_*` (NEE, LE, H, FN2O, FCH4) | 33.1 | `4x.1_FluxProcessingChain_L3.2_*.parquet` |
| 46 | `40_FLUX_PROCESSING_CHAIN/46.0_MergeFluxProcessingChainResults` | 41.1–45.1, 13.1, 33.1 | `46.1_….parquet`, `46.2_…subset-forREddyProc.csv` |
| 51 | `50_REDDYPROC/51.0_UstarDetection_MDS_Gapfilling_NEE_Partitioning.Rmd` (R) | 46.2 | `51.1_UstarThresholds_MDS-gapfilled_NEE-partitioned.csv` |
| 61–62 | `60_USTAR_FILTERING/61.0`, `62.0` | 46.1, 51.1 | `62.1_FLUXES_L3.3_….parquet` |
| 71 | `70_SPLIT_TREATMENTS/71.0_AttributionTreatments` | 62.1 | `71.1_…PARCELS.parquet` |
| 81.1–81.4 | `80_GAP-FILLING/81_NEE/`: prepare inputs → feature selection → hyperparameter optimisation → gap-filling | 71.1, 13.1, 21.1, `data/CANOPY/` | `81.4.1_NEE_GF-XGBoost.parquet`, `81.4.2_PartitioningSubsetForREddyProc.csv` |
| 81.5 | `80_GAP-FILLING/81_NEE/81.5.0_PartitioningREddyProc.R` (R, **run from the repository root**) | 81.4.2 | `81.5.1_NEE_XG-GAPF_PART_ReddyProc.csv` |
| 81.6 | `80_GAP-FILLING/81_NEE/81.6.0_CollectGapFillingPartitioningResults` | 81.4.1, 81.5.1, 51.1 | `81.6.1_NEE_GF-XGBoost_GPP_RECO.parquet` |
| 82, 83 | `80_GAP-FILLING/82_FN2O/`, `83_FCH4/` (same four steps) | 71.1, 81.6.1, 13.1, 21.1, `data/CANOPY/` | `82.4.1_FN2O_GF-XGBoost.parquet`, `83.4.1_FCH4_GF-XGBoost.parquet` |
| 84 | `80_GAP-FILLING/84.0_MergeGapFillingResults` | 81.6.1, 82.4.1, 83.4.1 | `84.1_GF-FLUXES+PARTITIONED.parquet` |
| 90 | `90_FINAL_MERGE/90.0_MergeFluxesMeteoManagement` | 71.1, 84.1, 13.1, 21.1 | **`90.1_…_FULL.parquet`**, **`90.2_…_CORE.parquet/.csv`** (see [Final product](#final-product)) |

The u\* thresholds (step 51) are estimated for four seasons defined by the management: start of the dataset (tillage, digestate, sowing), day after mulching (2 Nov 2024), slurry application and start of spring growth (6 Mar 2025), day after the first cut (14 May 2025).

Feature selection (8x.2) and hyperparameter optimisation (8x.3) write `best_features_*.txt`, `ranked_features_*.txt` and `best_hyperparameters_*.json` next to the notebooks; these files are tracked by git. Once they exist, the slow steps 8x.2 and 8x.3 can be skipped (`python run_pipeline.py --skip-tuning`).

### Final product

| File | Content |
|---|---|
| `90.1_CH-TAN_ley_2024-25_FLUXES_METEO_MGMT_FULL.parquet` | All variables of all processing steps (> 2000 columns) |
| `90.2_CH-TAN_ley_2024-25_FLUXES_METEO_MGMT_CORE.parquet/.csv` | Core variables most users need (fluxes, flags, uncertainties, meteo, soil, management); see notebook 90.0 |

All files are half-hourly with the time zone naive index `TIMESTAMP_MIDDLE` (CET, UTC+1).

---

## Reproducing the dataset

1. **Get the data:** download the Zenodo archive and place the `data/` folder in the repository root.
2. **Python environment** (Python 3.11, [diive](https://github.com/holukas/diive) v0.89.0):
   ```bash
   conda env create -f environment.yml
   conda activate ch-tan-flux
   ```
3. **R environment** (steps 51 and 81.5): R ≥ 4.3 with `tidyverse`, `REddyProc`, `caTools`, `data.table`, `viridis`, `mlegp` (and `rmarkdown`/`knitr` for step 51). In RStudio, open the `.Rproj` file so that the working directory is the repository root (needed for 81.5; the `.Rmd` is knitted in its own folder).
4. **Run the steps in numerical order**, either all at once with the runner (next section) or by opening the notebooks one by one with their own folder as working directory (default in Jupyter and VS Code).

Steps 11 and 12 need access to the internal Swiss FluxNet database and are included for documentation; their result, the on-site meteo gap-filled with MeteoSwiss data (`data/METEO/CH-TAN_meteo_gapfilled-meteoswiss_2024-25.csv`), is part of the raw data, so reproduction starts at step 13. XGBoost results can differ slightly between hardware and library versions.

### Running all steps

[`run_pipeline.py`](run_pipeline.py) executes the notebooks and the two R scripts in order, each notebook in a fresh kernel with its own folder as working directory. Executed notebooks are saved in place with their outputs, so plots and prints can be checked afterwards. The run stops at the first error and the log is written to `run_pipeline.log`.

```bash
conda activate ch-tan-flux
python run_pipeline.py                  # all steps
python run_pipeline.py --list           # list the steps
python run_pipeline.py --from 41        # from step 41 to the end
python run_pipeline.py --from 81 --to 84
python run_pipeline.py --only 71 90     # selected steps only
python run_pipeline.py --skip-tuning    # skip 8x.2 and 8x.3 (uses the saved features and hyperparameters)
```

Rscript is searched on the PATH, in `Program Files\R` and in `AppData\Local\Programs\R` (per-user installation); otherwise pass it with `--rscript "<path>/bin/Rscript.exe"`. Step 12 is skipped automatically when its internal input (`11.1_…_diive.csv`) is not available.

A single notebook can also be run without opening it: `jupyter nbconvert --to notebook --execute --inplace <notebook>.ipynb`.

---

## Input data

| File(s) | Content | Source |
|---|---|---|
| `data/FLUXES/IRGA/*.csv`, `data/FLUXES/LGR/*.csv` | EddyPro (FLUXNET output) half-hourly fluxes | Swiss FluxNet, ETH Zurich |
| `data/METEO/CH-TAN_meteo_gapfilled-meteoswiss_2024-25.csv` | Screened on-site meteo and soil data (30 min, TIMESTAMP_END), gaps in the meteo variables filled with data of MeteoSwiss station Tänikon (TAE) (`*_is_meteoswiss` flags) | Swiss FluxNet database; [MeteoSwiss Open Government Data](https://opendatadocs.meteoswiss.ch/) – source: MeteoSwiss |
| `data/MANAGEMENT/CH-TAN_management_parcels.csv` | Management operations per parcel with N inputs | Swiss Future Farm, ETH Zurich |
| `data/CANOPY/02-*_model.csv` | LAI, canopy height, above-ground biomass, crop N (mean and per parcel), 30 min; biomass and crop N end with the first cut (13 May 2025) | *TODO: describe origin (field measurements, interpolation)* |

The raw files cover the whole station record (Oct 2023 – Jun 2025); the notebooks select the dataset period when reading them.

---

## License

- **Code** (notebooks, scripts, `src/`): [MIT License](LICENSE)
- **Data** (Zenodo archive): [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). MeteoSwiss data are redistributed under their open data terms (source: MeteoSwiss).

## Citation

If you use the data or the code, please cite the dataset (see also [CITATION.cff](CITATION.cff)):

> Turco, F. and Hörtnagl, L. (2026). *CH-TAN ley 2024-25 flux product: eddy covariance fluxes (CO2, H2O, H, LE, N2O, CH4), meteo and management of a grass-legume ley in Tänikon, Switzerland (August 2024 - June 2025)* [Data set]. Zenodo. https://doi.org/10.5281/zenodo.XXXXXXX *(DOI to be added)*

## Acknowledgments

We acknowledge the scientific advice by Lukas Hörtnagl, Iris Feigenwinter, and Nina Buchmann. The technical assistance in the maintenance of the eddy station by Thomas Baur, Philip Meier, Martin Rüegg and Peter Ravelhofer is greatly acknowledged. We thank Swiss Future Farm and particularly Ueli Schild and Florian Bachmann, for managing the field where the flux station was located.

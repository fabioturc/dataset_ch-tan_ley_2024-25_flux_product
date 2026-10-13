# Changelog

## v0.9.0 (2026-10-13)

First release. Code version that produced flux product version **FP2026.10** (`dataset_CH-TAN_Aug2024-Jun2025_FP2026.10`), published in the ETH Research Collection record [10.3929/ethz-c-000807457](https://doi.org/10.3929/ethz-c-000807457) together with the [wheat 2023-24 product](https://github.com/fabioturc/dataset_ch-tan_wheat_2023-24_flux_product) and the raw input data. Pre-publication release: v1.0.0 will follow with the related article.

Same processing as the wheat 2023-24 product (v0.9.0, including its fixes: VPD in hPa for REddyProc, `trt` = 0.5 for uncertain parcel attribution, fixed random seeds in the R steps), adapted to the ley:

- Dataset period 22 Aug 2024 – 4 Jun 2025; seasons of the u* thresholds from the ley management (establishment, winter, spring growth, regrowth after the first cut).
- Fertilisation variables common to both parcels (same management); notebook 21.0 stops if the management file contains a parcel-specific fertilisation.
- Notebook 13.0 gap-fills the soil variables with XGBoost only where gaps exist.
- Parcels labelled GL (grass-legume mixture) and GL+PL (with plantain).
- Notebooks are committed without outputs (nbstripout, see `.gitattributes`).

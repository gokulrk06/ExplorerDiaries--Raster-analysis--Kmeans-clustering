# Python for GIS: Raster Processing & Unsupervised Land Cover Classification

An end-to-end workflow for pulling satellite imagery directly from Google Earth Engine into Python, building composites, and running unsupervised classification to identify land cover types — no manual downloads, no desktop GIS software required.

## What this notebook does

1. **Data acquisition (via `xee` + Google Earth Engine)**
   - Loads Sentinel-2 Surface Reflectance (Harmonized, Level-2A) and Landsat 8 Level-2 Collection 2 imagery directly into `xarray`, filtered by a bounding box, date range, and cloud cover threshold
   - Builds a yearly median composite to minimize cloud contamination
   - Uses Google's `xee` bridge, so imagery streams straight from Earth Engine into an `xarray.Dataset` — well suited for small-to-medium AOIs (Dask is recommended for larger areas)

2. **Visualization**
   - Per-band plots for all Sentinel-2 (B2, B3, B4, B8, B11, B12) and Landsat 8 bands
   - **True Color Composite (TCC)**: B4/B3/B2
   - **False Color Composite (FCC)**: B8/B4/B3, for vegetation emphasis
   - **NDVI** (Normalized Difference Vegetation Index) computed from B8/B4

3. **Unsupervised classification (K-Means)**
   - Stacks all bands into a single `(pixels, bands)` array
   - Standardizes features (`StandardScaler`) to account for very different value ranges across visible vs. SWIR bands
   - Determines optimal cluster count `k` using the elbow method (inertia) and silhouette score
   - Runs K-Means and reshapes labels back into the original raster grid
   - Renders a discrete, labeled cluster map with a side-by-side TCC comparison

## Key finding

Testing `k = 2` to `9` on standardized Sentinel-2 band values showed inertia flattening from `k=5` onward, while the silhouette score peaked at **k=3** (~0.47) — both metrics converge on 3 as the most cohesive, well-separated cluster count for this AOI.

## Requirements

```
earthengine-api
xee
xarray
rioxarray
geopandas
numpy
pandas
matplotlib
seaborn
scikit-learn
shapely
```

Install with:

```bash
pip install earthengine-api xee xarray rioxarray geopandas numpy pandas matplotlib seaborn scikit-learn shapely
```

You'll also need a Google Earth Engine account with API access enabled — the notebook authenticates via `ee.Authenticate()` on first run.

## Usage

1. Set your area of interest as a bounding box:
   ```python
   bbox1 = [min_lon, min_lat, max_lon, max_lat]
   ```
2. Adjust the date range and cloud filter threshold as needed
3. Run cells sequentially — imagery pulls from Earth Engine, composites render, and clustering runs on the stacked band array
4. Tune `k` based on the elbow/silhouette diagnostics before generating the final cluster map

## Roadmap / possible extensions

- Interactive polygon drawing (`ipyleaflet`) for collecting labeled training samples toward a **supervised** classification (Random Forest, SVM)
- Vectorizing classified rasters into polygons (`rasterio.features.shapes`) for downstream GIS analysis
- Alternative clustering methods (Agglomerative, DBSCAN, ISODATA) for comparison against K-Means
- Scaling to larger AOIs with Dask-backed `xarray`

## Notes

- Median composites help reduce cloud/shadow noise but can still leave gaps in persistently cloudy regions — inspect band plots before trusting downstream results
- NaN/masked edge pixels are excluded from clustering and re-inserted as NaN in the final label grid
- Cluster IDs are arbitrary and unlabeled by K-Means — visual inspection against the TCC/FCC is needed to assign meaningful land cover names (water, vegetation, urban, bare soil, etc.)

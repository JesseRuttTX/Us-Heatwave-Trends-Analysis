# Reassessing U.S. Heatwave Trends with Bias-Corrected Observations

Analysis code for a study of how homogenization and station sampling affect apparent
U.S. heatwave trends, comparing the 1930s Dust Bowl with the present day.

The central question is how much of the difference between the 1930s and today comes
from the climate, and how much from two choices made in building a temperature record:
whether station data are bias-corrected, and which stations are in the network.

## What is here

| File | Contents |
| --- | --- |
| `US-Heatwave-Trends-Analysis-Code.ipynb` | Every figure in the paper. Shared code first, then one section per figure. Saved with its outputs, so the figures are visible without running anything. |
| `requirements.txt` | Python packages. |
| `.gitignore` | Keeps the data, the caches and the generated figure files out of the repository. |

### How the notebook is laid out

Sections 1-4 build what the figures share:

| Section | Contents |
| --- | --- |
| 1. Setup | Imports, data locations, caches, and the settings the figures share. |
| 2. Geometry | One CONUS polygon: fractional area weights for the gridded averages, a point-in-polygon test for the station figures. |
| 3. Anomaly-first pipeline | Station anomalies, IDW gridding, cell anomalies, area averaging, day-weighted seasonal means. |
| 4. Gridded products | NOAAGlobalTemp and CRUTEM5, read once and cached. |

Sections 5-10 are the figures:

| Section | Figure | What it shows |
| --- | --- | --- |
| 5 | Figure 1 | CONUS JJA anomalies, TMAX / TMIN / TAVG, eight datasets, 1900-2024 |
| 6 | Figure 2 | Seasonal bars, DJF-SON by element, Dust Bowl vs. modern |
| 7 | Figure 3 | CONUS daily TMAX record frequency, May-September |
| 8 | Figure 4 | Berkeley Earth exceedance maps, p95 from the full record |
| 9 | Figure 5 | Northern mid-latitude band, 24-50N, JJA |
| 10 | Figure 6 | Heat-wave days, CONUS vs. the global strip, Christy (2026) method |

Each figure section is split into *settings and helpers*, *build*, *check* and *plot*, so
the numbers can be read before anything is drawn.

### The CHECK cells

Every section ends with a `CHECK` cell that tests what was just defined, so a failure is
located before any figure is drawn. Most run on synthetic numbers and need no data at
all. They confirm, among other things, that a season is labelled by the year of its last
month; that a field equal to its own climatology gives exactly zero; that a station
sitting on a grid point is reproduced exactly by the interpolation; that a station
without enough baseline years is dropped; that ties in the record count are split rather
than awarded to the earliest year; that each record series integrates to exactly 153
May-September days; and that a five-day hot spell is not counted as a heat wave while a
six-day one is.

## What is NOT here

No data and no generated figure files. The raw archives run to several GB and the caches
to more, so they are excluded by `.gitignore`. The notebook expects them at

```
/Volumes/Expansion/homogenized_dataset_analysis/    # raw archives (external drive)
    GHCND/Global/ghcn_global_YYYY.parquet           # GHCN-Daily station archive
    GHCND/ghcnd-inventory.txt
    Berkeley_Earth/Processed/                       # Berkeley Earth daily fields
    Berkeley_Earth/RAW/
    ERA5/era5_2t_YYYYMM_daily.nc
    nclimdiv/                                       # NOAA statewide/divisional files
    noaaglobaltemp/, crutem5/
    ushcn_daily_homog/output/                       # USHCN raw and adjusted daily pairs
<working directory>/_cache_*/                       # caches the notebook builds itself
```

Change the paths in the Setup cell to point at your own copy; the CHECK cell right below
it reports which of them this machine can see. Caches are rebuilt automatically on a
first run; that run is slow (minutes per dataset), later runs take seconds. Figures are
written to `PAPER_FIGURES_FINAL/`.

### Data sources

| Dataset | Source |
| --- | --- |
| GHCN-Daily | NOAA NCEI |
| USHCN v2.5 (raw and bias-corrected) | NOAA NCEI |
| nCLIMDIV | NOAA NCEI, statewide file, CONUS region code 110 |
| Berkeley Earth | Berkeley Earth daily products |
| NOAAGlobalTemp v6, CRUTEM5 | NOAA NCEI / Met Office Hadley Centre and CRU |
| ERA5 | ECMWF, via the Copernicus Climate Data Store |

## Methods, in one paragraph each

**Anomalies come first.** Each station is converted to an anomaly against its own
1951-1980 monthly climatology before it is gridded, and each grid cell against its own
before cells are averaged. Averaging absolute temperatures first lets a change in which
stations report move the series, because stations differ in elevation and exposure.

**One CONUS domain.** A Natural Earth 50m polygon of the United States, clipped to
24-50N / 125-66W and minus lakes, tests both stations and grid cells. Each cell is
weighted by its spherical band area times the fraction of the cell inside the polygon.

**Station products are gridded by IDW** onto a 0.5 degree CONUS grid (power 2, 8
neighbours, 150 km cutoff), except where a figure counts at stations on purpose.

**Records** (Figure 3) are counted per calendar day, May-September: for each station or
cell, the year holding the highest TMAX of 1900-2024 gets that day, and years sharing the
highest value split it equally. A location must have data in at least 100 of 125 years
and 80% of possible days.

**Heat waves** (Figure 6) follow Christy (2026): runs of six or more days above a
day-of-season 90th percentile computed over the full record in a +/-3 day window. A
station or cell is dropped from a year's average if fewer than 70% of that year's
May-September days have a valid TMAX.

## Running it

```bash
pip install -r requirements.txt
jupyter lab US-Heatwave-Trends-Analysis-Code.ipynb
```

Run sections 1-4 once, then the figure you want; within a figure, run its cells in order.
Run section 7 (Figure 3) before section 10 (Figure 6): it builds the daily station cube
that the heat-wave figure reads.

Cartopy and geopandas are the awkward dependencies. `conda install -c conda-forge
cartopy geopandas` is usually easier than pip.

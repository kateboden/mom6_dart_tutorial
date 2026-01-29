# Ensemble Averaging NetCDF Files

This document explains how to **concatenate multiple NetCDF files representing ensemble members** and **compute ensemble means** in two common ways:

1. Using **NCO** (NetCDF Operators) 
2. Using **Python with Xarray**  

---

## 1. Using NCO

**Workflow:**

1. Concatenate all ensemble members along a new dimension (`record`) using `ncecat`     
2. Compute the ensemble mean along the `ensemble` dimension with `ncwa`  
3. Remove intermediate files to save space  

**Example: single day (day 001):**

```bash
# Load NCO module
ml nco
cd <rundir>
# Concatenate all 40 ensemble members for day 001
ncecat *mom6.h.z._*.0054-001.nc ens_all_001.nc

# Take the ensemble mean across the 'ensemble' dimension
ncwa -a ensemble ens_vars_001.nc ens_mean_001.nc

# Remove intermediate files to save space
rm -f ens_all_001.nc ens_vars_001.nc
```
---

## 2. Using Xarray in Python

**Example: loop through multiple days**
```python
import xarray as xr
import glob
import re
import numpy as np
from pathlib import Path

# Path to ensemble files
casename = f'g.e30_a07g.G_JRA.TL319_t232_zstar_N75.2025.SKEB_DA.sr.40_3'
fpath = f'/glade/derecho/scratch/kboden/{casename}/run/'
OUTDIR = Path(fpath + "ensmean_z")
OUTDIR.mkdir(exist_ok=True)

for day in range(1, 2):  # loop over days
    day_str = f"{day:03d}"
    print(day_str)

    # Get all ensemble files for this day
    files = glob.glob(f"{fpath}g.e30_a07g.*.mom6.h.z.*-{day_str}.nc")
    if len(files) == 0:
        print(f"⚠️  No files for day {day_str}")
        continue

    dsets = []
    for f in files:
        # Extract ensemble member number from filename
        m = re.search(r'_(\d{4})\.', f)
        ens = int(m.group(1))

        # Open dataset lazily with chunks
        ds = xr.open_dataset(f, engine="netcdf4", chunks={"time": 1})

        # Assign ensemble coordinate and expand dimension
        ds = ds.assign_coords(ens=ens).expand_dims("ens")
        dsets.append(ds)

    # Concatenate along ensemble dimension and take the mean
    ds_all = xr.concat(dsets, dim="ens")
    ds_mean = ds_all.mean("ens", skipna=True)

    # Write output
    out = OUTDIR / f"mom6_ensmean_day_{day_str}.nc"
    ds_mean.to_netcdf(out)

    # Close datasets to free memory
    ds_all.close()
    ds_mean.close()

    print(f"✅ Wrote {out}")
 ```

Notes:

- `.assign_coords(ens=ens).expand_dims("ens")` adds an ensemble dimension with the correct label
- `xr.concat(dsets, dim="ens")` stacks all ensemble members along the new dimension
- `.mean("ens")` computes the ensemble average (equally weighted)
-  `chunks={"time":1}` allows lazy loading for large datasets (compatible with Dask)

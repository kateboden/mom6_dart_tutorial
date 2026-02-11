---
layout: default
title: Multi-Instance MOM6 with DART
---


# Setting Up a Multi-Instance CESM Case with DART

This page describes step-by-step how to set up and run a multi-instance MOM6 case with data assimilation (DA) using DART. Note that CESM is the main controller here, through CIME you will set up a case, in the caseroot you will modify the appropriate xml variables to enable DA, the executable 'filter' from DART will live in the bld directory of the CESM case. 

---

## 1️⃣ Create a New Case

As for any CESM case the tool that generates a new case is create_newcase. This tool is located in the $SRCROOT directory under the cime/scripts directory. For a multi-instance case you need to use the -ninst and --multi-driver tags.

```bash
create_newcase --case <casename> --res <resolution> --compset <compset> --run-unsupported --output-root <output_dir> --ninst <ens_size> --multi-driver --project=<project_#>
```
This will create a new multi-instance CESM case. For my experiments I used a G-case (ocean/sea ice) with JRA forcing for the atmosphere.

## 2️⃣ Initialize the Ensemble

In the run directory, you must provide a restart file for each instance (ensemble member). For my 40-member ensemble, I used January 1 restart files from a previous spin-up simulation. I selected the 40 most recent January 1 restarts (1971–2010), assigned one year to each ensemble member, and reset the model dates in those restart files to January 1, 2011 so that all ensemble members begin from different physical states but share the same simulation start date.

Below is an example bash for loop illustrating how I constructed the ensemble from historical January 1 restart files. For each selected year:
- Restart files are copied into the run directory.
- Files are renamed to follow CESM’s multi-instance naming convention (e.g., _0001, _0002, …).
- Component-specific rpointer files are created for each ensemble member.
- Time metadata is modified so that all ensemble members begin on the same simulation start date.
  
```bash
for year in $(seq -f "%04g" ${start_year} 53); do
    echo "Working on ensemble member ${ens}"
    year_dir=${year}-01-01-00000
    # Copy all the files over
    cp ${rest_dir}/${year_dir}/g.e30* ${run_dir}
    echo "Copied files from ${rest_dir}/${year_dir}"
    # Rename the files
    mv ${type}.cice.r* ${casename}.cice_${ens}.r.${start_date}.nc  # cice
    mv ${type}.cpl.r* ${casename}.cpl_${ens}.r.${start_date}.nc    # cpl
    mv ${type}.datm.r* ${casename}.datm_${ens}.r.${start_date}.nc  # atm
    mv ${type}.drof.r* ${casename}.drof_${ens}.r.${start_date}.nc  # rof
    mv ${type}.mom6.r* ${casename}.mom6_${ens}.r.${start_date}.nc # mom6  
    #mv ${type}.mom6.r* ${casename}.mom6_${ens}.r.${start_date}.nc # mom6

    # Create rpointer files for each component
    echo "${casename}.drof_${ens}.r.${start_date}.nc" > rpointer.rof_${ens}.${start_date}        # rof
    echo "${casename}.mom6_${ens}.r.${start_date}.nc" > rpointer.ocn_${ens}.${start_date}       # mom6
    echo "${casename}.cice_${ens}.r.${start_date}.nc" > rpointer.ice_${ens}.${start_date}        # cice
    echo "${casename}.datm_${ens}.r.${start_date}.nc" > rpointer.atm_${ens}.${start_date}         # atm
    echo "${casename}.cpl_${ens}.r.${start_date}.nc" > rpointer.cpl_${ens}.${start_date}          # cpl      

    # Change the time variable
    mom6_file=${casename}.mom6_${ens}.r.0054-01-01-00000.nc
    cice_file=${casename}.cice_${ens}.r.0054-01-01-00000.nc
    cpl_file=${casename}.cpl_${ens}.r.0054-01-01-00000.nc
    ncap2 -O -s "Time(:)=19345" "$mom6_file" "$mom6_file"
    ncap2 -O -s "curr_ymd=00540101" "$cpl_file" "$cpl_file"
    ncatted -O -h -a myear,global,o,i,54 $cice_file

    ens=$(printf "%04d" $((10#$ens + 1)))
done
```
Note that the above code needs to be modified for your specific case, this is just an example for what worked in our simulation. 

## 3️⃣ Setup Data Assimilation

The following are the necessary XML variable changes to get DA going in your CESM-MOM6 case

1. Turn on data assimilation in the caseroot
```bash
./xmlchange DATA_ASSIMILATION_OCN=True
```

2. Direct CESM to your data assimilation script
```bash
./xmlchange DATA_ASSIMILATION_SCRIPT=<path_to_assimilation_script>
```
- the assimilation script will point to the input.nml for DART and the relevant observations

3. Set the number of DA cycles
```bash
./xmlchange STOP_N=<number_of_days_per_cycle>
./xmlchange DATA_ASSIMILATION_CYCLES=<number_of_cycles>
```
STOP_N determines the length of each MOM6 forecast before assimilation
DATA_ASSIMILATION_CYCLES set the total number of cycles (how long the simulations runs)

## 4️⃣ Submit the Case

Once everything is configured, run the following in the caseroot
./case.submit

Your multi-instance MOM6 + DART simulation will now start running, following the ensemble and assimilation settings you configured.

  

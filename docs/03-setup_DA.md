# Setting Up a Multi-Instance MOM6 Case with DART

This page describes step-by-step how to set up and run a multi-instance MOM6 case with data assimilation (DA) using DART.

---

## 1️⃣ Create a New Case

Use **CIME** to create a new case:

```bash
create_newcase --case <casename> --res <resolution> --compset <compset> --run-unsupported --output-root <output_dir> --ninst <ens_size> --multi-driver --project=<project_#>
```
This will initialize a new multi-instance MOM6 case directory (known as the caseroot) where all files will live.

## 2️⃣ Initialize the Ensemble

In the caseroot, setup your initial ensemble

1. Set the initial ensemble date to your start date, for example mine was Jan 1 2011.
2. Copy restarts from a pre-existing spinup run (e.g., Ian’s spinup).
     - Note: If you use Jan 1 restarts from 1971–2010, this may introduce bias due to the warming trend.

## 3️⃣ Setup Data Assimilation

The following are the necessary xml variable changes to get DA going in your CESM-MOM6 case

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
DATA_SASSIMILATION_CYCLES set the total number of cycles (how long the simulations runs)

## 4️⃣ Submit the Case

Once everything is configured, run the following in the caseroot
./case.submit

Your multi-instance MOM6 + DART simulation will now start running, following the ensemble and assimilation settings you configured.

  

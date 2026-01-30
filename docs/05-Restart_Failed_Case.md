# Restarting Failed Cases

MOM6 + DART runs can fail for a variety of reasons. Below is a general guide for recovering from failures.

---

## Common Causes of Failure

1. **Wallclock limit exceeded**  
   The job ran out of allocated time. 

2. **MPICH / MPI errors**  
   Errors related to parallel communication may cause the run to crash.

---

## Failure During MOM6 Forecast

If the failure occurred **during a MOM6 forecast**, you can typically recover by simply resubmitting the case from the caseroot:

```bash
# Navigate to the case root
cd /path/to/caseroot

# Resubmit the case
./case.submit
```
---

## Failure During Data Assimilation

If the failure occurred during a DA cycle I recommend starting from the previous restarts. 

**Step 1 : Clean the rundir** <br>

Navigate to the rundir and delete the most recent restart and rpointer files associated with the failed date

```bash
rm -rf *<bad_date>
```

**Step 2 : Check the DA cycle number** <br>

Ensure the DA cycle number is correct. This might involve updating a tracking file (e.g., cycle.txt) or verifying which observation file will be assimilated next.

**Step 3: Verify adaptive inflation files are correct** (if applicable) <br>

If using prior adaptive inflation, make sure the correct inflation file is available for the next cycle.

**Step 4 : Update CIME start time** <br>

In the caseroot, update CIME to use the new state time. This is usually done in the env_run.xml file using xmlchange

```bash
./xmlchange DRV_RESTART_POINTER=rpointer.cpl.<new_date>
```
I suggest doing an xml query first on DRV_RESTART_POINTER to make sure the formatting is correct for the rpointer file. 

---

## Additional Tips

- Test short runs (i.e 1 DA cycle) to verify the simulation correctly restarted before resuming full experiment
   

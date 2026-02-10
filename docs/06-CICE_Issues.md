# Potential Issues with CICE  

This page documents some common issues that can occur when running a G-case with DART and getting CICE errors 

A great resource is the CICE documentation: https://cesmcice.readthedocs.io/en/latest/

## 1️⃣ Errors: Picard Non-convergence and Thermo Energy Conservation Error

**General issue**: During both stochastic and reference MOM6 runs with DA, CICE crashes before running a full year of simulation completes. 

**Possible explanation**: The DA increments result in ocean temperatures below the physical freezing point given the local salinity. This produces non-physical initial conditions for CICE/Icepack, leading to vertical thermodynamic failure and Picard non-convergence

**Solution**: Implement a post-DA correction that enforces linear consistency between temperature and salinity. 

$$ T_{freeze} = -0.054 \cdot S    \qquad (1) $$  

Executed with a python script that executes the following if the model fails,
1. Read MOM6 restart files after DA updates.
2. Compute the local freezing temperature using Equation (1).
3. Clamp ocean temperatures such that $T \geq T_{freeze}+\epsilon $
4. Restart the model using the corrected restart files.
 

## 2️⃣ Error: Bad departure points

Source of issue: based on the documentation this error is written from ice_transport_remap.F90 when the ice speed is causing parcels of ice to go beyond a grid cell. 

Solution: no solution yet. 

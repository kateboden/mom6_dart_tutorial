Main namelists to control the assimilation are
1. filter_nml
2. assim_tools_nml
3. model_nml

You can either change this manually or set them in the assimilate script that CESM calls. A brief overview of these namelists is given below but refer to the DART docs for more details

1. filter_nml
   - set ensemble size
   - set filepath to the observation file
   - turn on inflation
   - set the time for the observation file in case there is a mismatch between model time and dart time

2. assim_tools_nml
   - set the localization radius, for example mine is set to 0.08 ~1,000 km
     
3. model_nml
   - set assimilation period
   - add clamping to model state variables, I recommend -1.92 as a lower bound for temperature

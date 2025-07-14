# CP2K Polaron MD

__Author__: Chris Ahart
-- -

![Hole polaron](https://www.westlake.edu.cn/campuslife/yungu/202212/W020221205545093655086.png)

The tutorial extends the previous tutorial on geometry optimisation of polarons to consider DFT-MD of polarons in anatase TiO2. Hopefully by the end of this tutorial you will be comfortable generating images such as the one above.

CP2K is generally regarded as the best for DFT-MD, as it was originally designed for MD simulations of large biological and chemical systems. The use of short-range basis sets (DZVP-MOLOPT-SR) in combination with efficient SCF minimization methods (Orbital Transform) and MD wavefunction propagators (Always stable predictor corrector) result in scalable and stable DFT-MD calculations even for very large system sizes. Do note however that CP2K has generally very poor support for k-points, so you should use VASP if you would like to perform DFT-MD with k-points. Similarly, I recommend using VASP if you intend to study metallic or small band gap materials as Orbital Transform requires a moderate band-gap (>0.5 eV). 

To begin, we perform PBE+U MD of a 2x2x1 supercell of anatase TiO2. Refer to your HPC provider of choice for a job submission script, and run CP2K using the provided input file. CP2K will generate a number of output files, including a log file and multiple MD specific output files: 'tio2-1.ener', 'tio2-frc-1.xyz', 'tio2-pos-1.xyz' and 'tio2-vel-1.xyz' which contain the energy, forces, positions and velocities of the atoms. You can load 'tio2-pos-1.xyz' in VMD and watch your atoms move.

CP2K does not have a good way to automatically increase the temperature during MD. While there is a simulated annealing keyword, the temperature profile is not linear and therefore I recommend performing equilibration manually. To do this, we generally want to start at some low temperature and slowly increase the temperature. For example we can perform 10 ps at 100 K, then restart at 200 K and run for another 10 ps, and repeat this process until the desired temperature is reached. For initial equilibration you can use the following keywords
```txt
     ENSEMBLE NVE
     COMVEL_TOL 1e-10
     TEMP_TOL 10
```
This performs velocity rescaling whenever the temperature is 10 K above or below the target temperature, and will also re-scale velocities to ensure that the centre of mass remains constant. Once you have performed a short initial equilibration at your target temperature, you can switch to the canonical sampling through velocity rescaling thermostat
```txt
     ENSEMBLE NVT
     &THERMOSTAT
       TYPE CSVR
       REGION GLOBAL
       &CSVR
         TIMECON  1
       &END CSVR
     &END THERMOSTAT
```

The keyword 'TIMECON' is  the time constant of the thermostat, the small the timecon the stronger the thermostatting. Ideally, you should use TIMECON 1 for equilibration and TIMECON 1000 for production runs however in practice I often use TIMECON 1 for production runs as well as equilibration at DFT-MD can often be prohibitively expensive.

Once you have performed equilibration with the CSVR thermostat for some appropriate amount of time, you may then want to switch to ENSEMBLE NPT_F in combination with STRESS_TENSOR ANALYTICAL which will allow your cell to change and maintain constant pressure. I would recommend first equilibrating with NVT and then switching to NPT_F.

## Other remarks:
```
1. The timestep 1.0 fs is generally suitable for most systems. Increasing this to 2.0 is possible, but may cause a decrease in accuracy and MD stability. 
2. MD stability in CP2K is generally good. However, very occasionally you will encounter bad geometries that may cause problems in the SCF convergence. You should check the column in 'tio2-1.ener' for 'UsedTime[s]' to make sure the value is roughly constant in time, and periodically check the CP2K output file to make sure the SCF is converging smoothly. There is a keyword 'IGNORE_CONVERGENCE_FAILURE' with default value FALSE, that will cause CP2K to crash if the SCF convergence fails. You may set this to TRUE, so long as you are careful. 
```
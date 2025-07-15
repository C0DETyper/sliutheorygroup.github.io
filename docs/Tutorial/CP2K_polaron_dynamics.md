# CP2K Polaron MD

__Author__: Chris Ahart
-- -

![Hole polaron](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-221/md/neutral-4hours-100k-COMVEL_TO-1e-10-TEMPTOL-10-200k-300k-dz-400k-500k-csvr-timecon-1-COMVEL_TO-1e-10-nvt-u-3.5-hole-timestep-1.0-o-3.5-ti-0/hirshfeld_spin_all.png)
![Hole polaron](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-221/md/neutral-4hours-100k-COMVEL_TO-1e-10-TEMPTOL-10-200k-300k-dz-400k-500k-csvr-timecon-1-COMVEL_TO-1e-10-nvt-u-3.5-hole-timestep-1.0-o-3.5-ti-0/bond_lengths_average.png)

The tutorial extends the previous tutorial 'CP2K Polaron Geometry Optimisation' to consider DFT-MD of polarons in anatase TiO2. Please work through part 1 before following this tutorial. Hopefully by the end of this tutorial you will be comfortable generating images such as the ones above.

CP2K is generally regarded as the best software for DFT-MD, as it was originally designed for MD simulations of large chemical and biological systems. The use of short-range basis sets (DZVP-MOLOPT-SR) in combination with efficient SCF minimization methods (Orbital Transform) and MD wavefunction propagators (Always Stable Predictor Corrector) result in highly scalable and stable DFT-MD calculations. Do note however that CP2K has generally very poor support for k-points, so you should use VASP if you would like to perform DFT-MD with k-points. Similarly, I recommend using VASP if you intend to study metallic or small band gap materials as Orbital Transform requires a finite band-gap (>0.5 eV). 

All files corresponding to the tutorial are available on our [group Github page](https://github.com/LiuTheoryLab/wiki_cp2k). I have organised the files such that the examples you can run are in the folder /examples, and my origonal files are also included for reference in a seperate folder. 

To begin, we perform PBE+U MD of a 2x2x1 supercell of anatase TiO2. Refer to your HPC provider of choice for a job submission script, and run CP2K using the [provided input file](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-221/example/neutral-100k/input/input.inp). CP2K will generate a number of output files, including a [log file](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-221/md/neutral-4hours-100k-COMVEL_TO-1e-10-TEMPTOL-10/cp2k_log.log) and multiple MD specific output files: ['tio2-1.ener'](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-221/md/neutral-4hours-100k-COMVEL_TO-1e-10-TEMPTOL-10/tio2-1.ener), ['tio2-frc-1.xyz'](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-221/md/neutral-4hours-100k-COMVEL_TO-1e-10-TEMPTOL-10/tio2-frc-1.xyz), ['tio2-pos-1.xyz'](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-221/md/neutral-4hours-100k-COMVEL_TO-1e-10-TEMPTOL-10/tio2-pos-1.xyz) and ['tio2-vel-1.xyz'](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-221/md/neutral-4hours-100k-COMVEL_TO-1e-10-TEMPTOL-10/tio2-vel-1.xyz) which contain the energy, forces, positions and velocities of the atoms. You can load ['tio2-pos-1.xyz'](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-221/md/neutral-4hours-100k-COMVEL_TO-1e-10-TEMPTOL-10/tio2-pos-1.xyz) in VMD and watch your atoms move.

CP2K does not have a good way to automatically increase the temperature during MD. While there is a [simulated annealing keyword](https://manual.cp2k.org/trunk/CP2K_INPUT/MOTION/MD.html#CP2K_INPUT.MOTION.MD.ANNEALING), the temperature profile is not linear and therefore I recommend performing equilibration using an external driving code such as MD or by doing so manually. To do this, we generally want to start at some low temperature and slowly increase the temperature. For example we can perform dynamics for some amount of time at 100 K, then restart at 200 K and run for another amount of time, and repeat this process until the desired temperature is reached. 

For initial rapid equilibration you can use the following keywords
```txt
     ENSEMBLE NVE
     COMVEL_TOL 1e-10
     TEMP_TOL 50
```
This performs velocity rescaling whenever the temperature is 50 K above or below the target temperature, and will also re-scale velocities to ensure that the centre of mass remains constant. 

Once you have performed a short initial equilibration at your target temperature, you should switch to the canonical sampling through velocity rescaling thermostat
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

The keyword [TIMECON](https://manual.cp2k.org/trunk/CP2K_INPUT/MOTION/MD/THERMOSTAT/CSVR.html#CP2K_INPUT.MOTION.MD.THERMOSTAT.CSVR.TIMECON) is the time constant of the thermostat, the smaller the timecon the stronger the thermostatting. Ideally, you should use TIMECON 1 for equilibration and TIMECON 1000 for production runs however in practice I typically use TIMECON 1 for both. I generally perform DFT-MD with hybrid functionals, where equilibrating the system fully is not possible and therefore a small TIMECON is necessary. If you are performing PBE-MD, then you should be able to perform full equilibration.

Once you have performed equilibration with the CSVR thermostat for some appropriate amount of time, you may then choose to switch to [ENSEMBLE](https://manual.cp2k.org/trunk/CP2K_INPUT/MOTION/MD.html#CP2K_INPUT.MOTION.MD.ENSEMBLE) NPT_F in combination with [STRESS_TENSOR](https://manual.cp2k.org/trunk/CP2K_INPUT/FORCE_EVAL.html#CP2K_INPUT.FORCE_EVAL.STRESS_TENSOR) ANALYTICAL which will allow your cell to change and maintain constant pressure. I would recommend always first equilibrating with NVT and then switching to NPT_F.

Finally, you can apply what you learnt in the previous tutorial to set the charge and mulitplicity of the calculation to run PBE+U MD with an[ electron hole polaron](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-221/md/neutral-4hours-100k-COMVEL_TO-1e-10-TEMPTOL-10-200k-300k-dz-400k-500k-csvr-timecon-1-COMVEL_TO-1e-10-nvt-u-3.5-hole-timestep-1.0-o-3.5-ti-0/input/input.inp). You can then plot images such as the ones at the beginning of this tutorial using the [provided Python script](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/scripts/bulk_dft_electron_tio2_share.py) as a guide.

## Other remarks:
1. MD stability in CP2K is generally good. However, occasionally you will encounter bad geometries that may cause problems in the SCF convergence. You should check the column in 'tio2-1.ener' for 'UsedTime[s]' to make sure the value is roughly constant in time, and periodically check the CP2K output file to make sure the SCF is converging smoothly. There is a keyword [IGNORE_CONVERGENCE_FAILURE](https://manual.cp2k.org/trunk/CP2K_INPUT/FORCE_EVAL/DFT/SCF.html#CP2K_INPUT.FORCE_EVAL.DFT.SCF.IGNORE_CONVERGENCE_FAILURE) with default value FALSE, that will cause CP2K to crash if the SCF convergence fails. You may set this to TRUE, so long as you are careful. 
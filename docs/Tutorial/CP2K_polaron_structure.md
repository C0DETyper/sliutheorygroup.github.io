# CP2K Polaron Geometry Optimisation

__Author__: Chris Ahart
-- -

![Hole polaron](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/polaron.png)

In this tutorial I will show how to perform calculations of polaron structures in CP2K. Hopefully by the end of this tutorial you will be comfortable generating images such as the one above. The system chosen is the hole polaron in anatase TiO2, where DFT+U calculations are capable of reproducing polaron structures in qualitative agreement with hybrid calculations. To make this tutorial suitable for those without tier 1 HPC access, I will not include any hybrid calculations and will only consider small supercells. If you are interested to use hybrid functionals in CP2K, then please speak with me.

This tutorial is not intended as a beginners guide to DFT or CP2K. For those new to CP2K, I recommend watching the [recorded lectures](https://www.youtube.com/watch?v=v2vnZbhNEpw&list=PLrmNhuZo9sgYJqeTWUhdhtYu3Ol988prg) and working through the exercises from the [CP2K Workshop 2019 at Ghent University](https://www.cp2k.org/events:2019_cp2k_workshop_ghent:index). I have included some additional links at the at end of tutorial.

All files corresponding to the tutorial are available on our [group Github page](https://github.com/LiuTheoryLab/wiki_cp2k). I have organised the files such that the examples you can run are in the folder /examples, and my original files are also included for reference in a separate folder. 

To begin, we perform PBE geometry optimisation of anatase TiO2. Refer to your HPC provider of choice for a [job submission script](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-441/example/neutral/pbe/run.slurm), and run CP2K using the [provided input file](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-441/example/neutral/pbe/input/input.inp). CP2K will generate a number of output files, including a [log file](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-441/mp-390/neutral/pbe-u-o/pbe/cp2k_log.log) which contains all relevant information for the calculation. 

For a CP2K log file ['cp2k_log.log'](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-441/mp-390/neutral/pbe-u-o/pbe/cp2k_log.log) run command 

```txt
grep 'Total F' cp2k_log.log 
```
```txt
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5789.263100175019645
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5789.265174970603766
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5789.265770267490552
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5789.266020404184019
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5789.266154911647391
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5789.266211006044614
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5789.266221379895796
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5789.266220389974478
```

For a geometry optimisation including 6 steps, there are 8 total energies. The last two energies should be equal within numerical error, as CP2K will re-evaluate the energy after completing the geometry optimisation. Because this is a optimisation for neutral geometry starting from a good starting geometry, the energy smoothly decreases to the minimum.

Run command 
```txt
 grep 'HOMO - LUMO gap' cp2k_log.log | tail -n 2
```
```txt
 HOMO - LUMO gap [eV] :    2.065950
 HOMO - LUMO gap [eV] :    2.065950
```

This is the HOMO LUMO gap, 2.07 eV compared to the [experimental band gap 3.2 eV](https://doi.org/10.1016/0038-1098(93)90427-O). This underestimation of the band gap is expected for PBE, and generally hybrid functionals are required to reproduce experimental band gaps. 

Next we perform a geometry optimisation starting using the automatically generated restart file 'tio2-1.restart'. You should make a new folder and re-name 'tio2-1.restart' to an input filename of your choice such as ['input.inp'](https://github.com/LiuTheoryLab/wiki_cp2k/blob/main/tio2/anatase/cell-441/mp-390/hole/from-neutral/pbe-u-o/pbe/input/input.inp). Change lines 'CHARGE 0' to 'CHARGE 1' and 'MULTIPLICITY 1' to 'MULTIPLICITY 2' in 'input.inp' to create a charge of 1, equivalent to removing an electron from the system. The multiplicity keyword can just be removed and CP2K will automatically calculate it, but it is worthwhile to get in the habit of controlling the multiplicity yourself as it must be manually set if studying magnetic systems. Run this new calculation, and run same commands 

```txt
grep 'Total F' cp2k_log.log 
```
```txt
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5789.465546168061337
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5789.465793318644501
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5789.465839580055217
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5789.465843418531222
```

```txt
grep 'HOMO - LUMO gap' cp2k_log.log | tail -n 2
```
```txt
 HOMO - LUMO gap [eV] :    2.074650
 HOMO - LUMO gap [eV] :    0.006997
```

We find that that the geometry optimisation does not result in polaron formation. This can be seen by the negligible change in energy upon geometry optimisation, and by the HOMO-LUMO gap. CP2K will always remove electrons from the beta spin channel and add electrons to the alpha spin channel, as such the alpha spin channel HOMO-LUMO gap is unchanged as the number of alpha electrons is the same while we have removed an electron from the valence band of the beta spin channel. The HOMO-LUMO gap of 0.0 eV for the beta spin channel means that the electron hole is fully delocalised across the valence band. When a polaron forms, there will be a non-zero HOMO-LUMO gap that is referred to as a 'mid-gap state'. 

Now, let us include a Hubbard U term. This is performed by adding a new section to the input file to the section that describes the electronic structure of oxygen atoms

```txt
     &KIND "O"
       BASIS_SET "DZVP-MOLOPT-SR-GTH-q6"
       ELEMENT "O"
       POTENTIAL "GTH-PBE-q6"
       &DFT_PLUS_U T
         L 1
         U_MINUS_J  [eV] 6.0
       &END DFT_PLUS_U
     &END KIND
```

CP2K automatically will default to units of Hartree for U_MINUS_J, so it is important to add units [eV] to tell CP2K to use eV. Running the file should result in a longer geometry optimisation, which for my calculation is 28 steps including the 6 steps from neutral geometry optimisation as we have restarted using the .restart file without resetting the number of geometry optimisation steps (keyword ['STEP_START_VAL'](https://manual.cp2k.org/trunk/CP2K_INPUT/MOTION/GEO_OPT.html#CP2K_INPUT.MOTION.GEO_OPT.STEP_START_VAL)).

Running the same commands:

```txt
 grep 'Total F' cp2k_log.log 
 ```
 ```txt
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.858935726821983
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.879371841569082
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.886847055649014
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.890017025734778
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.892148723839455
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.892938473601134
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.893230061487884
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.893451266526426
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.893728826911683
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.893903983739619
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.894352869430804
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.894818912010123
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.891301667059452
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.895923725079228
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.896613722910843
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.895942492019458
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.897773394742217
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.897976732104325
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.898087491957995
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.898147951789724
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.898181120099252
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.898200267414722
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.898205193726426
 ENERGY| Total FORCE_EVAL ( QS ) energy [a.u.]:            -5772.898198909129860
```

```txt
grep 'HOMO - LUMO gap' cp2k_log.log | tail -n 2
```
```txt
 HOMO - LUMO gap [eV] :    2.385723
 HOMO - LUMO gap [eV] :    2.290350
```

We can calculate the difference in the first and last energy -5772.858935726821983 - -5772.898198909129860 = 0.039263182307877 and convert to eV = 1.07 eV. This is the trapping energy, also known as the polaron self-trapping energy, polaron formation energy or the polaron reorganisation energy. In addition, we see that there is a HOMO-LUMO gap of 2.29 eV in the beta spin channel. This means that a polaron mid-gap state has formed close to the conduction band. You can also look at the Hirshfeld charge analysis, which for my calculation is found at hirshfeld/hirshfeld-1_28.hirshfeld. '28' refers to the index of the geometry optimisation, so find your equivalent file. In this file I find line 

```txt
    175       O      2       6.000    3.560   2.581            0.980     -0.141
```
This means that the polaron has localised on atom 175, with a spin moment of +0.980 and a charge of -0.141. While a negative charge looks wrong for what should be an electron hole polaron, you can compare with your neutral geometry optimisation where you will find a charge of -0.294. Therefore the electron hole polaron localises with a change in charge of  -0.141--0.294 = 0.153. This is much smaller than the change in spin moment of 0.980, and is typical of polaron formation in condensed phase systems where the electron density will redistribute to lower the total energy of the system. If you are new to charge analysis, then be aware that charges are not experimental observables and different charge partitioning schemes will calculate different charges. Therefore, you should not attempt to compare charges calcualted with different partioning schemes or between different codes. 

To finish, you can plot the CP2K output .cube files using VESTA or VMD. The files 'SPIN_DENSITY' contain the spin density (difference in electron density for the alpha and beta spin channels) printed at each geometry optimisation step. You should see that the spin density is initially delocalised over all atoms, and as the geometry optimisation progresses the spin density localises onto a single oxygen atom. The final spin density should be consistent with the image presented at the beginning of this tutorial.

This tutorial continues in part 2, running AIMD simulations in CP2K.

## Extensions:

1. To extend this work, you could look at the change in trapping energy and band gap as a function of the Hubbard U. You will find that the trapping energy increases linearly with U, and that no value of oxygen Hubbard U can reproduce the band gap. To reproduce the band gap, you can add a Hubbard U to the Ti atoms. I find that a value of U = 6 eV for Ti and a value of U = 2 for O reproduces the experimental band gap.
2. You could also look at the change in trapping energy with supercell size. Ideally, you should increase the supercell size until the trapping energy is converged. In practice, this may not always be possible and some compromise must be found between accuracy and computational cost. 
3. You can also run hybrid DFT calculations, which will allow you to get a much more accurate polaron trapping energy and related electron transfer parameters. This will require a few hundred CPUs, so I do not include such calculations here. 

## Regarding accuracy:
From my understanding of plane wave DFT calculations, the total energy can be converged with respect to the plane wave cutoff in a well behaved manner. This is not the case for Gaussian type orbital codes such as CP2K. You are never going to perform a CP2K calculation that is converged with respect to total energy. Instead, focus on energy differences relevant to properties that you are calculating. Generally, most structural and electronic properties are well described at double-zeta basis set level and it is rare to find condensed phase calculations that are at triple-zeta level or above. There is however [some evidence that the widespread use of double-zeta basis sets is problematic](https://pubs.acs.org/doi/10.1021/acs.jpcc.9b03554). If you are interested, refer to the [thesis of Tiziano Müller](https://www.zora.uzh.ch/id/eprint/258698/1/timuel-thesis.pdf) where he presents the convergence of total energy with respect to basis set size in CP2K versus the all-electron plane wave code Wien2k. He ultimately recommends use of triple-zeta basis sets. 

## Other remarks:
1. CP2K is a challenging code to use, with many keywords and generally poor documentation. It can take a while to become experienced with CP2K, and get a 'feel' for appropriate keyword choices. The values in this work are a reasonable starting point, and should work for most systems that are not magnetic or strongly correlated. 
2. Polarons are generally only indirectly measured by experiments. For example in conductivity measurements polarons are characterised by an increase in conductivity with temperature, as there is an activation barrier for polaron hopping. This is different to band transport, characterised by a decrease in conductivity with temperature as a result of phonon scattering. As such, polaron calculations can be seen as problematic becasue the structure and dynamics of polarons are only indirectly measurable by experiment. There is often considerable disagreement regarding the structure and dynamics of polarons in different materials.
3. Optimally tuned range-separated hybrid functionals such as HSE or PBE0-TC-LRC remain the only appropriate functionals for polaron structure and dynamics. As these are expensive, I am currently exploring how machine learning can be used to increase the accessible length-scales and time-scales while preserving hybrid level accuracy. I hope that in the future advances in density functional theory will provide more accurate and cheaper functionals, however this seems unlikely to happen in the near-future. 

## CP2K website resources:
1. [CP2K Github releases](https://github.com/cp2k/cp2k/releases)
2. [CP2K tutorials](https://www.cp2k.org/howto)
3. [CP2K manual](https://manual.cp2k.org/trunk)
4. [CP2K Google group](https://groups.google.com/group/cp2k)

## Other CP2K resources:
1. [CP2K introductory course provided by the UK HPC ARCHER2](https://www.archer2.ac.uk/training/courses/240408-cp2k/)
2. [CP2K publication](https://doi.org/10.1063/5.0007045)

## Other tutorials I have written:
1. [Potential control and current induced forces using CP2K+SMEAGOL, Imperial College London](https://wiki.ch.ic.ac.uk/wiki/index.php?title=Potential_control_and_current_induced_forces_using_CP2K%2BSMEAGOL)
2. [Converging magnetic systems in CP2K, Imperial College London](https://wiki.ch.ic.ac.uk/wiki/index.php?title=Converging_magnetic_systems_in_CP2K)

## Other tutorials I have contributed to:
1. [Trends in catalytic activity, Imperial College London](https://wiki.ch.ic.ac.uk/wiki/index.php?title=TrendsCatalyticActivity)
2. [Constrained DFT, CP2K website](https://manual.cp2k.org/trunk/methods/dft/constrained.html)



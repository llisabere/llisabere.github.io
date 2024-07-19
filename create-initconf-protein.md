---
theme: default
title: Create initial AA configuraion for long protein
---

Go [back](./).

### All-atom protein

We will make an esamble of coarse-grained configurations to backmap into all-stom and run normal md.

General procedure:
1. Make a coarse-grained coniguraion using tleap from amber tools;
2. Create mpipi force field topology and run gromacs simulation;
3. Check some basic run features: radius of gyration, minimum image distance;
4. Choose configurations and backmap them to AA using pulchra;
5. Run gromacs AA run.

Now let us look into every step in more details.

#### 1. Make a coarse-grained coniguraion using tleap from amber tools

An example is shown on elf3 protein. Run tleap input file with your sequence (example can be found [here](./create-configuration.html)). Pay attantion: number of amino acids per line should be reasonable, add new lines if nessesary. 

```usr/bin/amber18/bin/tleap -f script.leap```

#### 2. Create mpipi force field topology and run gromacs simulation

Now do some magic to clean up the file and prepare it for tabulated force field manipulations.
```
gmx editconf -f protein.pdb -o protein.pdb
sed -i 's/HIE/HIS/' protein.pdb
python3 create_top_WF-Mpipi.py -f protein.pdb -o cginit.gro -n elf3 -p cgelf3.top -e 1 -> .gro .top
python3 create_ndx_mpipi.py cginit.gro -> .ndx
```

Files ```create_top_WF-Mpipi.py``` and ```create_ndx_mpipi.py``` can be found here **add files**

Run md simulation (older gromacs version is needed to use tablulated force fields)
```
usr/bin/gromacs-2019.6-gpu/bin/gmx grompp -f ./langevin_NVT_20sigma.mdp -o md.tpr -c cginit.gro -p cgelf3.top -n index.ndx
sbatch run.slurm #gmx mdrun -v -deffnm md -nb cpu -table tables/table_WF-Mpipi.xvg -nt 1
```

Files for this can be found here **add files**

#### 3. Check some basic run features: radius of gyration, minimum image distance

Do some sanity checks like minimum image distance analysis
```gmx mindist -f md.xtc -s md.tpr -pi -od mindist_pi.xvg -dt 1e3```

#### 4. Choose configurations and backmap them to AA using pulchra
 
Calculate radius of gyration of the molecule
```gmx gyrate -f md.xtc -s md.tpr```

Choose interesting range of R_g (in some reasonable interval) and find frames that suit your choice. Get a collection of starting configurations ```initfrX.pdb```

Use pulchra software for backmapping
```
sed -i 's/BB/CA/' initfrX.pdb
usr/bin/pulchra/pulchra initfrX.pdb - >initfrX.rebuilt.pdb
```

Create gromacs topology files with force field and water model you need
```gmx pdb2gmx -f initfr4.rebuilt.pdb -o init4.heavy.gro -heavyh```

If you do not want to use hydrogen mass repartitioning delete ```-heavyh``` flag.

#### 5. Run gromacs AA simulation

Create a dodecahedron box to save up some computation time with less water molecules.
```
gmx editconf -f init4.heavy.gro -o init.newbox.gro -c -bt dodecahedron -box 15 15 15
```

Proceed with solvation, add ions in desired concentration (I assume you to have gromacs .mdp files).
```
#solvate
gmx solvate -cp init.newbox.gro -cs tip4p.gro -o init.solv.gro -p topol.top

#ions
gmx grompp -f ions.mdp -c init.solv.gro -p topol.top -o ions.tpr
gmx genion -s ions.tpr -o init.solv.ions.gro -p topol.top -pname NA -nname CL -neutral -conc 0.1

#in case of total negative charge -0.89 etc
gmx grompp -f ions.mdp -c init.solv.ions.gro -p topol.top -o ions.tpr 
gmx genion -s ions.tpr -o init.solv.ions.gro -p topol.top -pname NA -np 1

#run AA simulation
gmx grompp -f min.mdp -c init.solv.ions.gro -p topol.top -o min.tpr
gmx mdrun -v -deffnm min

gmx grompp -f nvt.mdp -c min.gro -p topol.top -o nvt.tpr -r min.gro
gmx mdrun -v -deffnm nvt

gmx grompp -f npt.mdp -c nvt.gro -p topol.top -o npt.tpr -r nvt.gro
gmx mdrun -v -deffnm npt

gmx grompp -f md.mdp -c npt.gro -p topol.top -o md.tpr
gmx convert-tpr -s md.tpr -o md.tpr -until 1e6 #1 micros
gmx mdrun -v -deffnm md -s md.tpr # -cpi md.cpt -append
```


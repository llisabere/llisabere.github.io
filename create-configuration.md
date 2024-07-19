---
layout: default
title: All-atom initial configurations
---

Go [back](./) .

Here is an example on how to create slab starting configurations with the use of ```gmx insert-molecules``` command.

## General recepy for creating slab all-atom initial configurations 

This tutorial is done using short peptide of 10 amino acids: WGRGRGRGWY. The main idea is to pack 100 peptides into a box with a density ~400 g/l and create a slab-like topology for gromacs for it. The initial density is found to be the highest gromacs accepts, adjust as needed.

### Outlook of the process:

1. Create an initial topology for 1 peptide (here we use tleap from ambertools);
2. Run 500 ns gromacs simulation;
3. Extract 2000 configurations grom the 1 peptide run;
4. Randomly pack 100 configurations into a box of given size;
5. Solvate the system;
6. Add ions to neutralize the peptides;
7. Run energy minimization;
8. Run nvt equilibration;
9. Run npt equilibration;
10. Elongate the box into the slab-like gometry;
11. Repeat steps 5-9;
12. Run md production run.
    
### Step by step procedure

#### 1. Create an initial topology for 1 peptide (here we use tleap from amber tools)
   1.1 Using tleap from amber (usually located in usr/bin/amber18/bin/tleap bur can vary) create an all-atom structure. An example tleap file looks as follows:

```
source leaprc.protein.ff14SB
wgr4= sequence { ACE TRP GLY ARG GLY ARG GLY ARG GLY TRP TYR }
savepdb wgr4 peptide.pdb 
quit
```

We are using ACE capping here for an example. 

Create a new directory called /r1_1pep_prep. Using gromacs create a topology for your peptide. We choose des-amber1.0 force field and tip4p water model in this example, adapt as needed.

```
gmx pdb2gmx -f ../peptide.pdb -o init.pdb2gmx.gro -ff des-amber1.0 -water tip4p -ignh
```

```-ignh``` flag is needed so gromacs can rearrange H atoms as per the force field you requested.

Now you should have the following files in you work directory:

| file        | meaning            |
|:------------|:-------------------|
| topol.top   | topology file      |
| posre.itp   | position restraints|

You should also have gromacs ```.mdp``` files (sample files are provided here (not really, look in gromacs tutorials)).

Create simulation box around the peptide (here, 4 nm cube), solvate it and add ions (here, 100mM concentration of Na and Cl ions):
```
gmx editconf -f init.pdb2gmx.gro -o init.newbox.gro -c -box 4
gmx solvate -cp init.newbox.gro -cs tip4p.gro -o init.solv.gro -p topol.top
gmx grompp -f ions.mdp -c init.solv.gro -p topol.top -o ions.tpr
gmx genion -s ions.tpr -o init.solv.ions.gro -p topol.top -pname NA -nname CL -neutral -conc 0.1
```

Finally, run a usual simulation procedure:
```
# energy minimization
gmx grompp -f min.mdp -c init.solv.ions.gro -p topol.top -o min.tpr
gmx mdrun -v -deffnm min

# nvt equilibration
gmx grompp -f nvt.mdp -c min.gro -p topol.top -o nvt.tpr -r min.gro
gmx mdrun -v -deffnm nvt

# npt equilibration
gmx grompp -f npt.mdp -c nvt.gro -p topol.top -o npt.tpr -r nvt.gro
gmx mdrun -v -deffnm npt

# md run
gmx grompp -f md.mdp -c npt.gro -p topol.top -o md.tpr
gmx convert-tpr -s md.tpr -o md.tpr -until 500e3 # 500 ns
gmx mdrun -v -deffnm md -s md.tpr 
```

After the run is complete (luckily, this should not take a lot of time!) apply periodic boundary conditions to the final md.xtc:

``` gmx trjconv -f md.xtc -s md.tpr -o pbc.xtc -dt 1e2 -pbc mol ```

Create a new directory /gros for sample .gro configuration files. Enter the directory. Use the script ```get_gros.sh``` to extract 2000 random conformations
```bash 
for i in {0..2000}
do
	frame=$(( ( RANDOM % 47000 )  + 2000 ))
	echo $i $frame
	echo Protein Protein | gmx trjconv -f ../pbc.xtc -s ../md.tpr -center -o initial${frame}.gro -dump $frame -pbc mol
done
```
Finally, run ```merge_pdbs.sh``` [here](./merge_pdbs.md) to pack peptides into a box. 

``` usage: merge_pdbs.sh Npep initial1111.gro Lz```

The script first packs as much peptides as it can into a Lz-10 nm box, then into Lz-5 nm box and finaly does two tries to pack into Lz box. This allows for the most packed region to be the central of the final box. Lx and Ly are fixed at 6 nm. In the end you will get ```init.Npep.gro``` configuration.


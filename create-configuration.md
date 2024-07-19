---
layout: default
title: All-atom initial configurations
---

Go [back](./) .

Here is an example on how to create slab starting configurations with the use of ```gmx insert-molecules``` command.

### General recepy for creating slab all-atom initial configurations 

This tutorial is done using short peptide of 10 amino acids: WGRGRGRGWY. The main idea is to pack 100 peptides into a box with a density ~400 g/l and create a slab-like topology for gromacs for it. The initial density is found to be the highest gromacs accepts, adjust as needed.

#### Outlook of the process:

1. Create an initial topology for 1 peptide (here we use tleap from ambertools);
2. Run 200 ns gromacs simulation;
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

#### Step by step procedure

1. Create an initial topology for 1 peptide (here we use tleap from amber tools)
   1.1 Using tleap from amber (usually located in usr/bin/amber18/bin/tleap bur can vary) create an all-atom structure. An example tleap file looks as follows:

```
source leaprc.protein.ff14SB
wgr4= sequence { ACE TRP GLY ARG GLY ARG GLY ARG GLY TRP TYR }
savepdb wgr4 peptide.pdb 
quit
```

We are using ACE capping here for an example. 

    1.2 Create a new directory called r1_1pep_prep. Using gromacs create a topology for your peptide. We choose des-amber1.0 force field and tip4p water model in this example, adapt as needed.

```
gmx pdb2gmx -f ../peptide.pdb -o init.pdb2gmx.gro -ff des-amber1.0 -water tip4p -ignh
```

```-ignh``` flag is needed so gromacs can rearrange H atoms as per the force field you requested.

Now you should have the following files in you work directory:

| file        | meaning            |
|:------------|:-------------------|
| topol.top   | topology file      |
| posre.itp   | position restraints|



  

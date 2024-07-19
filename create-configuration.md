---
layout: default
title: New
---

Go [back](./) .

Here is an example on how to create slab starting configurations with the use of ```gmx insert-molecules``` command.


### General recepy for creating slab all-atom initial configurations 

This tutorial is done using short peptide of 10 amino acids: WGRGRGRGWY. The main idea is to pack 100 peptides into a box with a density ~400 g/l and create a slab-like topology for gromacs for it.

##### Outlook of the process:

1. Create an initial topology for 1 peptide (here we use tleap from amber tools)
2. Run 200 ns gromacs simulation
3. Extract 2000 configurations grom the 1 peptide run
4. Randomly pack 100 configurations into a box of given size
5. Solvate the system
6. Add ions to neutralize the peptides
7. Run energy minimization
8. Run nvt equilibration
9. Run npt equilibration
10. Elongate the box into the slab-like gometry
11. Repeat steps 5-9
12. Run md production run

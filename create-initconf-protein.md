---
theme: default
title: Create initial AA configuraion for long protein
---

Go [back](./).

### All-atom protein

We will make an esamble of coarse-grained configurations to backmap into all-stom and run normal md.

General procedure:
1. Make a coarse-grained coniguraion using tleap from amber tools
2. Create mpipi force field topology and run gromacs simulation
3. Check some basic run features: radius of gyration, minimum image distance.
4. Choose configurations and backmap them to AA using pulchra
5. Run gromacs AA run

Now let us look into every step in more details.

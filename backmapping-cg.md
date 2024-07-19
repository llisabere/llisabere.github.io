---
theme: default
title: Backmapping
---

### Welcome to hell

Spoilert alert - I couldn't manage to stabilize those systems in all-atom resolution but maybe the backmapping is itself useful for someone

Here we use modeller, openmm modeller and in-house chaotic scripts. 

General scheme:
1. Make a coarse-grained run (one bead per residue)
2. Backmap to heavy atoms
3. Add hydrogens
4. Add implicit solvent
5. Enery minimization
6. gromacs run


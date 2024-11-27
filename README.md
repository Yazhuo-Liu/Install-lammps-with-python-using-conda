# Install-lammps-with-python-using-conda
Here we use traditional `make` build system to build LAMMPS and include additional packages like REPLICA etc. Below are detailed steps to build LAMMPS using make within your Conda environment, including the MEAM package and setting up the LAMMPS Python module for parallel computing.

## 1. Create and Activate the Conda Environment
Open a terminal and create a new Conda environment named lammps with Python 3.9:
```bash
conda create -n lammps python=3.9
```
Activate the environment:
```bash
conda activate lammps
```

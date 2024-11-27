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

## 2. Install Necessary Dependencies
Install the required packages within the lammps environment:
```bash
conda install -c conda-forge cmake openmpi mpi4py numpy boost pybind11 fftw compilers libblas liblapack gfortran blas lapack
```

## 3. Download LAMMPS Source Code
Navigate to a directory where you want to download the LAMMPS source code and clone the repository:
```bash
git clone -b stable https://github.com/lammps/lammps.git
cd lammps
```

## 4. Enable the Desired Packages
Navigate to the `src` directory:
```bash
cd src
```
Enable the REPLICA package and any other packages you need:
```bash
make yes-KSPACE yes-MC yes-MANYBODY yes-MISC yes-REPLICA yes-RIGID yes-MEAM
```

## 5. Build LAMMPS as a Shared Library
Build LAMMPS as a shared library with MPI support:
```bash
make mpi mode=shlib -j8
```
Potential Output:

The build process may take several minutes. Watch for any errors related to missing libraries or compilation issues. If errors occur, review the `Makefile` for correct paths and ensure all dependencies are installed.

## 6. Verify the Shared Library
After a successful build, confirm that `liblammps.so` exists in the `src` directory:
```bash
ls liblammps.so
```
You should see:
```bash
liblammps.so
```

## 7. Install the LAMMPS Python Module
Navigate to the python directory and install the Python bindings:
```bash
cd ../python
pip install .
```
- Alternative Method Using setup.py:
  If preferred, you can install using setup.py:
  ```bash
  python setup.py install --prefix=$CONDA_PREFIX
  ```
  Note: Using pip is generally recommended for easier management.

## 8. Set Environment Variables and Test
Ensure that the Conda environment can locate the LAMMPS shared library and Python module.
### (a) Update `LD_LIBRARY_PATH`
Add the `src` directory (where `liblammps.so` resides) to LD_LIBRARY_PATH:
```bash
export LD_LIBRARY_PATH=~/softwares/lammps/src:$LD_LIBRARY_PATH
```
- Note: Replace `~/softwares` by the path to your directory where you download the LAMMPS source code.
### (b) Update `PYTHONPATH`
Ensure that Python can locate the LAMMPS module. This is typically handled by the `pip install` step, but you can verify or manually set it if necessary:
```bash
export PYTHONPATH=$CONDA_PREFIX/lib/python$(python -c "import sys; print(f'{sys.version_info.major}.{sys.version_info.minor}')")/site-packages:$PYTHONPATH
```
### (c) Verify the Python Installation
Test if the lammps Python module is correctly installed and can be imported:
```bash
python -c "from lammps import lammps; print('LAMMPS module imported successfully')"
```
Expected Output:
```bash
LAMMPS module imported successfully
```

### (d) Test Parallel Computing with LAMMPS in Python
Navigate to Your Working Directory:
```bash
mkdir -p ~/temp
cd ~/temp
```
Create a file named `test_basic.in` with the following content:
```lammps
# test_basic.in
units metal
atom_style atomic
boundary p p p

# Create a simple simulation box
lattice fcc 3.52
region box block 0 10 0 10 0 10
create_box 1 box
create_atoms 1 box

# Define mass
mass 1 63.546

# Define a simple pair style and coefficients
pair_style lj/cut 2.5
pair_coeff 1 1 0.01034 3.40

# Initialize simulation
run 0
```
Create a file named `test_basic.py` with the following content:
```python
# test_basic.py
from lammps import lammps

# Initialize LAMMPS
lmp = lammps()

# Execute the input script
lmp.file('test_basic.in')

print("LAMMPS simulation executed successfully.")
```
Run the script using `mpirun` to utilize multiple processes. For example, to use 4 processes:
```bash
mpirun -np 4 python test_basic.py
```
You should see output similar to the following:
```bash
LAMMPS (29 Aug 2024 - Update 1)
LAMMPS simulation executed successfully.
Total wall time: 0:00:00
```

## 9. Automating Environment Variable Setup When Activating the `lammps` Conda Environment
To ensure that the necessary environment variables are automatically set every time you activate the lammps Conda environment, you can use Conda's activation scripts. This eliminates the need to manually export variables each time.

For LAMMPS to function correctly, especially with Python integration and MPI support, you typically need to set:
- `LD_LIBRARY_PATH`: To include the directory where liblammps.so resides.
- `PYTHONPATH`: To include the directory where the LAMMPS Python module is installed.

### (a) Navigate to the Conda Environment Directory
First, deactivate your lammps environment:
```bash
conda deactivate
```
Then, determine the path to your lammps environment:
```bash
conda env list
```
Look for the `lammps` environment and note its path (e.g., `/home/user/miniconda3/envs/lammps`).
```bash
cd /home/user/miniconda3/envs/lammps
```
### (b) Create Activation Scripts Directory:
```bash
mkdir -p etc/conda/activate.d
mkdir -p etc/conda/deactivate.d
``` 
### (c) Create the Activation Script:
Create a file named `env_vars.sh` in the `activate.d` directory:
```bash
nano etc/conda/activate.d/env_vars.sh
```
Add the Following Content:
```bash
#!/bin/bash

# add LD_LIBRARY_PATH
if [[ ":$LD_LIBRARY_PATH:" != *":~/softwares/lammps/src:"* ]]; then
    export LD_LIBRARY_PATH=~/softwares/lammps/src:$LD_LIBRARY_PATH
fi

# add PYTHONPATH
NEW_PYTHONPATH=$CONDA_PREFIX/lib/python$(python -c "import sys; print(f'{sys.version_info.major}.{sys.version_info.minor}')")/site-packages
if [[ ":$PYTHONPATH:" != *":$NEW_PYTHONPATH:"* ]]; then
    export PYTHONPATH=$NEW_PYTHONPATH:$PYTHONPATH
fi
```
### (d) Create the Deactivation Script:
Create a file named `env_vars.sh` in the `activate.d` directory:
```bash
nano etc/conda/deactivate.d/env_vars.sh
```
Add the Following Content:
```bash
#!/bin/bash

# remove LD_LIBRARY_PATH
if [[ ":$LD_LIBRARY_PATH:" == *":~/softwares/lammps/src:"* ]]; then
    export LD_LIBRARY_PATH=$(echo "$LD_LIBRARY_PATH" | sed -e 's|:~/softwares/lammps/src||' -e 's|~/softwares/lammps/src:||')
fi

# remove PYTHONPATH
NEW_PYTHONPATH=$CONDA_PREFIX/lib/python$(python -c "import sys; print(f'{sys.version_info.major}.{sys.version_info.minor}')")/site-packages
if [[ ":$PYTHONPATH:" == *":$NEW_PYTHONPATH:"* ]]; then
    export PYTHONPATH=$(echo "$PYTHONPATH" | sed -e "s|:$NEW_PYTHONPATH||" -e "s|$NEW_PYTHONPATH:||")
fi
```

### (e) Ensure Scripts Are Executable
Although not strictly necessary, it's good practice to make these scripts executable.
```bash
chmod +x etc/conda/activate.d/env_vars.sh
chmod +x etc/conda/deactivate.d/env_vars.sh
```
### (f) Verify the Automated Setup
After setting up the activation and deactivation scripts, it's crucial to verify that they work as intended.
- restart the terminal
- activate the lammps environment
- try 8(d) again, if it works, you finished.














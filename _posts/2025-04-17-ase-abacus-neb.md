---
title:  ASE + ABACUS + NEB
date: 2025-04-17 18:22:30 +/-TTTT
categories: [ABACUS]
tags: [ABACUS, ASE, NEB]     # TAG names should always be lowercase
description: Use ABACUS as an ASE calculator to perform NEB calculations.
toc: true
---

> ABACUS currently does not have built-in NEB calculation capability and can only perform NEB calculations by invoking ABACUS through ASE (Atomic Simulation Environment, a Python library).

> Note: I copy many things form ATST-Tools directly. Its Github Repo: https://github.com/QuantumMisaka/ATST-Tools . This post is only for archiving.

> All files in this document can be downloaded from: https://moseyqaq.github.io/assets/archive/abacus-neb.zip

## 1. Install ASE-ABACUS Interface

The original ASE does not support ABACUS, so we need to manually install it. (Note: Recommended to install in conda environment)

```bash
git clone https://gitlab.com/1041176461/ase-abacus.git
cd ase-abacus
pip install .

# Install other required Python libraries
pip install pymatgen
pip install pymatgen-analysis-diffusion
```

## 2. Generate NEB Initial Trajectory

## 2.1 Generate

Prepare initial and final state files (supported formats: VASP's OUTCAR, ABACUS's running_scf.log/running_relax.log)

Example case (polarization reversal):

1) Initial state output file: ```up_running_scf.log```

2) Final state output file:```down_running_scf.log```

```bash
# Interpolate 20 images, output as neb-init.traj
python 01-neb-make.py -i up_running_scf.log down_running_scf.log -n 20 -o neb-init.traj
```

After running, the following files will be generated:

1）neb-init.traj (ASE format trajectory)

2）POSCAR-xx (corresponding POSCAR for visualization)

## 2.2 Check

Strongly recommend visualizing the generated initial trajectory before calculation. Use ASE to view```neb-init.traj```：

```bash
ase gui neb-init.traj
```

## 2.3 Help Message

To view help message, simply use: ```python3 01-neb-make.py -h ```:

```Bash
usage: 01-neb-make.py [-h] -n N [-f FORMAT] -i INPUT INPUT [-m {IDPP,linear}] [-o O]
                      [-sort_tol SORT_TOL] [--fix FIX] [--mag MAG]

Make input files for NEB calculation

options:
  -h, --help            show this help message and exit
  -n N                  Number of images
  -f FORMAT, --format FORMAT
                        Format of the input files, default is abacus-out
  -i INPUT INPUT, --input INPUT INPUT
                        IS and FS file
  -m {IDPP,linear}, --method {IDPP,linear}
                        Method to generate images
  -o O                  Output file
  -sort_tol SORT_TOL    Sort tolerance for matching the initial and final structures, default is 1.0
  --fix FIX             [height]:[direction] : fix atom below height (fractional) in direction
                        (0,1,2 for x,y,z), default None
  --mag MAG             [element1]:[magmom1],[element2]:[magmom2],... : set initial magmom for atoms
                        of element, default None
```

## 3. Submit Job

### 3.1 python script

Modify the ```02-neb-run.py```, each parameter has comments for refernece:

![alt text](/assets/img/ase-neb.png)

### 3.2 Runscript

Submission script```runscript```（Note: Requires activating conda environment with ASE-ABACUS）：

```bash
#!/bin/bash
#SBATCH --partition=liushi,intel-sc3-32c,intel-sc3
#SBATCH -N 1               # Should be defined manually
#SBATCH --cpus-per-task=4      # Should be defined manually. Equal to $OMP_NUM_THREADS
#SBATCH --ntasks-per-node=8    # Should be defined manually
#SBATCH --mem=100G
#SBATCH --qos=huge

module purge
module load abacus/3.7.1-zen3
source /home/liushiLab/lidenan/software/miniconda3/bin/activate
export OMP_NUM_THREADS=4

python3 02-neb-run.py
```

## 4. Job Monitoring

After submission, the slurm output will look like this. It also generates ```neb.traj``` containing NEB results at each step, readable by ASE"

```bash
      Step     Time          Energy          fmax # Maximum force acting on atom
BFGS:    0 13:08:33   -27773.721348        0.084962
BFGS:    1 13:11:17   -27773.723322        0.065365
BFGS:    2 13:14:02   -27773.727231        0.052534
BFGS:    3 13:16:45   -27773.728308        0.053460
BFGS:    4 13:19:29   -27773.729948        0.049554
BFGS:    5 13:22:13   -27773.731250        0.056479
BFGS:    6 13:24:56   -27773.732594        0.056046
BFGS:    7 13:27:42   -27773.733596        0.045237
BFGS:    8 13:30:27   -27773.734615        0.044459
BFGS:    9 13:33:12   -27773.735753        0.057208
BFGS:   10 13:35:56   -27773.737030        0.066975
BFGS:   11 13:38:39   -27773.738280        0.067569
BFGS:   12 13:41:24   -27773.739483        0.062210
BFGS:   13 13:44:08   -27773.740690        0.057106
BFGS:   14 13:46:53   -27773.741956        0.056265
BFGS:   15 13:49:38   -27773.743143        0.070447
BFGS:   16 13:52:22   -27773.744168        0.069835
BFGS:   17 13:55:05   -27773.745026        0.047235
BFGS:   18 13:57:49   -27773.745843        0.048347
BFGS:   19 14:00:33   -27773.746733        0.065234
BFGS:   20 14:03:17   -27773.747711        0.071390
BFGS:   21 14:06:01   -27773.748747        0.063422
BFGS:   22 14:08:46   -27773.749800        0.050066
...
```

## 5. Data Extraction


```bash
python3 03-neb-post.py neb.traj 20 # 20 = nimages
```


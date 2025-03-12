---
title: Install pyatb in Westlake-SE
date: 2025-03-11 15:03:01 +/-TTTT
categories: [compile]
tags: [pyatb, compile]     # TAG names should always be lowercase
description: Install pyatb at Westlake-HPC-SE cluster
toc: false
---

## Create new conda environment

```bash
conda create -n pyatb python=3.13
conda activate pyatb
```

## Load Intel MKL and MPI

```bash
module load oneAPI/2024.0
```

## Install dep

```bash
pip3 install pybind11 mpi4py
```

## Get source code

```bash
git clone https://github.com/pyatb/pyatb.git
cd pyatb
```

## Change siteconfig.py

> You should change it according to your own setup
{: .prompt-tip }

```bash
compiler = 'icpx' # Use non-mpi c++ compiler

mkl_library_dir = '/soft/compiler/intel/oneapi-2024.0/mkl/2024.0/lib/intel64/' # path to lib of MKL

mkl_include_dir = '/soft/compiler/intel/oneapi-2024.0/mkl/2024.0/include' # path to header of MKL

eigen_include_dir = '/soft/devtools/eigen-3.4.0/include/eigen3/' # path to header of eigen3
```

## Install

```bash
pip install .
```

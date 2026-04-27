---
layout: splash
title: "ExaGRyPE Performance Report"
permalink: /reports/exagrype
classes: wide  
---

# ExaGRyPE Performance Report

A test submission for the performance assessment of [ExaGRyPE](https://hpcsoftware.pages.gitlab.lrz.de/Peano/dc/d1a/applications_exahype2_ExaGRyPE.html), available as part of the [Peano repository](https://gitlab.lrz.de/hpcsoftware/Peano.git) under the stable branch `p4`. Pre-assessment was completed on 27-04-2026, with a recommendation to proceed with high-level assessment.

## Disclaimers

1. This report is not a commentary on code quality, but an indicator of the quality of the current SHAREing testing methodology as of 07-04-2026.
2. The pre-assessment is only a preliminary assessment of submission suitability and does not guarantee a full assessment. It will be provided to the submitter indicating if the full assessment will be undertaken or detail reasons for rejection.

## Table of contents

- [x] [1: Benchmark setup](#1-benchmark-setup)
- [x] [2: Description of working environment](#2-description-of-working-environment)
- [x] [3: Compilation and optimisations](#3-compilation-and-optimisations)
- [x] [4: Runtime behaviour, memory, storage and I/O](#4-runtime-behaviour-memory-storage-and-io)
- [x] [5: Computational complexity and scaling](#5-computational-complexity-and-scaling)
- [x] [6: Pre-assessment outcome](#6-pre-assessment-outcome)

## 1: Benchmark setup

### Fetch and build program

The main codebase, Peano, that ExaGRyPE is a part of, can be cloned using:

```bash
git clone -b p4 https://gitlab.lrz.de/hpcsoftware/Peano.git
```

The submitter specifies the version on the `p4` branch (which is stable) must be used. To compile Peano, the submitter provided the following instructions:

```bash
libtoolize; aclocal; autoconf; autoheader; cp src/config.h.in .

automake --add-missing

./configure CXX=icpx CC=icx CXXFLAGS="-O3 -std=c++20 -xHost -fiopenmp -fsycl -Wno-unknown-attributes -Wno-gcc-compat" LDFLAGS="-fiopenmp -fsycl" LIBS="-ltbb" --enable-particles --enable-exahype --enable-blockstructured --with-multithreading=omp --enable-loadbalancing --with-mpi=mpicxx

make
```

### Fetch and run benchmark

The benchmark itself is part of the codebase and can be accessed, compiled and run using:

```bash
cd benchmarks/exahype2/ccz4/single-black-hole

export PYTHONPATH=../../../../python:../../../../applications/exahype2/ccz4

python3 performance-studies.py -s fv-6 -et 1e-2 --plot 0.0 --cell-size 1.8 --trees 32 --scheduler native --output test

./test
```

The submitter has confirmed that the benchmark should take approximately 10 minutes to run. The regions of interest in the output can be identified as follows (comments starting with `//` are quotes from the submitter explaining each section):

```c
// Calculation starts as the code plots start
AlgorithmicStep [TimeStep]

//The quantity of interest is the actual solve phase which is plotted eventually as
time stepping: 85.1353s (avg=17.0271,#measurements=5,max=23.0463(value #0),min=5.28955(value #3)
```

The parameter of interest to vary in the benchmark is `--cell-size`. This varies the number of cells value which can be read from the output with the following format:

```text
27 tree(s): (#0:53/390)(#1:53/390)(#2:105/754)(#3:53/390)(#4:105/754)(#5:261/1534)(#6:105/754)(#7:53/390)(#8:105/754)(#9:53/390)(#10:105/754) (#11:261/1534)(#12:105/754)(#13:261/1534)(#14:20229/3250)(#15:261/1534)(#16:105/754)(#17:261/1534)(#18:105/754)(#19:53/390)(#20:105/754) (#21:53/390)(#22:105/754)(#23:261/1534)(#24:105/754)(#25:53/390)(#26:105/754) total=23479/24622 (local/virtual)
```

### Reference architecture

The submitter has provided the [Hamilton supercomputer](https://www.durham.ac.uk/research/institutes-and-centres/advanced-research-computing/hamilton-supercomputer/) at Durham University as a reference hardware. The assessor is based at Durham University and used the same system for this pre-assessment.

## 2: Description of working environment

### Hardware information

The hardware details for Hamilton are [available online](https://www.durham.ac.uk/research/institutes-and-centres/advanced-research-computing/hamilton-supercomputer/systems/). This information is corroborated by running `cat /proc/cpuinfo` and `likwid-topology` on one of the compute nodes via an interactive session on the test queue: `srun -p test -N 1 --pty bash`.

| Specification               | Details                                                                                                                                                           |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Number of compute nodes     | Standard: 120; High-memory: 2                                                                                                                                     |
| Processors                  | 2 $\times$ [AMD EPYC 7702 64-Core Processors](https://www.amd.com/en/support/downloads/drivers.html/processors/epyc/epyc-7002-series/amd-epyc-7702.html) per node |
| Clock speed                 | 3.30 MHz per CPU                                                                                                                                                  |
| Sockets                     | 2 per node                                                                                                                                                        |
| Cores                       | 128 per node                                                                                                                                                      |
| RAM                         | Standard: 256 GB (246 GB available to users) per node; High-memory: 2 TB per node                                                                                 |
| Local storage               | 400 GB SSD per node                                                                                                                                               |
| NUMA domains                | 8 per node (2 per socket), each with a dedicated memory channel and further split into 4 groups containing 4 cores each                                           |
| Cache                       | 16 MB L3 cache per group, 512 KB of L2 cache per core and 32 KB of L1 cache per core                                                                              |

Hamilton also has one GPU node with the following configuration (N.B. the assessor currently does not have access to this node and cannot corroborate the information):

| Specification               | Details                                                                                                                                                           |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Processors                  | 2 $\times$ [AMD EPYC 9555 64-Core Processors](https://www.amd.com/en/products/processors/server/epyc/9005-series/amd-epyc-9555.html)                              |
| GPUs                        | 8 $\times$ [NVIDIA H200 NVL](https://www.nvidia.com/en-gb/data-center/h200/)                                                                                      |
| RAM                         | 2.2 TB                                                                                                                                                            |
| Local storage               | 3 TB NVMe                                                                                                                                                         |

### Job submission configurations

[SLURM](https://slurm.schedmd.com/overview.html) is used for job scheduling on compute nodes. The full list of options on Hamilton are provided in the [online documentation](https://www.durham.ac.uk/research/institutes-and-centres/advanced-research-computing/hamilton-supercomputer/usage/jobs/#d.en.1181521) for the cluster. The following configurations are relevant to this assessment to compile the code and run the executables, with values adjusted based on use case:

```bash
-p shared      # Or "multi" if using a full node
-t 01:00:00    # Jobs are not expected to take longer than an hour to run, but will be adjusted if needed
-n 2 -c 16     # Running on 32 cores, using 2 MPI ranks and 16 threads per rank
-N 1           # Using cores on the same node
--mem=16G      # Allocating 16 GB of RAM to the job 
```

### Libraries and modules

The system uses [lmod](https://lmod.readthedocs.io/) to manage compute environments such as loading compilers etc. The documentation of the Peano codebase includes instructions for [setting up the working environment on Hamilton]([Peano](https://hpcsoftware.pages.gitlab.lrz.de/Peano/dc/d26/page_machines_hamilton.html)):

```bash
module purge
module load oneapi/2025.2
export FLAVOUR_NOCONFLICT=1
module load gcc/15.2
module load tbb/2022.0
module load intelmpi/2021.16
export I_MPI_CXX=icpx
module load python/3.13.9
```

The environment setup is confirmed using `module list`:

```text
Currently Loaded Modulefiles:
  1) oneapi/2025.2      2) gcc/15.2           3) tbb/2022.0         4) intelmpi/2021.16   5) python/3.13.9
```

### Assessment tools

1. Pre-assessment:
   - `top` and `time` for high-level wall time execution and memory usage analysis
   - `cat /proc/cpuinfo` and `likwid-topology` for hardware information

2. High-level assessment:
   - `likwid-bench` and `likwid-perfctr` for measuring FLOPS
   - `likwid-pin` for thread pinning to measure performance within and across NUMA domains
3. Low-level assessment (may require admin privileges):
   - `vtune` for detailed profiling and hardware metrics (especially for Intel systems and compilers)
   - `valgrind` for detailed memory analysis
   - `maqao` to identify opportunities for compiler optimisation

## 3: Compilation and optimisations

The [GNU Autotools](https://www.gnu.org/software/automake/manual/html_node/Autotools-Introduction.html) build system (`configure` and `make`) is used for compilation. The submitter has included the `-xHost` flag to enable hardware-specific optimisations. The `-03` optimisation flag ensures aggressive optimisation while complying with FP standards. The submitter has not reported any issues from these optimisations on the convergence and correctness of the results.

The code is compiled for the C++20 standard and using Intel compilers (`icx` and `icpx`). The submitter indicated a preference for the latest Intel compilers. On Hamilton, the latest compilers are a part of the Intel OneAPI 2025.2 suite which will be loaded with the [recommended environment setup](#libraries-and-modules). To disable MPI, the `--with-mpi` flag can be removed. Intel's version of OpenMP is used with the `-fiopenmp` flag. GPUs are not enabled by default and would have to be enabled with a `--with-gpu` flag. But the submitter has indicated that they prefer the initial analysis to be done without GPUs. The submitter has stated further details on compilation flags can be found within [Peano's documentation](https://hpcsoftware.pages.gitlab.lrz.de/Peano/de/dd9/page_installation_home.html).

The Peano codebase and the benchmark were compiled on a compute node via an interactive job with 8 GB of RAM and 32 threads, allocated on the same node: `srun --mem=8G -c 32 -n 1 -N 1 -p shared -t 01:00:00 --pty bash`. After loading the [modules](#libraries-and-modules), [Peano and the benchmark were compiled](#1-benchmark-setup) to create the `test` executable. To speed up the compilation, `make` was run with multi-threading using `-j 32`. The `-g` flag was also included to add debug symbols to the executable to enable the use of performance analysis tools during lower-level assessments.

 The compilation of Peano completed successfully. For the benchmark compilation, the `performance-studies.py` Python script was also given the additional `-j 32` parameter to limit the compilation to 32 threads (as it would attempt to compile over the whole node otherwise). However, the benchmark compilation resulted in errors due to some missing Python libraries. The assessor found the list of required libraries within the [Peano documentation](https://hpcsoftware.pages.gitlab.lrz.de/Peano/df/dbc/tutorials_exahype2_applications_exagrype_faq.html#autotoc_md723): `jinja2`, `ast_comments` and `vtk` and installed them in a virtual environment, after which the benchmark compilation ran successfully.

The executable was run after the compilation on the same interactive job. However, the run exited early with the following error:

```text
Please verify that both the operating system and the processor support Intel(R) X87, CMOV, MMX, SSE, SSE2, SSE3, SSSE3, SSE4_1, SSE4_2, MOVBE, POPCNT, AVX, F16C, FMA, BMI, LZCNT, AVX2 and ADX instructions.
```

The assessor investigated this error and determined that the issue [may be arising from the use of the `-xHost` flag](https://github.com/easybuilders/easybuild-framework/issues/4744) which is meant to enable optimisations specific to the hardware, but may not be working correctly for the AMD processors on Hamilton. The [performance assessment workbook]([perf_analysis_workbook_brief.pdf](https://shareing-dri.github.io/assets/pdfs/perf_analysis_workbook_brief.pdf)) also indicates that using `-xHost` on non-Intel hardware could result in unoptimised code. Furthermore, looking into [compilation suggestions for Peano in its documentation](https://hpcsoftware.pages.gitlab.lrz.de/Peano/d6/d5c/page_machines.html), recommendations for DINE (which also has AMD processors) indicate using the `-march=native` flag instead of `-xHost` as the latter results in runtime issues. The assessor recompiled the codebase and benchmark, replacing `-xHost` with `-march=native`, which compiled successfully. The run of the updated `test` executable with the same configuration completed without any reported errors.

## 4: Runtime behaviour, memory, storage and I/O

To analyse baseline behaviour, the benchmark was run on three configurations: single core (`-n 1 -c 1`), 32 threads on 1 MPI rank (`-n 1 -c 32`), and with 16 threads per rank on 2 MPI ranks (`-n2 -c 16`). Jobs were launched on compute nodes using a [SLURM submission script](https://www.durham.ac.uk/research/institutes-and-centres/advanced-research-computing/hamilton-supercomputer/usage/jobs/examplecpujobs/#d.en.2624582) per configuration. The submitter did not provide any information on the memory footprint of the benchmark, however, this was also not requested in the current version of the assessment form. The jobs were given access to 16 GB RAM as they only use a subset of a node's resources.

The `/usr/bin/time -v` command was used to measure the execution time, and the `sacct --format="MaxRSS"` command was used to get memory usage statistics for the jobs after completion. For the latter, all configurations were run with `mpirun` to separate the memory usage of the actual job from [resources used by the `sbatch` command](https://stackoverflow.com/questions/52447602/slurm-sacct-shows-batch-and-extern-job-names). For the `-n 2 -c 16` configuration, the maximum execution time between the 2 ranks is reported.

| MPI ranks | Threads per rank | Execution time (s) | Memory usage (MB) | Memory usage (%) | Lines of output |
| --------- | ---------------- | ------------------ | ----------------- | ---------------- | --------------- |
| 1         | 1                | 244.62             | 7018.24           | 44.25            | 193             |
| 1         | 32               |  56.94             | 439.77            | 71.25            | 399             |
| 2         | 16               |  46.46             | 3001.32           |  3.90            | 399             |

The benchmark produced up to 400 lines of output, depending on the number of threads used. This included the lines reporting the value of interest i.e. number of cells [indicated by the submitter](#fetch-and-run-benchmark), along with the processor ID, MPI rank ID and time step. All three configurations produced a total of 19683 cells by the end of the run (N.B. this value is lower than the expected 23479 cells indicated by the submitter). For the `-n 2 -c 16` configuration, the trees built per rank had 19318 and 365 cells respectively at the end of the run.

Apart from the essential console output, the submitter had stated that unnecessary I/O (plots) can be switched off by providing the `--plot 0.0` argument to `performance-studies.py`. The submitter has emphasized that using any other value for `--plot` enables the output. They also noted that the code neither reads nor writes to large input files. The assessor corroborates that the benchmark, which was compiled with `--plot 0.0` as per the submitter's instructions, did not write any files to storage not read any in.

## 5: Computational complexity and scaling

The maximum runtime of the benchmark, which is on a single core, is just over 4 minutes. Scaling the same job to 32 threads (or 2 MPI ranks with 16 threads each), which is a quarter of a node on Hamilton, already takes less than one minute, making a strong scaling analysis on this job size difficult.

However, the submitter has indicated that the benchmark's problem size should scale with the "number of cells" value. The primary problem size parameter to be varied is the `--cell-size`. In the example provided, this is set to `1.8`. The submitter stated that this can be changed in multiples of three. Increasing the size of the parameter, for example to `5.4` leads to a smaller and coarser setup. The assessor tested this and with `5.4`, the total number of cells generated were reduced to 729, taking only 12s on a single core. Conversely, running with `--cell-size 0.6` increased the number of cells generated to 531441, taking 1170.27s to run. The latter case also increased the required memory, with the `-n 1 -c 32` and `-n 2 -c 16` configurations crashing due to out-of-memory errors with a 16 GB allocation. The `--cell-size` parameter can, therefore, be easily varied for a weak scaling analysis. Furthermore, with a single core runtime of almost 20 minutes, the finer case can be used for strong scaling analysis.

The submitter has provided [a link](https://hpcsoftware.pages.gitlab.lrz.de/Peano/d4/d35/benchmarks_exahype2_ccz4_single_black_hole_hpc_assessment.html) to detailed instructions and background information to the benchmark. This includes graphs detailing [scaling](https://hpcsoftware.pages.gitlab.lrz.de/Peano/d4/d35/benchmarks_exahype2_ccz4_single_black_hole_hpc_assessment.html#autotoc_md974) in terms of load (number of cells and number of trees per rank) and runtime for two types of methodologies (regular grid and adaptive mesh). Explanations for each setting are also provided. Apart from `--cell-size`, the [linked documentation](https://hpcsoftware.pages.gitlab.lrz.de/Peano/d4/d35/benchmarks_exahype2_ccz4_single_black_hole_hpc_assessment.html) lists other variable parameters that could impact the performance and scaling such as the type of solver used, the load balancing paradigm, task scheduling etc. The submitter has also indicated that they assume that a "one MPI rank per NUMA domain" configuration may be optimal and that there is "no restriction on GPU-MPI association".

## 6: Pre-assessment outcome

The assessor was successfully able to compile and run the benchmark with multiple configurations. The problem size was also easy to scale and so the benchmark can be used for both strong and weak scaling analysis during the main analysis.

The assessor will now proceed with high-level assessment.  The submitter has specified that out of the three performance dimensions, the ones relevant to this assessment and benchmark are:

1. Core-level assessment
2. Intra-node assessment
3. GPU/accelerator assessment

As the submitter has asked that the initial assessment should be done without GPUs, the "GPU/accelerator assessment" will not be prioritised in this iteration of the full assessment However, the assessor will endeavour to gain access to the GPU node on Hamilton, or use another system, during the high-level assessment, and update the pre-assessment section accordingly.

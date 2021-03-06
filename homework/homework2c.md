Homework 2 — Parallelizing a Particle Simulation
================
Due: April 28, 2019 @ 11:59pm

**Part C: GPU**

[GitHub classroom repository: click here to get your team’s starter files](https://classroom.github.com/g/tsphKpRs)

## Important Notice

**Unlike the other homework assignments, you might not be able to run this on
your laptop or personal computer.** You **MUST** have a NVIDIA GPU, and it must
be on this [Supported GPUs list](https://developer.nvidia.com/cuda-gpus). If it’s on the list, then you
will also need to install the [CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit). If you already have
the CUDA Toolkit installed, then verify that it’s version 7 or higher.

**In addition, the settings in the SLURM submission scripts and the way you
launch interactive mode on Bridges is different from the other homeworks.**
Please review the [Submitting and running the code](#submitting-and-running-the-code)
section of this file before attempting to run an interactive GPU session.

## Problem statement

Your goal is to parallelize on XSEDE’s
[Bridges](https://portal.xsede.org/psc-bridges) supercomputer a toy particle
simulator (similar particle simulators are used in
[mechanics](http://www.thp.uni-duisburg.de/~kai/index_1.html),
[biology](http://www.ks.uiuc.edu/Research/namd/),
[astronomy](http://www.mpa-garching.mpg.de/gadget/clusters/index.html), etc.)
that reproduces the behaviour shown in the following animation:

![Animation of particle simulation](/img/animation.gif)

The range of interaction forces *(cutoff)* is limited as shown in grey for a
selected particle. Density is set sufficiently low so that given *n* particles,
only *O(n)* interactions are expected.

Suppose we have a code that runs in time *T = O(n)* on a single processor. Then
we’d hope to run in time *T/p* when using *p* processors. We’d like you to write
parallel codes that approach these expectations.

You will be executing your code on a [NVIDIA Tesla K80 GPU](https://www.nvidia.com/en-us/data-center/tesla-k80/), which
has a [compute capability of 3.7](http://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#compute-capabilities).

## Correctness and Performance

A simple correctness check which computes the minimal distance between 2
particles during the entire simulation is provided. A correct simulation will
have particles stay at greater than *0.4* (of cutoff) with typical values
between *0.7-0.8*. A simulation were particles don’t interact correctly will be
less than *0.4* (of cutoff) with typical values between *0.01-0.05* .

Adding the checks inside the GPU code provides too much of an overhead so an
`autocorrect` executable is provided that checks the output txt file for the
values mentioned above.

### Important note for Performance:

While the job-bridges-\* scripts we are providing have small numbers of
particles (4000) to allow for the *O(n<sup>2</sup>)* algorithm to finish
execution, the final code will be tested with values in the range of
`auto-bridges-gpu`.

## Grading

25% of your grade will be based on your code’s efficiency, and 75% of your grade
will depend on your report.

**GPU Scaling** will be tested via fitting multiple runs of the code to a line
on a log/log plot and calculating the slope of that line. This will determine
whether your code is attaining the *O(n)* desired complexity versus the starting
*O(n<sup>2</sup>)*. With an achieved result of *O(n<sup>x</sup>)* you will
receive:

  - If x is between 1.4 and 1.2 you will receive a scaling score between 0 and
    40 proportional to x. (Ex: 1.3 gives a score of 20.0)

  - If x is below 1.2 you will receive a scaling score of 40.

**GPU speedup** will be tested by comparing the runs with serial *O(n)* code and
finding the average over a range of particle sizes. Depending on the average
speedup the score will be:

  - If the speedup is between 2 and 4 you will receive a score between 0 and 40
    proportional to it (Ex: 3 gives a score of 20)

  - If the speedup is between 4 and 8 you will receive a score between 40 and 60
    proportional to it (Ex: 7 gives a score of 55)

  - If the speedup is above 8 you will receive a score of 60

  - If the scaling score for the GPU is 0 (aka still have *O(n<sup>2</sup>)*
    code ) the score for the speedup will be set to 0.

The total code grade is the sum of the GPU scaling and speedup scores.

## Report

This homework will be completed in groups, and each group must submit a single
joint report.

The report is to be written in a style appropriate for an academic journal with
any relevant citations provided in a bibliography. The report should only
discuss the details of the final version of your submitted code. The report is
to contain the following:

1.  A section detailing how you optimized the code. This requires both a
    conceptual discussion, for example how you partitioned the space in the problem
    (figures are helpful\!), and a code summary, where you explain each important
    step in your solution (this should be supported with code snippets).

2.  A benchmark section where you show how your code performs against the naive
    implementation, and how it scales with respect to the appropriate parameters.
    Figures are helpful when reporting this data\! Benchmark data should be taken
    from runs performed on Bridges, not your local computer or laptop. Make sure you
    discuss the reason for any odd behavior in the reported performance.

**As you can see, no introductory section is needed for this report as the
simulation and general problem is the same as in Part A and Part B.** Your
report should be converted to the PDF format prior to submission. The filename
should follow this format: `Team#_hw2c.pdf`.

## How to submit

Put your report file `Team#_hw2c.pdf` into the `doc/` directory in your GitHub
repository, and save, commit, and push the report and the final version of your
code into the master branch. Do not push the `particles` binary or any temporary
files (such as `.o` files) to GitHub (the `.gitignore` file is meant to help
prevent this). After everything is pushed and up-to-date, do the following:

1.  Have **one** group member upload the report PDF to the Homework 2c
    assignment posted on Blackboard.

2.  Have **each** group member navigate to the group’s repository on the GitHub
    website, click the green `Clone or download` button, and click **Download
    ZIP**. Rename the zipfile to `Team#_hw2c.zip` (replace `#` with your team’s
    number) and upload it to the [Moodle](https://moodle.xsede.org) site.

## Source code

You will start with an OpenMP and MPI implementation that is unacceptably
inefficient. **You are also provided with an efficient solution for the serial
implementation.** **You are highly encouraged to port this over to your OpenMP
and MPI implementations.**

  - **src/cli.cu** (DO NOT EDIT) and **vendor/CLI/CLI11.hpp** (DO NOT EDIT)
    
      - Unified interface for defining and controlling command-line options

  - **src/serial.cu** and **src/serial.cuh**
    
      - an efficient serial implementation with *O(n)* scaling.

  - **src/common.cu** and **src/common.cuh**
    
      - an implementation of common functionality, such as I/O, numerics and timing

  - **src/gpu.cu** and **src/gpu.cuh**
    
    a serial CUDA implementation, similar to the OpenMP and MPI codes you
    started from.

  - **src/gpu\_kernels.cu** and **src/gpu\_kernels.cuh**
    
    The CUDA kernels (functions) used in `gpu.cu`.

  - **src/clion.cuh**
    
    CLion doesn’t natively support CUDA syntax, so this header file helps fix
    that. This header file is ignored during compilation, so it has no effect on
    performance.

  - **src/autocorrect**
    
    A pre-built binary for checking the correctness of a program by verifying
    the txt output file. This program will run at a serial *O(n)* speed and will
    give the results for the minimum and maximum distance between particles
    during the run. This program can be run on any txt output file from a
    simulation with small sized tests (*n \< 10000*).

  - **CMakeLists.txt** (DO NOT EDIT) and **src/CMakeLists.txt**
    
      - cmake configuration files for compiling your code. Compiler flags and
        the full list of source files to compile can be edited in
        **src/CMakeLists.txt**.

  - **job-bridges-gpu**
    
      - sample batch file to launch job on Bridges. Use *sbatch* to submit on
        Bridges. This file runs the `src/autocorrect` test to check for
        correctness.

  - **auto-bridges-serial**
    
      - sample batch file to launch autograder jobs on Bridges. Use *sbatch* to
        submit on Bridges. Use this file to check performance.

  - **scripts/hw2-visualize/animate.py**
    
      - A Python script for animating the particle trajectories generated by the
        simulation. Requires a recent version of Anaconda to be installed.
        Imagemagick must also be installed to generate animated gif files.
        `ffmpeg` must also be installed to render mp4 movie files. Look in
        **job-bridges-serial** for an example on how to generate the animations.

  - **scripts/hw2-autograder/hw2c\_autograder.py**
    
      - A Python script for grading your code. Requires a recent version of
        Anaconda to be installed. Look in **auto-bridges-openmp16** and
        **auto-bridges-gpu** for an example on how to run the autograder.

  - **scripts/get-cuda-sm/get\_cuda\_sm.sh**
    
      - A small script that, if it runs successfully on your setup, will print
        out the compute capability of your GPU.

## Logging in to Bridges

The easiest way to access the machines is to login directly with your own ssh
client to **login.xsede.org** and from there **gsissh** into the correct
machine. You need to set up two-factor authentication with Duo in order to use
the single sign-on hub. More information on is available here on the [single
login system](https://portal.xsede.org/web/xup/single-sign-on-hub).

An example of logging on to XSEDE would be to first connect to the single
sign-on hub:

    ssh XSEDE-username@login.xsede.org

Enter your password and complete the 2-factor authentication request. Then, run
the following to hop over to Bridges:

    gsissh bridges

Another way to login to XSEDE resources is via the [Accounts
tab](https://portal.xsede.org/group/xup/accounts) in XSEDE User Portal. To reach this page login to XSEDE
User Portal, navigate to MY XSEDE tab and from there select the Accounts page.
All machines you have access to will have a **login** option next to them that
will launch OpenSSH via a Java Applet.

Please be aware that most XSEDE resources are not available via direct ssh and
all requests must go through the XSEDE single login system.

To clone the files from your Github copy, use the following command:

    git clone git@github.com:mason-sp19-csi-702-003/homework-2c-YOURTEAMNAME.git

## Compiling the code

Instead of a regular Makefile, compilation is handled via CMake. For your
convenience, a simple script file named `make.sh` is included that will
automatically compile your code on Bridges. From the root directory of this
repository, simply run:

    ./make.sh

Your code should compile, and the compiled binary will be placed into a folder
called `bin/`. Running this after you make a change to the code will recompile
the changed file and update the binary.

If you are developing on your local computer, you can compile the code by
running:

    ./make.sh nomodule

The virtual machine is not configured for this assignment even if you have a
CUDA-enabled GPU (I am not sure it’s even possible for CUDA code to run in a
VirtualBox environment), so you’ll need to have all the relevant libraries
installed and run CMake manually. If you are unfamiliar with how to use CMake,
you can open `make.sh` with your text editor to get an idea of how to use it.

## Submitting and running the code

The jobs queue on Bridges is managed via the [SLURM scheduler](https://slurm.schedmd.com/documentation.html).
To submit a job, use the `sbatch` command like so:

    sbatch job-bridges-gpu

To check the status of your running jobs you can use the following command:

    squeue -u $USER

Append a `-l` flag will print additional information about the running jobs. If
you want even more information, consider using the `sacct` command, for example:

    sacct -j $JOBID --format JobID,ReqMem,MaxRSS,TotalCPU,State

where `$JOBID` is the ID number of the job.

If you want to cancel a job, run:

    scancel $JOBID

If you would like to receive emails for job submissions add the following lines
to the submission scripts. This sometimes helps tracking down issues.

    #SBATCH --mail-type=ALL
    #SBATCH --mail-user=youremailaddress

For more details on SLURM commands please see [Bridges documentation
page](https://portal.xsede.org/psc-bridges#jobs).

Finally, there is an interactive mode that allows you to request computing
nodes, but maintain a command line. This is ideal for prototyping and debugging
purposes. To activate this mode, type

    interact -p GPU-shared -N 1 --gres=gpu:k80:1 --ntasks-per-node=1

The `interact.sh` script included in the repository provides you with a reminder
on how to activate the interactive session. For additional information, [read
the documentation on interactive sessions](https://portal.xsede.org/psc-bridges#jobs:interactive).

## CLI interface

When testing the code on a local computer or within an interactive session on
Bridges, you will use a simple command-line interface to launch the simulation.
For Part C, there are two runtime modes: “serial” and “gpu”. The “serial” mode
is the default.

Running `./bin/particles -h` will bring up the help:

    Usage: ./bin/particles [OPTIONS] [mode]
    
    Positionals:
      mode TEXT in {gpu,serial}
    
    Particle simulation run modes
      serial:   (default) Serial version of simulation.
      gpu:      GPU/CUDA version of simulation.
    
    Options:
      -h,--help                   Print this help message and exit
      -n INT=1000                 Set the number of particles
      -o,--output TEXT            Specify the output file name
      -s,--summary TEXT           Specify the summary file name

The most important options are `-n`, `-o`, and `-s`. The `-n` option lets you
control the number of particles in the simulation. The `-o` option lets you
output a history of the particle positions to a file, which can be used to
generate an animated gif or mp4 file. The `-s` option lets you save the amount
of time it takes to run a simulation for a given number of particles to a file.
If the file exists, then new benchmark results are appended to the end of the
file. The summary file will be used to compute your code grade.

For example, to run a particle simulation with 2000 particles that outputs the
benchmark summary data to a file named `serial_summary.txt`, you would run:

    ./bin/particles -n 2000 -s serial_summary.txt

To run the same benchmark on the GPU, you would run:

    ./bin/particles -n 2000 -s gpu_summary.txt gpu

## File transfer

When copying files to and from Bridges, [you can use `scp` in conjunction with
`data.bridges.psc.edu`](https://portal.xsede.org/psc-bridges#transfer:tfa) to avoid having to copy your files to
Single Site Login node first. **This will work with the Two-Factor
Authentication setup.** Try running the following to copy files directly to
Bridges:

    scp -P 2222 myfile XSEDE-username@data.bridges.psc.edu:/path/to/file

To copy from Bridges:

    scp -P 2222 XSEDE-username@data.bridges.psc.edu:/path/to/file myfile

## Optional: Other improvements

As always start on these only if you are happy with your current implementation
for the GPU code.

  - Creating a Hybrid that uses CUDA per node and MPI between nodes

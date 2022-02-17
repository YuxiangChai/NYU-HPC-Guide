# NYU HPC Guide

By Yuxiang Chai

For more information, please check the [NYU HPC official website](https://sites.google.com/nyu.edu/nyu-hpc) or email **hpc@nyu.edu**

**Note**

In this file, `<net-id>` means your net ID and you should replace it with something like `bs1234`. And `/scratch/<net-id>` should be something like `/scratch/bs1234`

## Contents

- [Prerequisite](#prerequisite)
- [Setup](#setup)
    - [Use VS Code](#use-vs-code)
    - [Use Terminal](#use-terminal)
- [Intro to Greene](#intro-to-greene)
    - [Storage System](#storage-system)
    - [GPU Resources](#gpu-resources)
- [Environment Setup](#environment-setup)
    - [Venv](#venv)
    - [Conda](#conda)
- [Submit a Job](#submit-a-job)
    - [SBATCH](#sbatch)
    - [SRUN](#srun)
    - [OOD](#ood)

## Prerequisite

- Apply for an account, through course instructor or other staff
- NYU VPN (Recommend)
- VS Code (Recommend)

## Setup

### Use VS Code

- Connect to NYU VPN
- Open VS Code and enable `remote ssh` extension
- Use `remote ssh` extension to connect to `<net-id>@greene.hpc.nyu.edu`
- Open your folder. (for most time, use `/scratch/<net-id>` instead of `/home/<net-id>`, explained later)

### Use Terminal 

(need knowledge of vim and other command line tools)

- ssh to gateway
```{bash}
$ ssh <net-id>@gw.hpc.nyu.edu
```
- from gateway, ssh to greene
```{bash}
$ ssh <net-id>@greene.hpc.nyu.edu
```

**OR**

- Connect to NYU VPN
- Directly ssh to greene
```{bash}
$ ssh <net-id>@greene.hpc.nyu.edu
```

## Intro to Greene

### Storage System

Basically we use three systems.

- `/home/<net-id>`, or $HOME, which is for small files. Not flushed
- `/scratch/<net-id>`, or $SCRATCH, which is for large files. **Files not accessed for 60 days will get flushed.**
- `/archive/<net-id>`, or $ARCHIVE, which is for long-term files. Not flushed.

I recommend to use `/scratch` for most projects, and try not to encounter the disk space problem.

### GPU resources

For most cases, we use HPC for GPU resources. And Greene provides two types of GPU: **RTX 8000** and **V100**

From my experience, RTX 8000 wait-time is shorter than V100 wait-time, though RTX8000 may take longer time to train models.

## Environment Setup

I only use Python on HPC so this guide will only cover Python environment. Basically I use two virtual environments, and one is from `venv` and the other one is from `conda`. I will introduce both of them.

### venv

`venv` is a built-in Python virtual environment command. It's easy to use but also has some disadvantages. 

Pros:

- No need to mess with conda environment.
- Easy to use when submitting a job.
- Fit for most popular packages.

Cons:

- It cannot create an environment with specific Python version, and it can only create an environment with current Python version.
- For some packages, it's hard to install with `pip` and may encounter some problems.

To use `venv`,

- First we need to load a Python module.
```{bash}
$ module purge    # purge everything
$ module avail python    # list all python available versions, right now only 3.8.6 is available
$ module load python/intel/3.8.6    # load a python module
```
- Then we can create our own virtual environment.
```{bash}
$ cd /scratch/<net-id>    # or cd $SCRATCH
$ python -m venv myenv    # create a folder(environment) under the current directory. You can replace 'myenv' with other names
$ source myenv/bin/activate    # activate the environment you created
(myenv) $ python --version    # now we can see myenv is activated and shown at the front
```
- Install packages you need.
```{bash}
(myenv) $ pip install numpy
```
- Submit a job (in the section [Submit a job](#submit-a-job))

### Conda

`conda` is a widely used environment on all platforms.

Pros:

- Packages more than Python packages
- Specify Python version

Cons:

- sometimes will get messed up due to dependency conflicts
- more steps to setup
- more steps to use when submitting a job

To use `conda`:

- First we need to load anaconda module.
```{bash}
$ module purge    # purge everything
$ module avail anaconda    # list all anaconda available versions, the latest should be anaconda3/2020.07
$ module load anaconda3/2020.07    # load the module
```
- Then we use anaconda to create an environment.
```{bash}
$ conda init bash    # initialize the bash
$ source ~/.bashrc    # to reload the bash and make conda work
(base) $ conda create -p /scratch/<net-id>/env39 python=3.9    # we can specify python version when creating the environment, you can replace the 'env39' with other names
(base) $ conda activate /scratch/<net-id>/env39/    # activate the environment env39
(env39) $ conda list    # this command will show the installed packages in env39
```

## Submit a job

Three ways to run a job. 

- SBATCH
- SRUN
- OOD

### SBATCH

This is the most common way to run a job.

1. First we create a file called `script.sbatch` (the name doesn't matter)
```{bash}
$ touch script.sbatch
```
2. Then in the file, we need to add some sbatch commands.

**For `venv`**

```{txt}
#!/bin/bash
#
#SBATCH --job-name=myjob
#SBATCH --output=myjob.out
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=8GB
#SBATCH --time=2:00:00
#SBATCH --gres=gpu:1
#SBATCH --mail-type=END
#SBATCH --mail-user=<net-id>@nyu.edu

module purge;
cd /scratch/<net-id>;
source myenv/bin/activate;
cd project;
python train.py --optim sgd;
```

**For `conda`** (details [here](https://sites.google.com/nyu.edu/nyu-hpc/hpc-systems/greene/software/conda-environments-python-r?authuser=0))
```{txt}
#!/bin/bash
#
#SBATCH --job-name=myjob
#SBATCH --output=myjob.out
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=8GB
#SBATCH --time=2:00:00
#SBATCH --gres=gpu:1
#SBATCH --mail-type=END
#SBATCH --mail-user=<net-id>@nyu.edu

module purge;
module load anaconda3/2020.07;
source /share/apps/anaconda3/2020.07/etc/profile.d/conda.sh;
conda activate /scratch/<net-id>/env39/;
export PATH=/scratch/<net-id>/env39/bin:$PATH;
cd /scratch/<net-id>/project;
python train.py --optim sgd;
```

**SBATCH arguments**

- `job-name`: name of job
- `output`: console output will be put in this file
- `nodes`: how many nodes you need, usually 1, unless parallel
- `ntasks-per-node`: how many tasks per node, usually 1, unless parallel
- `cpus-per-task`: how many cpus per task, usually 1
- `mem`: memory size you need
- `time`: set time so HPC will not kill the job after 1 hour. HPC will kill the job after the time you set
- `gres`: gpu usage. can be `gpu:1`, `gpu:2`(for parallel), `gpu:rtx8000:1`, `gpu:v100:2` ...
- `mail-type` and `mail-user`: send an email when the job end or start (if you set the mail-type to start)

3. Submit the job (see more [here](https://sites.google.com/nyu.edu/nyu-hpc/training-support/general-hpc-topics/slurm-submitting-jobs?authuser=0#h.p_ID_66))

```{bash}
$ sbatch script.sbatch
```

4. other commands (see more [here](https://sites.google.com/nyu.edu/nyu-hpc/training-support/general-hpc-topics/slurm-main-commands?authuser=0))

```{bash}
$ squeue -u <net-id>    # to check the status of your jobs
$ scancel <job-id>    # to cancel the job
```

### SRUN

`srun` is to start a interactive bash job. I usually use this to test my programs and then use `sbatch` to train the whole job.

```{bash}
$ srun --mem=8GB --time=2:00:00 --gres=gpu:rtx8000:1 --pty /bin/bash
```

command line arguments are the same as SBATCH arguments listed above.

### OOD 

OOD (Open OnDemand) is the GUI tools for HPC. Log into [https://ood.hpc.nyu.edu](https://ood.hpc.nyu.edu) and select **Interactive Apps**, where you can find many tools including Jupyter Notebook. 

Also we can easily use `conda` environments and `venv` environments in OOD. All you need to do is to activate the environment and install `ipykernel` and register it. You can find tutorials online.

---
title: Slurm Job Management
author: Center for Advanced Research Computing <br> University of Southern California
date: 2021-06-04
---


## Outline

1 - Overview  
2 - Cluster info  
3 - Submitting jobs  
4 - Monitoring jobs  
5 - Other commands  
6 - Job dependencies  
7 - Job arrays


## 1 - Overview


## What is Slurm?

- Open-source management and job scheduling system for Linux computing clusters
- Three main functions:
  - Allocates access to resources (compute nodes)
  - Provides a framework to run and monitor jobs on allocated nodes
  - Manages a job queue for competing resource requests
- Specific configuration for CARC clusters
- Official documentation: [https://slurm.schedmd.com](https://slurm.schedmd.com)


## Overview of Slurm commands

| Category | Command | Purpose |
|---|---|---|
| Cluster info    | sinfo    | Display compute partition and node information |
| Submitting jobs | sbatch   | Submit a job script for remote execution |
|                 | srun     | Launch parallel tasks (job steps) for MPI jobs |
|                 | salloc   | Allocate resources for an interactive job |
|                 | squeue   | Display status of jobs and job steps |
|                 | sprio    | Display job priority information |
|                 | scancel  | Cancel pending or running jobs |
| Monitoring jobs | sstat    | Display status information for running jobs |
|                 | sacct    | Display accounting information for jobs |
|                 | seff     | Display job efficiency information for past jobs |
| Other           | scontrol | Display or modify Slurm configuration and state |


## Typical batch job workflow

1. Create job script
2. Submit job script with `sbatch`
3. Check job status with `squeue`
4. When job completes, check log file
5. If job failed, modify job and resubmit (back to step 1)
    - Check job information with `sacct` or `seff` if needed
6. If job succeeded, check job information with `sacct` or `seff`
    - If possible, use less resources next time for similar job


## 2 - Cluster info


## sinfo

- Display compute partition and node information  
- `sinfo --help`  
- [https://slurm.schedmd.com/sinfo.html](https://slurm.schedmd.com/sinfo.html)
- Default output lists information by partition and node status:

```
$ sinfo
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
debug        up      30:00      1    mix a02-26
debug        up      30:00      5   idle e05-[42,76,78,80],e22-13
epyc-64      up 2-00:00:00      1   drng b22-21
epyc-64      up 2-00:00:00      1    mix b22-31
epyc-64      up 2-00:00:00     30  alloc b22-[01-20,22-30,32]
main*        up 2-00:00:00      5 drain* d17-[40-44]
...
```


## sinfo (continued)

- Useful options include `--Node`, `--partition`, and `--states`
- Many formatting options with `--format` or `--Format`
  - Can also use SINFO_FORMAT environment variable to set
  - `export SINFO_FORMAT=...`
  - Add to `~/.bashrc` to automatically set when logging in
- `sinfo2` is alias for listing information by partition, node type, and node status


## sinfo example

```
$ sinfo -lNp debug
Thu Mar 11 14:14:02 2021
NODELIST   NODES PARTITION       STATE CPUS    S:C:T MEMORY TMP_DISK WEIGHT AVAIL_FE REASON
a02-26         1     debug       mixed 16      2:8:1  63000        0      1 xeon-264 none
e05-42         1     debug        idle 16      2:8:1  63400        0      1 xeon-265 none
e05-76         1     debug        idle 16      2:8:1  63400        0      1 xeon-265 none
e05-78         1     debug        idle 16      2:8:1  63400        0      1 xeon-265 none
e05-80         1     debug        idle 16      2:8:1  63400        0      1 xeon-265 none
e22-13         1     debug        idle 20     2:10:1 128000        0      1 xeon-264 none
```


## Codes for common node states

- ALLOCATED
  - The node has been allocated to one or more jobs
- DOWN
  - The node is unavailable for use
- DRAINING
  - The node is currently executing a job, but will not be allocated additional jobs
- IDLE
  - The node is not allocated to any jobs and is available for use
- MAINT
  - The node is currently in a reservation with a flag value of "maintenance"
- MIXED
  - The node has some of its CPUs ALLOCATED while others are IDLE
- RESERVED
  - The node is in an advanced reservation and not generally available


## sinfo example 2

```
$ sinfo --states=idle
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
debug        up      30:00      5   idle e05-[42,76,78,80],e22-13
epyc-64      up 2-00:00:00      0    n/a
main*        up 2-00:00:00      0    n/a
oneweek      up 7-00:00:00     34   idle e01-[48,52,60,62],e02-[41-46,48-63,65-72]
largemem     up 7-00:00:00      0    n/a
```


## Exercise 1

Display info for the epyc-64 partition


## 3 - Submitting jobs


## Three commands for submitting jobs

- For batch jobs, use `sbatch`
- For parallel (MPI) tasks (job steps) within batch jobs, use `srun`
- For interactive jobs, use `salloc`


## sbatch

- Submit a job script for remote execution
- A job script is a special type of Bash script
- `sbatch --help`
- [https://slurm.schedmd.com/sbatch.html](https://slurm.schedmd.com/sbatch.html)

```
$ sbatch jl.job
Submitted batch job 3353548
```


## Example job script for multi-threaded job

```
#!/bin/bash

#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --mem=32GB
#SBATCH --time=2:00:00
#SBATCH --account=<account_id>

module purge
module load gcc/8.3.0
module load julia/1.5.2

julia --threads $SLURM_CPUS_PER_TASK script.jl
```

- Generic structure:
  - Specify SBATCH options (resource requests)
  - Load software and set up environment
  - Run commands


## Commonly used sbatch options

| Option | Description |
|---|---|
| `--nodes=<number>`               | Number of nodes to use |
| `--ntasks=<number>`              | Number of processes to run |
| `--cpus-per-task=<number>`       | Number of cores per task |
| `--mem=<number>`                 | Total memory (per node) |
| `--mem-per-cpu=<number>`         | Memory per processor core |
| `--constraint=<attribute>`       | Node property to request (e.g., `xeon-4116`) |
| `--partition=<partition_name>`   | Request nodes on specified partition |
| `--time=<D-HH:MM:SS>`            | Maximum run time |
| `--account=<account_id>`         | Account to charge resources to |
| `--mail-type=<value>`            | E-mail notifications (e.g., `begin|end|fail|all`) |
| `--mail-user=<address>`          | E-mail address |
| `--output=<filename>`            | File for standard output |
| `--error=<filename>`             | File for standard error |


## Default values for sbatch options

| Option | Default value |
|---|---|
| `--nodes`          | 1 |
| `--ntasks`         | 1 |
| `--cpus-per-task`  | 1 |
| `--mem-per-cpu`    | 2GB |
| `--time`           | 1:00:00 |
| `--partition`      | main |
| `--account`        | default project account |


## sbatch notes

- Shell environment is transferred to job when submitted
- Use `module purge` to clear modules
- Use `--mem=0` to request all available memory on a node
- For submitting lots of similar jobs, use job arrays
- For submitting lots of short-running jobs, pack them together in one job


## Exercise 2

Submit the following job script:

```
#!/bin/bash

#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem=1GB
#SBATCH --time=0:10:00
#SBATCH --account=<account_id>

module purge

echo "Hello world"
sleep 280
```

## Example job script for GPU job

```
#!/bin/bash

#SBATCH --gres=gpu:p100:1
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=16GB
#SBATCH --time=1:00:00
#SBATCH --account=<account_id>

module purge
module load gcc/8.3.0
module load cuda/10.1.243

./program
```

- Add option: `#SBATCH --gres=gpu:<gpu_type>:<number>`
- [CARC User Guide for Using GPUs](https://carc.usc.edu/user-information/user-guides/software-and-programming/using-gpus)


## Job output files

- By default, output log files are named `slurm-<jobid>.out` and saved to the submit directory
- By default, both output and error messages are printed to the same output file
- Use `--output` and/or `--error` options to customize filenames or split output
- Filename patterns can be used (e.g., %x = job name &rarr; `#SBATCH --output=%x.out`)


## Current job limits on Discovery

| Partition | Maximum run time | Maximum concurrent CPUs | Maximum concurrent GPUs | Maximum number of jobs queued | Maximum number of jobs running |
|---|---|---|---|---|---|
| main     |  48 hours   | 1,200 | 36  | 5,000 | 500 |
| epyc-64  |  48 hours   | 1,200 | n/a | 5,000 | 500 |
| oneweek  | 168 hours   | 208   | n/a | 50    | 50  |
| largemem | 168 hours   | 120   | n/a | 10    | 2   |
| debug    | 30  minutes | 48    | 4   | 5     | 5   |

*By default, Endeavour condo partitions will inherit the main partition limits but with a two-week maximum run time*  
*Custom limits can be requested for Endeavour condo partitions*


## Environment variables for sbatch

- Output variables can be used in job or application scripts
- Some examples:

| Variable | Description |
|---|---|
| SLURM_JOB_ID         | The ID of the job allocation |
| SLURM_JOB_NODELIST   | List of nodes allocated to the job |
| SLURM_JOB_NUM_NODES  | Total number of nodes in the job's resource allocation |
| SLURM_NTASKS         | Number of tasks requested |
| SLURM_CPUS_PER_TASK  | Number of CPUs requested per task |
| SLURM_SUBMIT_DIR     | The directory from which `sbatch` was invoked |
| SLURM_ARRAY_TASK_ID  | Job array ID (index) number |


## Example job script with Slurm variables

```
#!/bin/bash

#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32GB
#SBATCH --time=2:00:00
#SBATCH --account=<account_id>

module purge
module load gcc/8.3.0
module load julia/1.5.2

echo "Job ID: $SLURM_JOB_ID"
echo "Nodelist: $SLURM_JOB_NODELIST"

cd $SLURM_SUBMIT_DIR/scripts

julia --threads $SLURM_CPUS_PER_TASK script.jl
```


## srun

- Launch parallel tasks (job steps) for MPI jobs
- `srun --help`
- [https://slurm.schedmd.com/srun.html](https://slurm.schedmd.com/srun.html)
- [CARC User Guide for Using MPI](https://carc.usc.edu/user-information/user-guides/software-and-programming/mpi)


## Example job script for MPI job

```
#!/bin/bash

#SBATCH --nodes=6
#SBATCH --ntasks=144
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=3GB
#SBATCH --time=24:00:00
#SBATCH --constraint=xeon-4116
#SBATCH --exclusive
#SBATCH --account=<account_id>

module purge
module load gcc/8.3.0
module load openmpi/4.0.2
module load pmix/3.1.3

ulimit -s unlimited

srun --mpi=pmix_v2 -n $SLURM_NTASKS ./mpi_program.x
```


## salloc

- Allocate resources for an interactive job
- Similar options to `sbatch`
- `salloc --help`
- [https://slurm.schedmd.com/salloc.html](https://slurm.schedmd.com/salloc.html)

```
ttrojan@discovery1:~$ salloc --time=2:00:00 --cpus-per-task=8
salloc: Pending job allocation 3353609
salloc: job 3353609 queued and waiting for resources
salloc: job 3353609 has been allocated resources
salloc: Granted job allocation 3353609
salloc: Waiting for resource configuration
salloc: Nodes d05-11 are ready for job

ttrojan@d05-11:~$ hostname
d05-11.hpc.usc.edu

ttrojan@d05-11:~$ exit
exit
salloc: Relinquishing job allocation 3353609

ttrojan@discovery1:~$
```


## Exercise 3

Request an interactive job on the debug partition


## Additional commands related to job submission

- To see the job queue, use `squeue`
- To check job priority, use `sprio`
- To cancel jobs, use `scancel`


## squeue

- Display status of jobs and job steps
- `squeue --help`
- [https://slurm.schedmd.com/squeue.html](https://slurm.schedmd.com/squeue.html)

```
$ squeue -u ttrojan
  JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
3351279      main model4.j  ttrojan PD       0:00      1 (Resources)
3351155      main model1.j  ttrojan  R   19:49:52      1 d14-13
3351270      main model2.j  ttrojan  R    9:42:42      1 d11-35
3351276      main model3.j  ttrojan  R    9:45:12      1 d06-27
```


## Common codes for job state

- PD PENDING
  - Job is awaiting resource allocation
- R RUNNING
  - Job currently has an allocation
- CD COMPLETED
  - Job has terminated all processes on all nodes with an exit code of zero
- CG COMPLETING
  - Job is in the process of completing. Some processes on some nodes may still be active
- CA CANCELLED
  - Job was explicitly cancelled by the user or system administrator. The job may or may not have been initiated


## Common codes for pending reason

- Resources
  - The job is waiting for resources to become available
- Priority
  - One or more higher priority jobs exist for this partition or advanced reservation
- ReqNodeNotAvail
  - Some node specifically required by the job is not currently available
- QOSMaxCpuPerUserLimit
  - The job has reached the maximum CPU per user limit
- QOSMaxGRESPerUser
  - The job has reached the maximum GPU per user limit
- AssocGrpCPUMinutesLimit
  - The project account has run out of CPU time
- InvalidAccount
  - The job's account is invalid


## squeue notes

- Useful options include `--start` and `--partition`
- Many formatting options with `--format` or `--Format`
  - Can also use SQUEUE_FORMAT environment variable to set
  - `export SQUEUE_FORMAT=...`
  - Add to `~/.bashrc` to automatically set when logging in
- Create shorter alias
  - `alias myq="squeue -u $USER"`
  - Add to `~/.bashrc` to automatically set when logging in


## Job priorities

- Based on a fairshare algorithm and job age
- Fairshare values depend on a number of factors:
  - Number of jobs submitted
  - Resources used
  - Project account activity
- [https://slurm.schedmd.com/fair_tree.html](https://slurm.schedmd.com/fair_tree.html)


## sprio

- Display job priority information
- `sprio --help`
- Can be difficult to interpret
- If normalized, a priority value closer to 1 means a higher priority
- [https://slurm.schedmd.com/sprio.html](https://slurm.schedmd.com/sprio.html)

```
$ sprio -j 3351270
  JOBID PARTITION   PRIORITY       SITE        AGE  FAIRSHARE    JOBSIZE  PARTITION        QOS                 TRES
3351270 main            1264          0         70          5         17       1000          0 cpu=13,mem=22,gres/g

$ sprio -j 3351270 -n
  JOBID PARTITION PRIORITY   AGE        FAIRSHARE  JOBSIZE    PARTITION  QOS        TRES
3351270 main      0.00000029 0.0700661  0.0005269  0.0173378  1.0000000  0.0000000  cpu=0.01,mem=0.01,gr
```


## scancel

- Cancel pending or running jobs
- `scancel --help`
- [https://slurm.schedmd.com/scancel.html](https://slurm.schedmd.com/scancel.html)

```
scancel 3358600

scancel -u ttrojan
```


## 4 - Monitoring jobs



## Three commands for monitoring jobs

- To view job status for currently running job, use `sstat`
- To view information about current or past jobs, use `sacct`
- To view efficiency information about past jobs, use `seff`


## sstat

- Display status information for running jobs
- `sstat --help`
- [https://slurm.schedmd.com/sstat.html](https://slurm.schedmd.com/sstat.html)

```
sstat -j <jobid>

sstat -j <jobid> --format=JobID,MaxRSS,AveCPUFreq,MaxDiskRead,MaxDiskWrite
```


## sacct

- Display accounting information for jobs
- `sacct --help`
- [https://slurm.schedmd.com/sacct.html](https://slurm.schedmd.com/sacct.html)

```
sacct

sacct -j <jobid>

sacct --format=JobID,MaxRSS,AveCPUFreq,MaxDiskRead,MaxDiskWrite,State,ExitCode
```


## sacct notes

- By default, only displays jobs from past day
- Some useful options are `--starttime`, `--endtime`, `--brief`, and `--state`
- Many formatting options with `--format`
  - Can also use SACCT_FORMAT environment variable to set
  - `export SACCT_FORMAT=...`
  - Add to `~/.bashrc` to automatically set when logging in


## Job exit codes

- Exit status, 0-255
- [https://slurm.schedmd.com/job_exit_code.html](https://slurm.schedmd.com/job_exit_code.html)
- 0 &rarr; success, completed
- non-zero &rarr; failure
- Codes 1-127 indicate error in job
- Exit codes 129-255 indicate jobs terminated by Unix signals
  - For these, subtract 128 from the number and match to signal code
  - Enter `kill -l` to list signal codes
  - Enter `man signal` for more information


## seff

- Display job efficiency information for past jobs (CPU and memory use)
- Use to optimize resource requests

```
$ seff 3043889
Job ID: 3043889
Cluster: discovery
User/Group: ttrojan/ttrojan
State: COMPLETED (exit code 0)
Cores: 1
CPU Utilized: 00:05:16
CPU Efficiency: 21.81% of 00:24:09 core-walltime
Job Wall-clock time: 00:24:09
Memory Utilized: 4.64 GB
Memory Efficiency: 6.63% of 70.00 GB
```


## Exercise 4

Check efficiency of previously submitted job


## 5 - Other commands


## scontrol

- Display or modify Slurm configuration and state
- Mostly for admins, some commands for users
- [https://slurm.schedmd.com/scontrol.html](https://slurm.schedmd.com/scontrol.html)
- Some examples:

```
scontrol show partition <partition>

scontrol show node <nodeid>

scontrol show job <jobid>

scontrol hold <jobid>

scontrol release <jobid>
```


## 6 - Job dependencies


## Why use a job dependency?

- If a job depends on another job
- Defer the start of a job until the specified dependencies have been satisfied
- Can be useful for workflows
- Also useful for checkpointing and splitting up long-running jobs


## Setting up a job dependency

- Add `#SBATCH --dependency=<type>` to job script
- Or use `sbatch --dependency=<type> my.job`
- Different dependency types available
- Need job IDs (use `squeue` or `sacct`)


## Commonly used job dependency types

`--dependency=afterok:<job_id>[:<job_id>...]`

This job can begin execution after the specified jobs have successfully executed (ran to completion with an exit code of 0)

`--dependency=afternotok:<job_id>[:<job_id>...]`

This job can begin execution after the specified jobs have terminated in some failed state (non-0 exit code, node failure, timed out, etc.)

`--dependency=after:<job_id>[[+time][:<job_id>[+time]...]]`

After the specified jobs start or are cancelled and time in minutes from job start or cancellation happens, this job can begin execution. If no time is given, then there is no delay after start or cancellation

`--dependency=afterany:<job_id>[:<job_id>...]`

This job can begin execution after the specified jobs have terminated


## Job dependency example

```
$ sbatch jl.job
Submitted batch job 3353548

$ sbatch --dependency=afterok:3353548 jl2.job
Submitted batch job 3353549

$ squeue -u ttrojan
  JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
3353549      main  jl2.job  ttrojan PD       0:00      1 (Dependency)
3353548      main   jl.job  ttrojan  R       0:32      1 b22-31
```


## 7 - Job arrays


## Why use a job array?

- For submitting and managing collections of similar jobs quickly and easily
- Some examples:
  - Varying simulation or model parameters
  - Running the same statistical models on different datasets
- [https://slurm.schedmd.com/job_array.html](https://slurm.schedmd.com/job_array.html)


## Setting up a job array

- Add `#SBATCH --array=<index>` option to job script
- Each job task will use the same resources requested
- Modify job or application script to use array index
- Multiple methods are possible


## Job array example

```
#!/bin/bash

#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=16GB
#SBATCH --time=2:00:00
#SBATCH --array=1-3

module purge
module load gcc/8.3.0
module load openblas/0.3.8
module load r/4.0.0

echo "Task ID: $SLURM_ARRAY_TASK_ID"

Rscript --vanilla script.R
```


## Job array example (continued)

```
# R script to process and model data

library(data.table)

files <- list.files("./data", full.names = TRUE)
task <- as.numeric(Sys.getenv("SLURM_ARRAY_TASK_ID"))
file <- files[task]
file

data <- fread(file)

summary(data)

...
```


## Job array example (continued)

```
$ sbatch array.job
Submitted batch job 3355483

$ squeue -u ttrojan
    JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
3355483_1      main array.jo  ttrojan  R       0:05      1 d05-35
3355483_2      main array.jo  ttrojan  R       0:05      1 e16-15
3355483_3      main array.jo  ttrojan  R       0:05      1 d18-29
```


## Additional resources

- [Official Slurm documentation](https://slurm.schedmd.com/)
- [CARC User Guide for Running Jobs](https://carc.usc.edu/user-information/user-guides/hpc-basics/running-jobs)
- [CARC Slurm Job Script Templates](https://carc.usc.edu/user-information/user-guides/hpc-basics/slurm-templates)
- Video learning: [Submitting jobs](https://carc.usc.edu/education-and-outreach/video-learning/submitting-jobs)


## Getting help

- [Submit a support ticket](https://carc.usc.edu/user-information/ticket-submission)
- [User Forum](https://hpc-discourse.usc.edu/)
- Office Hours
  - Every Tuesday 2:30-5pm (currently via Zoom)
  - Register [here](https://carc.usc.edu/news-and-events/events)

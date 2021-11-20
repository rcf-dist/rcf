---
title: "User Guide"
---
Once the user is able to log into purpleJeans (Connect to Resources), upload and access their files on the cluster (Data Storage and Transfer), and upload software tools using the (Software) module system, you are ready to scheduling access to the compute cluster to perform calculations.

# Introduction
The purpleJeans compute cluster is a shared resource used by the entire RCF community. Sharing computational resources creates unique challenges:

* Jobs must be managed (job scheduler) equally for all users.
* Resource consumption must be logged.
* Access to resources must be controlled.

The purpleJeans compute cluster uses a scheduler to handle requests for access to compute resources. These requests are called jobs. In particular, Slurm's resource manager is used to schedule jobs, as well as interactive access to compute nodes.

Slurm is an open source, fault tolerant, and highly scalable system for cluster management and job sheduling for large and small Linux clusters. Slurm does not require kernel modifications for its operation and is relatively self-contained. As a cluster workload manager, Slurm has three key functions. First, it gives users exclusive and / or non-exclusive access to resources (compute nodes) for a certain period of time, so that they can run jobs. Second, it provides a framework for starting, running, and monitoring jobs (usually jobs to run in parallel) on the set of allocated nodes. Finally, it arbitrates contention for resources by managing a queue (partitions) of pending jobs. Optional plug-ins can be used for resource usage accounting, advanced booking, group scheduling (sharing time for parallel jobs), backfill scheduling, topology-optimized resource selection, boundaries of resources per user or account and sophisticated multi-factor job prioritization algorithms.

Below is the essential information you need to know to get started on purpleJeans. For more detailed information on running specialized compute jobs, see Running Jobs on purpleJeans (link).

# Computing nodes types
The purpleJeans compute cluster consists of compute nodes with different architectures and configurations. A partition is a collection of compute nodes that all have the same or similar architecture and configuration. Currently, purpleJeans has the following partitions:

| Resources           | Partitions | Nodes | CPU/Node                                             | Core/Node | Memory/Node   | Notes                                                                       |
|-------------------|------------|------|------------------------------------------------------|-----------|----------------|-------------------------------------------------------------------------------------------------------------------------|
| purpleJeans (2019)| xhicpu     |    4 | 2 CPU Intel(R) Xeon(R) Xeon 16-Core 5218 2,3Ghz 22MB |        32 |         192 GB | Cluster dedicato alla ricerca nell’ambito del machine learning e big data. Mellanox CX4 VPI SinglePort FDR IB 56Gb/s x16. |
|                   | xgpu       |    4 | 2 CPU Intel(R) Xeon(R) Xeon 16-Core 5218 2,3Ghz 22MB |        32 |         192 GB | 4 GPU  (NVLINK) NVIDIA Tesla V100 32GB SXM2. Infiniband Mellanox CX4 VPI SinglePort FDR IB 56Gb/s x16. |

It is possible to have a summary of the partitions on purpleJeans using the sinfo command:

```sh
$ sinfo shared
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
xhicpu*      up   12:00:00      4  alloc wnode[01-04]
xgpu         up    6:00:00      1    mix gnode02
xgpu         up    6:00:00      1  alloc gnode03
xgpu         up    6:00:00      2   idle gnode[01,04]
```

In the summary obtained with sinfo shared, the “NODES” column gives the total number of nodes in each partition. This summary also lists the partitions reserved for use by certain laboratories.

# Interactive jobs
After submitting an interactive job on purpleJeans, Slurm will connect you to a compute node and load an interactive shell environment for use on that compute node. This interactive session will persist until disconnected from the compute node or until the maximum time required is reached. The default required time is 2 hours.

## srun
An alternative to sinteractive is the srun command:

```sh
$ srun --pty bash
srun: job 2808 queued and waiting for resources
```

Unlike sinteractive, this command does not set X11 forward, which means that it is not possible to display graphics using srun. Both the srun and sinteractive commands have the same options. During the execution of an interactive task, you have access to your home and consequently to all installed libraries. An example of using srun with various options is as follows:

```sh
$ srun --ntasks=1 \
> --nodes=1 \
> --partition=xhicpu \
> --cpus-per-task=8 \
> --mem=4000 \
> --pty \
> bash
```

| Option             | Description                                                 |
|--------------------|-------------------------------------------------------------|
| --ntasks=1         | Only one task to perform                                    |
| --nodes=1          | Ask for one node                                      |
| --partition=xhicpu | Request xhicpu partition nodes |
| --cpus-per-task=8  | Request to allocate 8 CPU for each task                    |
| --mem=4000         | Request using a total memory of 4000 MB (4 GB)          | 
| --pty              | Launch task in terminal mode                        |
| bash               |Executable to run, in this case bash command       |

In this example, only the wnode01 node will be used, requiring a limited number of resources (44GB of memory and 8 CPUs for 1 task), this allows the parallel execution of multiple jobs (without distinction between srun and sbatch)

## Batch Job Type
The sbatch command is the command most commonly used by RCF users to request compute resources on the purpleJeans cluster. Instead of specifying all options on the command line, users typically write a “script sbatch” that contains all the commands and parameters needed to run the program on the cluster.

In a sbatch script, all Slurm parameters are declared with #SBATCH, followed by further definitions.

Here is an example of a sbatch script:

```sh
#!/bin/bash
#SBATCH --job-name=example
#SBATCH --output=example.out
#SBATCH --error=example.err
#SBATCH --time=00:10:00
#SBATCH --partition=xhicpu
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=16
#SBATCH --mem-per-cpu=2000
 
# Set stack size (to avoid warning message)
ulimit -s 10240

# Load OpenMPI module
module load ompi-4.1.0-gcc-8.3.1

# Run
mpirun ./hello-mpi
```

Below are the details of the commands:

| Option                     | Description                                                                           |
|-----------------------------|---------------------------------------------------------------------------------------|
| #SBATCH --job-name = example | Assign the name example to the job |
| #SBATCH --output = example.out | Write console output to example.out |
| #SBATCH --error = example.err | Write the error message to the file example.err |
| #SBATCH --time = 00: 10: 00 | Reserve requested resources for 10 minutes (or even less if the process ends earlier) |
| #SBATCH --partition = xhicpu | Requires compute nodes present on the xhicpu partition |
| #SBATCH --nodes = 4 | Requires 4 compute nodes |
| #SBATCH --ntasks-per-node = 16 | Requires 16 CPUs for each single node |
| #SBATCH --mem-per-cpu = 2000 | Requires 2000MB (2GB) of RAM per single CPU|

In this example, we required 4 compute nodes with 16 CPUs each. Therefore, we required a total of 64 CPUs to run our program. The last two lines of the script load the OpenMPI module and start the MPI-based executable we called hello-mpi (see MPI job).

Going on the example above, let's assume this script is stored in the current directory in a file called example.sbatch. This script is sent to the cluster using the following command:

```sh
$ sbatch ./example.sbatch
```

Many other options are available for submitting jobs using the sbatch command. For more specialized computational needs, see Running Jobs on purpleJeans (link). Also, for a complete list of available options, see the official SBATCH documentation (link: https://slurm.schedmd.com/sbatch.html).

**Please note:**
Setting the stack size is a good idea to avoid runtime warning messages.
```ulimit -s 10240```

# Temporary storage
Many applications generate temporary or intermediate files written in /tmp. (These applications can write files to /tmp even without you knowing this is happening.) This folder is typically located on a local drive or virtualized RAM disk in system memory.

- Contents in /tmp left by a user's job will not be automatically deleted before restarting the corresponding node, and therefore may affect other jobs that are subsequently run on the same node. Therefore, RCF enforces a data purge policy for files written to /tmp on compute nodes:
  
- For each running job, a folder /tmp /jobs /${SLURM_JOB_ID}, readable and writable only by the job, is created on each assigned node. Its content is only safely deleted after the job is finished (when it is successfully completed, canceled or killed).

- For all running jobs, the SLURM_TMPDIR and TMPDIR environment variables are set to /tmp/jobs/${SLURM_JOB_ID}. Whenever possible, users should write to the paths specified by these environment variables rather than explicitly using / tmp. (Most applications should already use these environment variables by default, so in many cases this doesn't require any code changes.)

- In addition to using $TMPDIR, users should also ensure that no other files are written to /tmp.

- Note that after a job ends, any folders or files directly in /tmp that belong to the sender of this job will be deleted.

- The contents of /tmp do not persist at the end of the works. RCF is not responsible for the retrieval or recovery of the data stored there. For critical outputs, save them to persistent file storage systems; see Archiving and data transfer.

**Important:**
Folders or files created by users in /tmp outside $TMPDIR are accessible by all jobs submitted by the same user and running on the node. For example, consider the case where the user has two jobs running (A and B) on the same node and job B is writing files to / tmp directly. If job A ends before job B, the contents of / tmp will also be deleted and, in some cases, job B may fail. To avoid errors, users must then write any temporary data to the secure working folder, SLURM_TMPDIR or TMPDIR.

# Job Management
The Slurm offers several command line tools to check the status of jobs and manage them. For a complete list of Slurm commands, see the Slurm man pages (link: https://slurm.schedmd.com/man_index.html). Here are some commands you might find particularly useful:
- **squeue**: find out the status of jobs submitted by you and other users.
- **sacct**: Retrieve job history and past job statistics.
- **scancel**: cancel the jobs you sent.

## Checking Job Status
Use the **squeue** command to check the status of jobs and other jobs running on purpleJeans. The simplest call lists all the jobs that are currently running or waiting in the job queue ("PENDING"), along with details about each job such as the id and number of nodes required:

```sh
$ squeue
```
Any jobs with 0:00 in the “TIME” column are still waiting in the queue.

To view only submitted jobs, use the *--user* flag

```sh
$ squeue --user=$USER
```

This command has many other useful options for querying the queue status and getting information about individual jobs. For example, to get information about all jobs waiting to run on the xhicpu partition, enter:

```sh
$ squeue --state=PENDING --partition=xhicpu
```

Alternatively, to get information about all jobs running on the xhicpu partition, type:

```sh
$ squeue --state=RUNNING --partition=xhicpu --user=$USER
```

The last column of the output tells us which nodes are allocated for each job. For example, if
show wnode01 for one of the jobs with your name, you can type:

```sh
$ ssh wnode01 
```

to access that compute node and check its progress locally.

For more information, consult the command line help by typing squeue --help or visit the official online documentation.

## Cancel Jobs
To cancel a submitted job, use the **scancel** command. This requires you to specify the id of the job you want to cancel. For example, to cancel a job with ID 1008, proceed as follows:

```sh
$ scancel 1008
```

If you are unsure which job id you want to cancel, see the “JOBID” column from running squeue --user = $ USER.

To cancel more than one job separate the ids with a comma:

```sh
$ scancel 1008,1009,1012
```

To cancel all submitted jobs that are running or waiting in queue:

```sh
$ scancel --user=$USER
```

## Parallel job steps
Jobs involving a very large number of independent computations should be combined in some way to reduce the number of jobs submitted to Slurm. Here we outline a strategy for doing this using srun. The parallel program executes the tasks simultaneously until all the tasks, which are called job steps, are completed.

```sh
#!/bin/bash
#SBATCH --job-name=parallel-tasks
#SBATCH --output=parallel-tasks.out
#SBATCH --error=parallel-tasks.err
#SBATCH --partition=xhicpu
#SBATCH --nodes=1
#SBATCH --ntasks=3
#SBATCH --mem-per-cpu=2000
 
srun -n1 --exclusive ./program1.sh &
srun -n1 --exclusive ./program2.sh &
srun -n1 --exclusive ./program3.sh &
 
wait
```

| Option | Description |
|------------------------------------|-------------|
| #SBATCH --job-name = parallel-tasks | Name the job parallel-tasks |
| #SBATCH --output = parallel-tasks.out | Write console output to parallel-tasks.out file |
| #SBATCH --error = parallel-tasks.err | Write the error message to the parallel-tasks.err file |
| #SBATCH --partition = xhicpu | Requires compute nodes present on the xhicpu partition |
| #SBATCH --nodes = 1 | Requires 1 compute node |
| #SBATCH --ntasks = 3 | Requires the execution of up to 3 parallel tasks |
| #SBATCH --mem-per-cpu = 2000 | Requires 2000MB (2GB) of RAM per single CPU |

The example wants to execute a single job in which three programs are executed in parallel that are not necessarily connected to each other. In order to perform more than a single task, the --ntasks flag must be specified, which by default is set to 1. Once specified, we can use the srun command within the sbatch file and use the & symbol at the end of each line. Each srun can have its own arguments, which must be consistent with the maximum resources required by the script. In the example, each srun requires exactly one task with the -n1 option (a single CPU), the --exclusive parameter, which is not mandatory, ensures that a separate CPU is allocated for each activity. The wait command waits for all tasks to finish, fail or be killed.

## GPU Job
The **xgpu** partition is dedicated to software that uses Graphical Processing Unit (GPU).

CUDA and OpenACC are two widely used tools for GPU-based software development. For information on compiling code with CUDA and OpenACC, consult the respective online documentation.

### Run GPU code

To submit a job to one of the GPU nodes, you need to include the following command lines in the sbatch script (srun equivalently):

```sh
#SBATCH --partition=xgpu
#SBATCH --gres=gpu:tesla:N
```

Where --gres explicitly requests the GPU resource, gpu: tesla is the specific nomenclature to use on purpleJeans, N is the number of GPUs required. Allowed settings for range N from 1 to 4. If your application is fully GPU-based, there is no need to explicitly request cores as a CPU core will be assigned by default as the master to start GPU-based compute. However, if your application is a mixed CPU-GPU, you will need to request the number of cores with –-ntasks as required by your job.
Slurm automatically sets the CUDA_VISIBLE_DEVICES environment variable with the GPU resource free at the time of the request. Do not try to manually set the CUDA_VISIBLE_DEVICES variable.

Ecco un esempio di script sbatch che alloca le risorse della GPU e carica i compilatori CUDA

```sh
#!/bin/bash
#SBATCH --job-name=gpu  
#SBATCH --output=gpu.out 
#SBATCH --error=gpu.err  
#SBATCH --time=01:00:00  
#SBATCH --nodes=1        
#SBATCH --partition=xgpu
#SBATCH --ntasks=1       
#SBATCH --gres=gpu:tesla:1     

# Set stack size (to avoid warning message)
ulimit -s 10240

module load cuda/10.1

# Add the commands needed to perform GPU processing
```

| Option | Description |
|---------------------------|-----------------------------------------------------------------------------|
| #SBATCH --job-name = gpu | Name the job gpu |
| #SBATCH --output = gpu.out | Writes console output to gpu.out |
| #SBATCH --error = gpu.err | Write the error message to the gpu.err file |
| #SBATCH --time = 01: 00: 00 | Reserve requested resources for 1 hour (or less if process ends earlier) |
| #SBATCH --nodes = 1 | Requires 1 compute node |
| #SBATCH --partition = xgpu | Requires compute nodes present on xgpu partition |
| #SBATCH --ntasks = 1 | Only one task to run |
| #SBATCH --gres = gpu: tesla: 1 | Requires only one GPU

**Important**: The "stoneware" shortcuts for using GPUs, shown in the official documentation, are not configured.

### Multiple Jobs on nodes
Running a job on a compute node does not assign that node exclusively to the user who performed the job. This is possible with a responsible request for the available computing resources. It is possible to run multiple jobs on a single node even if the job being executed belongs to another user, without interfering with it. To do this it is necessary to specify the parameters in the "sbatch scripts" or srun that allow you to reserve only the resources you need.

Let's see the following example:

```sh
#!/bin/bash
#SBATCH --job-name=limited-resources
#SBATCH --partition=xhicpu
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --mem=16000
#SBATCH --cpus-per-tasks=8

# Set stack size (to avoid warning message)
ulimit -s 10240

./program1.sh
```

In the example, you are asked to execute a job by allocating 16GB of memory and a total of 8 CPU Cores on the node that will be assigned by the scheduler. Considering that this node has 192GB of RAM and 32 CPU Cores, the node still has enough resources (176GB and 24 CPU Cores) to allow other jobs to run on it.

## Know the free resources on each single node
To know the free resources on each single node of each partition, so as to be able to allocate the necessary resources for the job you are trying to launch, you can use the sinfo command with the following options:

```sh
$ sinfo -N -o "%N %.5a %.11T %.10l %.10m %.10e %.15C" 
 
NODELIST   AVAIL     STATE  TIMELIMIT  MEMORY  FREE_MEM  CPUS(A/I/O/T)
gnode01     up       idle    6:00:00   178000    29353     0/32/0/32
gnode02     up      mixed    6:00:00   178000    14831    16/16/0/32
gnode03     up  allocated    6:00:00   178000   179125     32/0/0/32
gnode04     up       idle    6:00:00   178000    55842     0/32/0/32
wnode01     up       idle   12:00:00   178000   188149     0/32/0/32
wnode02     up  allocated   12:00:00   178000   181558     32/0/0/32
wnode03     up  allocated   12:00:00   178000   182190     32/0/0/32
wnode04     up  allocated   12:00:00   178000   180030     32/0/0/32
```

| Option | Description |                                                              
|---------|----------------------------------------------------------------------------|
| -N | Print the information for each single node |
| -o | Specify what information to display |
| % N | List of nodes |
| % .5a | Availability of the node |
| % .11T | Node state |
| % .10l | Time limit for which a single job can be run |
| % .10m | Total node memory |
| % .10e | Required free memory of the node |
| % .15C | Amount of CPU in each possible state (Allocated / Idle / Other / Total) |

To also check the status of the allocation of the defined generic resources (eg GPU), you can use the following command, for example:

```sh
$ sinfo -N -O "nodelist:12,freemem:10,cpusstate:15,gresused"
 
NODELIST    FREE_MEM  CPUS(A/I/O/T)  GRES_USED           
gnode01     29356     0/32/0/32      gpu:tesla:0(IDX:N/A)
gnode02     23472     24/8/0/32      gpu:tesla:1(IDX:0)  
gnode03     175952    32/0/0/32      gpu:tesla:0(IDX:N/A)
gnode04     55845     0/32/0/32      gpu:tesla:0(IDX:N/A)
wnode01     188152    0/32/0/32      gpu:0               
wnode02     180899    32/0/0/32      gpu:0               
wnode03     179464    32/0/0/32      gpu:0               
wnode04     179768    32/0/0/32      gpu:0  
```
**Please Note**: The equivalent of gresused is not available using the -o option. To know all the possible information that can be displayed, refer to the official sinfo documentation.

### Job Limits
To distribute compute resources equally to all purpleJeans users, RCF sets limits on the amount of compute resources that can be required by a single user at any one time.

The maximum execution time for a single job (batch or interactive) is 12 hours for the xhicpu partition and 6 hours for the xgpu partition.

Additional information about limits, such as the maximum number of CPUs that a user can request at any one time or the number of jobs that can be submitted simultaneously on a given partition, is available by entering the qos command on any purpleJeans login or compute node. . Note that these limits are often different depending on the partition.

Usage limits may vary, the qos command always provides the most up-to-date information.

If the user's research work requires a temporary exception to a particular limit, a special assignment can be requested by contacting RCF (rcf@uniparthenope.it). Special allocations are valued on an individual basis and may or may not be granted.

# Software

purpleJeans uses environment modules for software management. The module system allows you to set up the shell environment to make it easier to run and compile the software. It also allows you to make available numerous software packages and libraries that would otherwise conflict with each other.

When you first log into purpleJeans, you will be placed in a very basic user environment with minimal software available. The module system is a script-based system used to manage the user environment and to "activate" software packages. To access the software installed on purpleJeans, you must first load the corresponding software module.

Base module commands:

| Command | Description |
|----------------------|---------------------------------------------------|
| module avail | list all available software modules |
| module disp [name] | list modules matching [name] |
| module load [name] | loads the module named |
| module unload [name] | download the indicated form |
| module list | list modules currently loaded for user |

Complete list of software modules:

```
anaconda/3                                     netcdf/4.8.1-gcc-4.8.5
cmake/3.19.2                                   netcdf/4.8.1-gcc-8.3.1
cmake/3.21.4                                   null
cuda/10.0                                      nvhpc/20.11
cuda/10.1                                      nvhpc-byo-compiler/20.11
cuda/11.0                                      nvhpc-nompi/20.11
cuda/11.2                                      ompi-4.0.1-gcc-4.8.5
cuda/11.3                                      ompi-4.0.1-gcc-8.3.1
cuda/11.4.2                                    ompi-4.1.0-gcc-8.3.1
cuda/11.5                                      opengrads/2.2.1
dot                                            openmpi/3.1.3/2019
gcc-8.3.1                                      paraview/5.8.1
intel_MKL-2019u5                               tau/tau-2.30.2-ompi-4.0.1-cuda-11.2-gcc-4.8.5
jasper/2.0.14-gcc-8.3.1                        tau/tau-2.30.2-ompi-4.1.0-cuda-11.2-gcc-8.3.1
module-git                                     use.own
module-info                                    vapor/3.2.0
modules                                        vapor/3.3.0
mvapich2-2.3.5-gcc-8.3.1 
```

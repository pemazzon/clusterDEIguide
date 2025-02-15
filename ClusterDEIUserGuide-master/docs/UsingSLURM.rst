Using Slurm
===========

.. _slurm:

Slurm is a free and open-source job scheduler for
Linux and Unix-like kernels, used by many of the
world's supercomputers and computer clusters.

Slurm is used @ CINECA, CERN, RIKEN, LIVERMORE,
OAK RIDGE, etc.

Installed Slurm version: 20.02.1 (Feb 2020)

Slurm Partitions
----------------

.. _slurmpartitions:

A Slurm partition is a group of resources. Each
partition can be considered as a job queue, each of
which has some constraint (e.g. number of running
jobs, job time limit, etc.).

ClusterDEI has 2 partitions:

  * **allgroups**: partition to run both serial and parallel
    software. This partition **should not** be used to run
    interactive software;
  * **interactive**: partition to run software in an
    interactive way. This partition has limited resources
    (default: 1 cores, 1GB RAM). Interactive sessions
    should be shorter than 24 hours.

Which partition should I use?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The great majority of Cluster users should use the
allgroups partition. Typical serial and parallel software
that require no interaction with the user should be run
in this partition.

The interactive partition should be used only when a
real time interaction is needed and/or for tasks having
low computation burden. Typical examples are the
installation of software having an interactive installation
procedure, simple file managing/manipulation (e.g.
compressing files), etc.

Important things to remember
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When the user submit a job, he **must** specify in the Slurm options three important things that are:
  
  * **CPU** and/or **TASKS**, if a user has to run a software that is designed to use more than a CPU he has to specify the number of the CPU that the job has to use and/or if a user has to run parallel code he has to specify the number of TASKS that is necessary;
  * **RAM**, since the requested ram is assigned for the exclusive use of the applicant, it is important to size it correctly for two reasons:
   - if more ram is required than necessary, it is not available to other users;
   - if more ram is required than necessary, it may take a long time to have a server with available ram;
  * **TIME**, another thing to consider in the submission of a job file is the expected amount of time the job takes to finish. This is very important because if the user specifies a small amount of time that isn't enough for the job to finish, the job will be killed automatically at the end of the specified time with the results of an unfinished job;

.. ATTENTION::
  In the submission of a job, the time limit set is a maximum of 35 days. In particular cases, if one or more jobs require more execution time, a ticket @`DEI Helpdesk System <https://www.dei.unipd.it/helpdesk/>`_ must be opened to report the request.



Slurm Jobs
----------

.. _slurmjobs:

To run a software on the cluster, the user the user must
prepare a so-called (Slurm) *job file*. A job file is a text file containing
some Slurm instructions/options and the call to user software.

The typical job file is something like this: 

::

   #!/bin/bash
   
   #SBATCH --ntasks 4
   #SBATCH --partition allgroups
   #SBATCH --time 02:00
   #SBATCH --mem 1G
   
   cd $WORKING_DIR   
   #your working directory
   
   srun <your_software>
   
Dissection of the job file
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _jobdissection:

``#!/bin/bash``

This is simply the interpreter of subsequent commands (bash in this case).

``#SBATCH ...``

Following the ``#SBATCH`` directive are special options geared at the SLURM
scheduler. We will see some of them in detail below.

``cd $WORKING_DIR``

This is a useful command to ask the scheduler to change the current directory
before launching the program. This way all the paths can be considered as
relative to the directory the job file resides in.

``srun <your_software>``

This is where you ask the scheduler to run your program for you.

Slurm mandatory options
^^^^^^^^^^^^^^^^^^^^^^^

.. _mandatoryopts:

Slurm (actually *srun*) recognizes a large number of options to suit your needs.
Some options can be specified using either the short (prefixed with just one dash 
and a letter) or the long (two dashes) form. 

There are four **mandatory** options you must specify on your job to successfully run
on the cluster platform. These are:

 -n, -\\-ntasks <num_tasks>
  Number of tasks to launch. For serial code <num_task> should be set to 1, for
  parallel code <num_task> should be set to the
  number of parallel execution flows.
  


 -p, -\\-partition <partition_name>
  Slurm partition. For typical serial or parallel job <partition_name> is *allgroups*

 -t, -\\-time <time>
  Maximum job execution time, where time could be specified as one of

  * mm
  * mm:ss
  * hh:mm:ss
  * dd-hh
  * dd-hh:mm
  * dd-hh:mm:ss


 -\\-mem <size[units]>
  Maximum amount of RAM memory requested. Different units can be specified using the suffix [K|M|G|T]
 
.. note::
  -c, -\\-cpus-per-task <ncpus>
   Number of CPU used for a single task. For single CPU use <cpu_per_task> should be set to 1, for codes that is capable of using more than a CPU <cpu_per_task> should be set to the number of CPUs required.
  
  
  
A more complete job
^^^^^^^^^^^^^^^^^^^

.. _slurmjobfull:

::

  #!/bin/bash

  #SBATCH --job-name <job_name>
  #SBATCH --output output_%j.txt
  #SBATCH --error errors_%j.txt
  #SBATCH --mail-user james@gmail.com
  #SBATCH --mail-type ALL
  #SBATCH
  #SBATCH --time 02:00
  #SBATCH --ntasks 4
  #SBATCH --partition allgroups
  #SBATCH --mem 1G

  cd $WORKING_DIR   
  #your working directory

  srun <your_software>


-\\-job-name <job_name>
  When you queue your job this option can provide a visual clue to distinguish between your jobs.

-\\-output output_%j.txt
  Your output file will be numbered with your JOBID (%j). Subsequent runs will not overwrite the output file.

-\\-error errors_%j.txt
  Same as above for standard error.

-\\-mail-user james@gmail.com
  Depending on what you specify on the companion directive ``mail-type`` the specified user will be
  notified via email.

-\\-mail-type ALL 
  Notify user by email when certain event types occur. The event list can be seen on the *srun* manual page 
  on the frontend node (issuing a ``man srun`` at the command prompt).


SLURM Interaction
-----------------

.. _slurminteract:

Submit a job
^^^^^^^^^^^^

.. _jobsubmit:

Once you wrote your job file you can *submit* it to the scheduler
to get it executed using the sbatch command:

::

 sbatch [options] <job_file>

e.g.: ``sbatch test.slurm``. Upon (successful) job submission, you will get a message like this:

::

 Submitted job 129774

Here 129744 is the JOBID. This number can be used to check for the job progress, to remove it from
the execution queue or for other operations. You can read the sbatch documentation using ``man sbatch``
from the frontend node or visiting the `sbatch web page <https://slurm.schedmd.com/sbatch.html>`_

Options specified inside the job file (after the ``#SBATCH`` directives) can be overridden or
modified on the command line, e.g.:

::

 sbatch --mem 10G --jobname test10G test.slurm

The above command line will set - just for this submission - the jobname to 'test10G' and will
request ten gigabytes of RAM, possibly overriding what specified inside the slurm job file.


Checking job status
^^^^^^^^^^^^^^^^^^^

Once the job enters the queue you can use the *squeue* command to check its status::

 squeue [-l]

The above command will list *all* the jobs in the queue. Since the list can be very long
you can filter only your jobs::

 squeue [-l] -u <user_id>

or you can check a single job providing the JOBID

::

 squeue -j JOBID

To see the complete list of output options and command flags use ``man squeue``
from the frontend node or visit the `squeue web page <https://slurm.schedmd.com/squeue.html>`_

Checking running jobs
^^^^^^^^^^^^^^^^^^^^^

The status of jobs in a **running** state can be checked with::

 sstat

To see the complete list of output statistics (e.g. min/max/avg bytes read/written, min/max/avg CPU time, min/max/avg
memory usage, etc.)  and command options use ``man sstat`` from the frontend node or 
visit the `sstat web page <https://slurm.schedmd.com/sstat.html>`_

Remove a job
^^^^^^^^^^^^

To remove a job from the queue use::

 scancel JOBID

Alternatively if you want to remove **all your jobs** from the queue you can use

::
 
 scancel -u <user_id>

.. caution:: there are no confirmation prompts.

Job accounting
^^^^^^^^^^^^^^

Upon job completion you might want to checkout some information on resources you used.
For this the sacct command can be used::

 sacct -o reqmem,maxrss,averss,elapsed –j <job_id>

Other options can be used. To see a full list consult ``man sacct`` on the frontend node
or the `web version <https://slurm.schedmd.com/sacct.html>`_ 

Job efficiency
^^^^^^^^^^^^^^

Job efficiency measures how precisely you requested the computing resources. **This is a
parameter you should not underestimate.** In fact:

  - If you request too few resources your job will likely crash;
  - If you request too much resources you will likely **wait a lot** for your job to start or,
    **worse**, you will reserve for yourself resources you will never use. This has a
    negative impact on other users too!

Check the job efficiency of a completed job issuing::

 seff JOBID

As an example::

 [admin@runner-01~] seff 54321
 Job ID: 54321
 Cluster: cluster_DEI
 User/Group: admin/admin
 State: COMPLETED (exit code 0)
 Cores: 1
 CPU Utilized: 00:48:40
 CPU Efficiency: 98.68% of 00:49:19 core-walltime
 Memory Utilized: 4.06 GB
 Memory Efficiency: 10.39% of 39.06 GB

The above job was very good at requesting computing cores. On the opposite side
40 GB of RAM was requested (and were therefore *reserved* throughout job 
execution!) but just above 4 GB were needed...

Slurm vs SGE commands
^^^^^^^^^^^^^^^^^^^^^

+----------------------+-----------+------------------------+
| User commands        | Slurm     | SGE                    |
+======================+===========+========================+
| Job submission       | sbatch    | qsub                   |
+----------------------+-----------+------------------------+
| Job deletion         | scancel   | qdel                   |
+----------------------+-----------+------------------------+
| Job status (by job)  | squeue    | qstat -u \*-j          |
+----------------------+-----------+------------------------+
| Job status (by user) | squeue -u | qstat -u               |
+----------------------+-----------+------------------------+
| Queue list           | squeue    | qconf -slq             |
+----------------------+-----------+------------------------+

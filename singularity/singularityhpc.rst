**Singularity and High Performance Computing**
----------------------------------------------

High Performance Computing resources fill an important role in research computing and can support container execution through runtimes such as Singularity or, hopefully soon, rootless Docker, among other options.  

Conducting analyses on HPC clusters happens through different patterns of interaction than running analyses on a cloud VM.  When you login, you are on a node that is shared with lots of people, typically called the "login node". Trying to run jobs on the login node is not "high performance" at all (and will likely get you an admonishing email from the system administrator). Login nodes are intended to be used for moving files, editing files, and launching jobs.

Importantly, most jobs run on an HPC cluster are neither **interactive**, nor **realtime**.  When you submit a job to the scheduler, you must tell it what resources you need (e.g. how many nodes, how much RAM, what type of nodes, and for how long) in addition to what you want to run. Then the scheduler finally has resources matching your requirements, it runs the job for you. If your request is very large, or very long, you may never make it out of the queue. 

For example, on a VM if you run the command:

.. code-block:: bash

  singularity exec docker://python:latest /usr/local/bin/python

The container will immediately start. 

On an HPC system, your job submission script would look something like:

.. code-block:: bash

  #!/bin/bash
  #
  #SBATCH -J myjob                      # Job name
  #SBATCH -o output.%j                  # Name of stdout output file (%j expands to jobId)
  #SBATCH -p development                # Queue name
  #SBATCH -N 1                          # Total number of nodes requested (68 cores/node)
  #SBATCH -n 17                         # Total number of mpi tasks requested
  #SBATCH -t 02:00:00                   # Run time (hh:mm:ss) - 4 hours

  module load singularity/3/3.1
  singularity exec docker://python:latest /usr/local/bin/python

This example is for the Slurm scheduler.  Each of the #SBATCH lines looks like a comment to the bash kernel, but the scheduler reads all those lines to know what resources to reserve for you.

It is usually possible to get an interactive session as well, by using an interactive flag, `-i`. 

.. Note::

  Every HPC cluster is a little different, but they almost universally have a "User's Guide" that serves both as a quick reference for helpful commands and contains guidelines for how to be a "good citizen" while using the system.  For TACC's Stampede2 system, see the  `user guide <https://portal.tacc.utexas.edu/user-guides/stampede2>`_. For The University of Arizona, see the `user guide <https://docs.hpc.arizona.edu/>`_.


How do HPC systems fit into the development workflow?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A few things to consider when using HPC systems:

#. Using ``sudo`` is not allowed on HPC systems, and building a Singularity container from scratch requires sudo.  That means you have to build your containers on a different development system.  You can pull a docker image on HPC systems
#. If you need to edit text files, command line text editors don't support using a mouse, so working efficiently has a learning curve.  There are text editors that support editing files over SSH.  This lets you use a local text editor and just save the changes to the HPC system.

These constraints make HPC systems perfectly suitable for execution environments, but currently a limiting choice for a development environment.  We usually recommend your local laptop or a VM as a development environment where you can iterate on your code rapidly and test container building and execution.

Singularity and MPI
~~~~~~~~~~~~~~~~~~~

Singularity supports MPI fairly well.  Since (by default) the network is the same insde and outside the container, the communication between containers usually just works.  The more complicated bit is making sure that the container has the right set of MPI libraries.  MPI is an open specification, but there are several implementations (OpenMPI, MVAPICH2, and Intel MPI to name three) with some non-overlapping feature sets.  If the host and container are running different MPI implementations, or even different versions of the same implementation, hilarity may ensue.

The general rule is that you want the version of MPI inside the container to be the same version or newer than the host.  You may be thinking that this is not good for the portability of your container, and you are right.  Containerizing MPI applications is not terribly difficult with Singularity, but it comes at the cost of additional requirements for the host system.

.. Note::

  Many HPC Systems, like Stampede2 at TACC and Ocelote at UAHPC, have high-speed, low-latency networks that have special drivers.  Infiniband, Ares, and OmniPath are three different specs for these types of networks.  When running MPI jobs, if the container doesn't have the right libraries, it won't be able to use those special interconnects to communicate between nodes.


Base Docker images
~~~~~~~~~~~~~~~~~~

Depending on the system you will use, you may have to build your own MPI enabled Singularity images (to get the versions to match).

When running at TACC, there is a set of curated Docker images for use in the FROM line of your own containers.  You can see a list of availabe images at `https://hub.docker.com/u/tacc <https://hub.docker.com/u/tacc>`_

Specifically, you can use the ``tacc/tacc-ubuntu18-mvapich2.3-psm2`` image to satisfy the MPI architecture and version requirements for running on Stampede2.

Because you may have to build your own MPI enabled Singularity images (to get the versions to match), here is a 3.1 compatible example of what it may look like:

.. code-block:: bash
  BootStrap: debootstrap
  OSVersion: xenial
  MirrorURL: http://us.archive.ubuntu.com/ubuntu/
  
  %runscript
      echo "This is what happens when you run the container..."

  %post
      echo "Hello from inside the container"
      sed -i 's/$/ universe/' /etc/apt/sources.list
      apt update
      apt -y --allow-unauthenticated install vim build-essential wget gfortran bison libibverbs-dev libibmad-dev libibumad-dev librdmacm-dev libmlx5-dev libmlx4-dev
      wget http://mvapich.cse.ohio-state.edu/download/mvapich/mv2/mvapich2-2.1.tar.gz
      tar xvf mvapich2-2.1.tar.gz
      cd mvapich2-2.1
      ./configure --prefix=/usr/local
      make -j4
      make install
      /usr/local/bin/mpicc examples/hellow.c -o /usr/bin/hellow

You could also build in everything in a Dockerfile and convert the image to Singularity at the end.

Once you have a working MPI container, invoking it would look something like:

.. code-block:: bash

  mpirun -np 4 singularity exec ./mycontainer.sif /app.py arg1 arg2

This will use the **host MPI** libraries to run in parallel, and assuming the image has what it needs, can work across many nodes.

For a single node, you can also use the **container MPI** to run in parallel (usually you don't want this)

.. code-block:: bash

  singularity exec ./mycontainer.sif mpirun -np 4 /app.py arg1 arg2


Example Containerized MPI App
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In your Docker development environment, make a new directory in which to build up a new image and download (or copy and paste) two files in that directory:

`https://raw.githubusercontent.com/TACC/containers_at_tacc/master/docs/scripts/Dockerfile.mpi <https://raw.githubusercontent.com/TACC/containers_at_tacc/master/docs/scripts/Dockerfile.mpi>`_

`https://raw.githubusercontent.com/TACC/containers_at_tacc/master/docs/scripts/pi-mpi.py <https://raw.githubusercontent.com/TACC/containers_at_tacc/master/docs/scripts/pi-mpi.py>`_

Take a look at both files.  ``pi-mpi.py`` is a simple MPI Python script that approximates pi (very inefficiently) by random sampling.  ``Dockerfile.mpi`` is an updated Dockerfile that uses the TACC base image to satisfy all the MPI requirements on Stampede2.

Next, try building the new container.

.. code-block:: bash

	$ docker build -t USERNAME/pi-estimator:0.1-mpi -f Dockerfile.mpi .

Don't forget to change USERNAME to your DockerHub username.  

Once you have successfully built an image, push it up to DockerHub with the ``docker push`` command so that we can pull it back down on Stampede2.

Running an MPI Container on Stampede2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To test, we can grab an interactive session that has two nodes.  That way we can see if we can make the two nodes work together. On TACC systems, the "idev" command will start an interactive session on a compute node:

.. code-block:: bash

    $ idev -m 60 -p normal -N 2 -n 128


Once you have nodes at your disposal and a container on DockerHub, invoking it would look something like:

.. code-block:: bash

  module load tacc-singularity
  cd $WORK
  singularity pull docker://USERNAME/pi-estimator:0.1-mpi
  time singularity exec pi-estimator_0.1-mpi.sif pi-mpi.py 10000000
  time ibrun singularity exec pi-estimator_0.1-mpi.sif pi-mpi.py 10000000

.. Note::
  TACC uses a command called ``ibrun`` on all of its systems that configures MPI to use the high-speed, low-latency network.  If you are familiar with MPI, this is the functional equivalent to ``mpirun``

The first ``singularity exec pi-estimator_0.1-mpi.sif pi-mpi.py 10000000`` command will use 1 CPU core to sample ten million times.  The second command, using ``ibrun`` will run 128 processes that sample ten million times each and pass their results back to the "rank 0" MPI process to merge the results.

This will use the **host MPI** libraries to run in parallel, and assuming the image has what it needs, can work across many nodes.

As an aside, for a single node you can also use the **container MPI** to run in parallel (but usually you don't want this).

When you are don with your interactive session, don't forget to ``exit`` to end the session and go back to the login node.


Singularity and GPU Computing
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

GPU support in Singularity is very good.

Since Singularity supported docker containers, it has been fairly simple to utilize GPUs for machine learning code like TensorFlow. We will not do this as a hands-on exercise, but in general the procedule is as follows.

.. code-block:: bash

  # Load the singularity module
  module load singularity/3/3.1
  
  # Pull your image
  singularity pull docker://nvidia/caffe:latest
  
  singularity exec --nv caffe-latest.sif caffe device_query -gpu 0

Please note that the --nv flag specifically passes the GPU drivers into the container. If you leave it out, the GPU will not be detected.

.. code-block:: bash

  # this is missing the --nv flag and will not work
  singularity exec caffe-latest.sif caffe device_query -gpu 0

The main requirement for GPU containers to work is that the version of the host drivers matches the major version of the library inside the container.  So, for example, if CUDA 10 is on the host, the container needs to use CUDA 10 internally.

For TensorFlow, you can directly pull their latest GPU image and utilize it as follows.

.. code-block:: bash

  # Change to your $WORK directory
  cd $WORK
  #Get the software
  git clone https://github.com/tensorflow/models.git ~/models
  # Pull the image
  singularity pull docker://tensorflow/tensorflow:latest-gpu
  # Run the code
  singularity exec --nv tensorflow-latest-gpu.sif python $HOME/models/tutorials/image/mnist/convolutional.py


The University of Arizona HPS `Singularity examples <https://docs.hpc.arizona.edu/display/UAHPC/Containers>`_. 


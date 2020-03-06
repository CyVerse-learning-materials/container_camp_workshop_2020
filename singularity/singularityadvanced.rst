**Advanced Singularity**
------------------------

|singularity|

5.0 Building your own Containers from scratch
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In this section we'll go over the creation of Singularity containers from a recipe file, called ``Singularity`` (equivalent to ``Dockerfile``).

5.1 Keep track of downloaded containers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default, Singularity uses a temporary cache to hold Docker tarballs:

.. code-block:: bash

  $ ls ~/.singularity

You can change these by specifying the location of the cache and temporary directory on your localhost:

.. code-block:: bash

  $ sudo mkdir tmp
  $ sudo mkdir scratch

  $ SINGULARITY_TMPDIR=$PWD/scratch SINGULARITY_CACHEDIR=$PWD/tmp singularity --debug pull --name ubuntu-tmpdir.sif docker://ubuntu

5.2 Building Singularity containers
==================================

Like Docker, which uses a `Dockerfile` to build its containers, Singularity uses a file called ``Singularity``

When you are building locally, you can name this file whatever you wish, but a better practice is to put it in a directory and name it ``Singularity`` - as this will help later on when developing on Singularity-Hub and GitHub.
Create a container using a custom Singularity file:

.. code-block:: bash

	$ singularity build ubuntu-latest.sif Singularity

We've already covered how you can pull an existing container from Docker Hub, but we can also build a Singularity container from docker using the build command:

.. code-block:: bash

	$ sudo singularity build --sandbox ubuntu-latest/  docker://ubuntu

	$ singularity shell --writable ubuntu-latest/

	Singularity ubuntu-latest.sif:~> apt-get update
  
Does it work?

.. code-block:: bash

	$ sudo singularity shell ubuntu-latest.sif

	Singularity: Invoking an interactive shell within container...

	Singularity ubuntu-latest.sif:~> apt-get update

When I try to install software to the image without `sudo` it is denied, because root is the owner of the container. When I use ``sudo`` I can install software to the container. The software remain in the sandbox container after closing the container and restart.

In order to make these changes permanant, I need to rebuild the sandbox as a ``.sif`` image

.. code-block:: bash

	$ sudo singularity build ubuntu-latest.sif ubuntu-latest/

.. Note::

	Why is creating containers in this way a **bad** idea?

5.2.1: Exercise (~30 minutes): Create a Singularity file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`SyLabs User-Guide <https://sylabs.io/guides/3.5/user-guide/>`_ 

A ``Singularity`` file can be hosted on Github and will be auto-detected by `Singularity-Hub <https://www.singularity-hub.org/>`_ when you set up your container Collection.

Building your own containers requires that you have `sudo` privileges - therefore you'll need to develop these on your local machine or on a VM that you can gain root access on.

- **Header**

The top of the file, selects the base OS for the container, just like ``FROM`` in Docker. 

`Bootstrap:` references another registry (e.g. ``docker`` for DockerHub, ``debootstrap``, or ``shub`` for Singularity-Hub). 

``From:`` selects the tag name. 

.. code-block:: bash

	Bootstrap: shub
	From: vsoch/hello-world

Pulls a container from Singularity Hub (< v2.6.1)

Using `debootstrap` with a build that uses a mirror:

.. code-block:: bash

	BootStrap: debootstrap
	OSVersion: xenial
	MirrorURL: http://us.archive.ubuntu.com/ubuntu/

Using a `localimage` to build:

.. code-block:: bash

	Bootstrap: localimage
	From: /path/to/container/file/or/directory

Using CentOS-like container:

.. code-block:: bash

	Bootstrap: yum
	OSVersion: 7
	MirrorURL: http://mirror.centos.org/centos-7/7/os/x86_64/
	Include:yum

Note: to use `yum` to build a container you should be operating on a RHEL system, or an Ubuntu system with `yum` installed.

The container registries which Singularity uses are listed in the `Introduction Section 3.1 <https://learning.cyverse.org/projects/container_camp_workshop_2019/en/latest/singularity/singularityintro.html#downloading-pre-built-images>`_.

- The Singularity file uses sections to specify the dependencies, environmental settings, and runscripts when it builds.

The additional sections of a Singularity file include:

*  %help - create text for a help menu associated with your container
*  %setup - executed on the host system outside of the container, after the base OS has been installed.
*  %files - copy files from your host system into the container
*  %labels - store metadata in the container
*  %environment - loads environment variables at the time the container is run (not built)
*  %post - set environment variables during the build
*  %runscript - executes a script when the container runs
*  %test - runs a test on the build of the container


**Setting up Singularity file system**
--------------------------------------

- **Help**

`%help` section can be as verbose as you want

.. code-block:: bash

	Bootstrap: docker
	From: ubuntu

	%help
	This is the container help section.

- **Setup**

`%setup` commands are executed on the localhost system outside of the container - these files could include necessary build dependencies. We can copy files to the `$SINGULARITY_ROOTFS` file system can be done during `%setup`

- **Files**

`%files` include any files that you want to copy from your localhost into the container.

- **Post**

`%post` includes all of the environment variables and dependencies that you want to see installed into the container at build time.

- **Environment**

`%environment` includes the environment variables which we want to be run when we start the container

- **Runscript**

`%runscript` does what it says, it executes a set of commands when the container is run.

**Example Singularity file**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Example Singularity file bootstrapping a `Docker <https://hub.docker.com/_/ubuntu/>`_ Ubuntu (16.04) image.

.. code-block:: bash

	BootStrap: docker
	From: ubuntu:18.04

	%post
   	   apt-get -y update
   	   apt-get -y install fortune cowsay lolcat

	%environment
   	   export LC_ALL=C
   	   export PATH=/usr/games:$PATH

	%runscript
   	   fortune | cowsay | lolcat

	%labels
   	   Maintainer Tyson Swetnam
   	   Version v0.1

Build the container:

.. code-block:: bash

    singularity build cowsay.sif Singularity

Run the container:

.. code-block:: bash

    singularity run cowsay.sif

.. Note::

	If you build a `squashfs` container, it is immutable (you cannot `--writable` edit it)


6.1 Using HPC Environments
==========================

Conducting analyses on high performance computing clusters happens through different patterns of interaction than running analyses on a cloud VM.  When you login, you are on a node that is shared with lots of people, typically called the "login node". Trying to run jobs on the login node is not "high performance" at all (and will likely get you an admonishing email from the system administrator). Login nodes are intended to be used for moving files, editing files, and launching jobs.

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
#. Singularity is in the process of changing image formats.  Depending on the version of Singularity running on the HPC system, new squashFS or .simg formats may not work.


6.2 Singularity and MPI
========================

Singularity supports MPI fairly well.  Since (by default) the network is the same insde and outside the container, the communication between containers usually just works.  The more complicated bit is making sure that the container has the right set of MPI libraries.  MPI is an open specification, but there are several implementations (OpenMPI, MVAPICH2, and Intel MPI to name three) with some non-overlapping feature sets.  If the host and container are running different MPI implementations, or even different versions of the same implementation, hilarity may ensue.

The general rule is that you want the version of MPI inside the container to be the same version or newer than the host.  You may be thinking that this is not good for the portability of your container, and you are right.  Containerizing MPI applications is not terribly difficult with Singularity, but it comes at the cost of additional requirements for the host system.

.. Note::

  Many HPC Systems, like Stampede2, have high-speed, low-latency networks that have special drivers.  Infiniband, Ares, and OmniPath are three different specs for these types of networks.  When running MPI jobs, if the container doesn't have the right libraries, it won't be able to use those special interconnects to communicate between nodes.

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


6.3 Singularity and GPU Computing
=================================

GPU support in Singularity is fantastic

Since Singularity supported docker containers, it has been fairly simple to utilize GPUs for machine learning code like TensorFlow. From Maverick, which is TACC’s GPU system:

.. code-block:: bash

  # Load the singularity module
  module load singularity/3/3.1
  
  # Pull your image
  
  singularity pull docker://nvidia/caffe:latest
  
  singularity exec --nv caffe-latest.sif caffe device_query -gpu 0

Please note that the --nv flag specifically passes the GPU drivers into the container. If you leave it out, the GPU will not be detected.

.. code-block:: bash

  singularity exec caffe-latest.sif caffe device_query -gpu 0

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

.. Note::

    You probably noticed that we check out the models repository into your $HOME directory. This is because your $HOME and $WORK directories are only available inside the container if the root folders /home and /work exist inside the container. In the case of tensorflow-latest-gpu.img, the /work directory does not exist, so any files there are inaccessible to the container.

The University of Arizona HPS `Singularity examples <https://docs.hpc.arizona.edu/display/UAHPC/Containers>`_. 

7.0 Cryptographic Security
~~~~~~~~~~~~~~~~~~~~~~~~~~

`Documentation <https://www.sylabs.io/guides/3.5/user-guide/signNverify.html>`_

.. |singularity| image:: ../img/singularity.png
  :height: 200
  :width: 200
  
  <a href="https://sylabs.io/" target="blank">singularity</a>

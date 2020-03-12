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


Cryptographic Security
~~~~~~~~~~~~~~~~~~~~~~~~~~

`Documentation <https://www.sylabs.io/guides/3.5/user-guide/signNverify.html>`_

.. |singularity| image:: ../img/singularity.png
  :height: 200
  :width: 200
  
  <a href="https://sylabs.io/" target="blank">singularity</a>

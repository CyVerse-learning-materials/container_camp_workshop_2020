**Introduction to Singularity**
-------------------------------

|singularity|

1. Prerequisites
================

There are no specific skills needed beyond a basic comfort with the command line and using a text editor. Prior experience installing Linux applications could be helpful but is not required.

.. Note:: 
      
      *Important*: `Singularity is compatible with Docker <https://www.sylabs.io/2018/04/singularity-compatibility-with-docker-containers/>`_, but they do have distinct differences. 
 
   Key Differences:
 
      **Docker**:
      
      * Inside a Docker container the user has escalated privileges, effectively making them `root` on that host system. This privilege is not supported by *most* administrators of High Performance Computing (HPC) centers. Meaning that Docker is not, and will likely never be, installed natively on your HPC.
      
      **Singularity**:
         
      * Same user inside as outside the container
      * User only has root privileges if elevated with `sudo` when container is run
      * Can run (and modify!) existing Docker images and containers

  These key differences allow Singularity to be installed on most HPC centers. Because you can run virtually all Docker containers in Singularity, you can effectively run Docker on an HPC. 

2. Singularity Installation
===========================

Sylabs Singularity homepage: `https://www.sylabs.io/docs/ <https://www.sylabs.io/docs/>`_

Singularity is more likely to be used on a remote system that you don't have control of (e.g. HPC).

2.1 Install Singularity on Laptop
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To Install Singularity on your laptop or desktop PC follow the instructions from Singularity: https://www.sylabs.io/guides/3.5/user-guide/installation.html#installation
  
2.2 HPC
~~~~~~~

Load the Singularity module on a HPC

If you are interested in working on HPC, you may need to contact your systems administrator and request they install `Singularity  <https://www.sylabs.io/guides/3.5/user-guide/installation.html#installation>`_. Because singularity ideally needs setuid, your admins may have some qualms about giving Singularity this privilege. If that is the case, you might consider forwarding `this letter <https://www.sylabs.io/guides/3.5/user-guide/installation.html#singularity-on-a-shared-resource>`_ to your admins.

Most HPC systems are running Environment Modules with the simple command `module`. 

You can check to see what is available:

.. code-block:: bash

  $ module avail singularity

If Singularity is installed, load a specific version:

.. code-block:: bash

	$ module load singularity/3/3.5

2.3 Atmosphere Cloud
~~~~~~~~~~~~~~~~~~~~~

CyVerse staff have deployed an Ansible playbook called ``ez`` for software installation which includes `Singularity <https://cyverse-ez-quickstart.readthedocs-hosted.com/en/latest/#>`_. This command only requires you to type a short line of code to install an entire software stack with all of its dependencies.

Start any *Featured* instance on `Atmosphere <../cyverse/boot.html>_`.

Type in the following in a web shell or ``ssh`` terminal.

.. code-block:: bash

	$ ezs -r 3.5.1
	DEBUG: set version to 3.5.1

	* Updating ez singularity and installing singularity (this may take a few minutes, coffee break!)
	Cloning into '/opt/cyverse-ez-singularity'...
	remote: Enumerating objects: 24, done.
	remote: Total 24 (delta 0), reused 0 (delta 0), pack-reused 24
	Unpacking objects: 100% (24/24), done.
	* ez singularity or singularity itself may not have updated successfully, but you can probably try executing it

	To test singularity, type: singularity run shub://vsoch/hello-world
	Hint: it should output "RaawwWWWWWRRRR!!")

2.4 Check Installation
~~~~~~~~~~~~~~~~~~~~~~

Singularity should now be installed on your laptop or VM, or loaded on the HPC, you can check the installation with:

.. code-block:: bash

    $ singularity pull shub://vsoch/hello-world
	INFO:    Downloading shub image
	 59.75 MiB / 59.75 MiB [=====================================================================================================] 100.00% 49.24 MiB/s 1s
	tswetnam@tysons-box:~$ singularity run hello-world_latest.sif 
	RaawwWWWWWRRRR!! Avocado!

Singularity’s command line interface allows you to build and interact with containers transparently. You can run programs inside a container as if they were running on your host system. You can easily redirect IO, use pipes, pass arguments, and access files, sockets, and ports on the host system from within a container.

The help command gives an overview of Singularity options and subcommands as follows:

.. code-block:: bash

	$ singularity
	Usage:
	  singularity [global options...] <command>

	Available Commands:
	  build       Build a Singularity image
	  cache       Manage the local cache
	  capability  Manage Linux capabilities for users and groups
	  config      Manage various singularity configuration (root user only)
	  delete      Deletes requested image from the library
	  exec        Run a command within a container
	  inspect     Show metadata for an image
	  instance    Manage containers running as services
	  key         Manage OpenPGP keys
	  oci         Manage OCI containers
	  plugin      Manage Singularity plugins
	  pull        Pull an image from a URI
	  push        Upload image to the provided URI
	  remote      Manage singularity remote endpoints
	  run         Run the user-defined default command within a container
	  run-help    Show the user-defined help for an image
	  search      Search a Container Library for images
	  shell       Run a shell within a container
	  sif         siftool is a program for Singularity Image Format (SIF) file manipulation
	  sign        Attach a cryptographic signature to an image
	  test        Run the user-defined tests within a container
	  verify      Verify cryptographic signatures attached to an image
	  version     Show the version for Singularity

	Run 'singularity --help' for more detailed usage information.

Information about subcommand can also be viewed with the help command.

.. code-block:: bash

	$ singularity help pull
	Pull an image from a URI

	Usage:
	  singularity pull [pull options...] [output file] <URI>

	Description:
	  The 'pull' command allows you to download or build a container from a given
	  URI. Supported URIs include:

	  library: Pull an image from the currently configured library
	      library://user/collection/container[:tag]

	  docker: Pull an image from Docker Hub
	      docker://user/image:tag

	  shub: Pull an image from Singularity Hub
	      shub://user/image:tag

	  oras: Pull a SIF image from a supporting OCI registry
	      oras://registry/namespace/image:tag

	  http, https: Pull an image using the http(s?) protocol
	      https://library.sylabs.io/v1/imagefile/library/default/alpine:latest

	Options:
	      --arch string      architecture to pull from library (default "amd64")
	      --dir string       download images to the specific directory
	      --disable-cache    dont use cached images/blobs and dont create them
	      --docker-login     login to a Docker Repository interactively
	  -F, --force            overwrite an image file if it exists
	  -h, --help             help for pull
	      --library string   download images from the provided library
				 (default "https://library.sylabs.io")
	      --no-cleanup       do NOT clean up bundle after failed build, can be
				 helpul for debugging
	      --nohttps          do NOT use HTTPS with the docker:// transport
				 (useful for local docker registries without a
				 certificate)


	Examples:
	  From Sylabs cloud library
	  $ singularity pull alpine.sif library://alpine:latest

	  From Docker
	  $ singularity pull tensorflow.sif docker://tensorflow/tensorflow:latest

	  From Shub
	  $ singularity pull singularity-images.sif shub://vsoch/singularity-images

	  From supporting OCI registry (e.g. Azure Container Registry)
	  $ singularity pull image.sif oras://<username>.azurecr.io/namespace/image:tag


	For additional help or support, please visit https://www.sylabs.io/docs/

3. Downloading pre-built images
================================

The easiest way to use a Singularity is to ``pull`` an existing container from one of the Registries.

You can use the ``pull`` command to download pre-built images from a number of Container Registries, here we'll be focusing on the `Singularity-Hub <https://www.singularity-hub.org>`_ or `DockerHub <https://hub.docker.com/>`_.

Container Registries: 

* `library` - images hosted on Sylabs Cloud
* `shub` - images hosted on Singularity Hub
* `docker` - images hosted on Docker Hub
* `localimage` - images saved on your machine
* `yum` - yum based systems such as CentOS and Scientific Linux
* `debootstrap` - apt based systems such as Debian and Ubuntu
* `arch` - Arch Linux
* `busybox` - BusyBox
* `zypper` - zypper based systems such as Suse and OpenSuse

3.1 Pulling an image from Singularity Hub
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Similar to previous example, in this example I am pulling a base Ubuntu container from Singularity-Hub:

.. code-block:: bash

    $ singularity pull shub://singularityhub/ubuntu
    WARNING: Authentication token file not found : Only pulls of public images will succeed
 	88.58 MiB / 88.58 MiB [===============================================================================================] 100.00% 31.86 MiB/s 2s

You can rename the container using the `--name` flag:
  
.. code-block:: bash

    $ singularity pull --name ubuntu_test.simg shub://singularityhub/ubuntu
    WARNING: Authentication token file not found : Only pulls of public images will succeed
 	88.58 MiB / 88.58 MiB [===============================================================================================] 100.00% 35.12 MiB/s 2s

The above command will save the alpine image from the Container Library as ``alpine.sif``

3.2 Pulling an image from Docker Hub
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This example pulls an ``ubuntu:16.04`` image from DockerHub and saves it to the working directory.

.. code-block:: bash

	$ singularity pull docker://ubuntu:20.04
	INFO:    Converting OCI blobs to SIF format
	INFO:    Starting build...
	Getting image source signatures
	Copying blob 8f6b7df711c8 done
	Copying blob 0703c52b8763 done
	Copying blob 07304348ce1b done
	Copying blob 4795dceb8869 done
	Copying config 05ac933964 done
	Writing manifest to image destination
	Storing signatures
	2020/03/09 16:14:12  info unpack layer: sha256:8f6b7df711c8a4733138390ff2aba1bfeb755bf4736c39c6e4858076c40fb5eb
	2020/03/09 16:14:13  info unpack layer: sha256:0703c52b8763604318dcbb1730c82ad276a487335ecabde2f43f69a6222e8090
	2020/03/09 16:14:13  info unpack layer: sha256:07304348ce1b6d24f136a3c4ebaa800297b804937a6942ce9e9fe0dac0b0ca74
	2020/03/09 16:14:13  info unpack layer: sha256:4795dceb8869bdfa64f3742e1df492e6f31baf9cfc36f1a042a8f981607e99a2
	INFO:    Creating SIF file...
	INFO:    Build complete: ubuntu_20.04.sif


.. warning::

	Pulling Docker images reduces reproducibility. If you were to pull a Docker image today and then wait six months and pull again, you are not guaranteed to get the same image. If any of the source layers has changed the image will be altered. If reproducibility is a priority for you, try building your images from the Container Library.

3.3 Pulling an image from Sylabs cloud library
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let’s use an easy example of ``alpine.sif`` image from the `container library <https://cloud.sylabs.io/library/>`_

.. code-block :: bash

	$ singularity pull library://alpine:latest
	WARNING: Authentication token file not found : Only pulls of public images will succeed
	INFO:    Downloading library image
 	2.08 MiB / 2.08 MiB [==================================================================================================] 100.00% 5.06 MiB/s 0s

.. Tip::

	You can use ``singularity search <name>`` command to locate groups, collections, and containers of interest on the Container Library

4 Interact with images
======================

You can interact with images in several ways such as ``shell``, ``exec`` and ``run``. 

For these examples we will use a ``cowsay_latest.sif`` image that can be pulled from the Container Library like so.

.. code-block:: bash

	$ singularity pull library://tyson-swetnam/default/cowsay
	INFO:    Downloading library image
	 67.00 MiB / 67.00 MiB [=====================================================================================================] 100.00% 5.45 MiB/s 12s
	WARNING: unable to verify container: cowsay_latest.sif
	WARNING: Skipping container verification

	tswetnam@tysons-box:~$ singularity run cowsay_latest.sif 
	 ________________________________________
	/ Expect a letter from a friend who will \
	\ ask a favor of you.                    /
	 ----------------------------------------
		\   ^__^
		 \  (oo)\_______
		    (__)\       )\/\
			||----w |
			||     ||
4.1 Shell
~~~~~~~~~

The ``shell`` command allows you to spawn a new shell within your container and interact with it as though it were a small virtual machine.

.. code-block:: bash

	$ singularity shell cowsay_latest.sif
	  Singularity cowsay_latest.sif:~>

The change in prompt indicates that you have entered the container (though you should not rely on that to determine whether you are in container or not).

Once inside of a Singularity container, you are the same user as you are on the host system.

.. code-block:: bash

	$ Singularity cowsay_latest.sif:~> whoami
	tswetnam

.. Note::

	``shell`` also works with the library://, docker://, and shub:// URIs. This creates an ephemeral container that disappears when the shell is exited.

4.2 Executing commands
~~~~~~~~~~~~~~~~~~~~~~

The exec command allows you to execute a custom command within a container by specifying the image file. For instance, to execute the ``cowsay`` program within the cowsay_latest.sif container:

.. code-block:: bash

	$ singularity exec cowsay_latest.sif cowsay container camp rocks
 	______________________
	< container camp rocks >
	 ----------------------
	        \   ^__^
	         \  (oo)\_______
	            (__)\       )\/\
	                ||----w |
	                ||     ||

.. Note::

	``exec`` also works with the library://, docker://, and shub:// URIs. This creates an ephemeral container that executes a command and disappears.

4.3 Running a container
~~~~~~~~~~~~~~~~~~~~~~~

Singularity containers contain `runscripts <https://www.sylabs.io/guides/3.0/user-guide/definition_files.html#runscript>`_. These are user defined scripts that define the actions a container should perform when someone runs it. The runscript can be triggered with the ``run`` command, or simply by calling the container as though it were an executable.

.. code-block:: bash

	singularity run lolcow_latest.sif 
	 _________________________________________
	/  You will remember, Watson, how the     \
	| dreadful business of the Abernetty      |
	| family was first brought to my notice   |
	| by the depth which the parsley had sunk |
	| into the butter upon a hot day.         |
	|                                         |
	\ -- Sherlock Holmes                      /
	 -----------------------------------------
	        \   ^__^
	         \  (oo)\_______
	            (__)\       )\/\
	                ||----w |
	                ||     ||

# Exercise - 1
##############

Now that you know how to run containers from Docker, I want you to run a Singular container from `simple-script` Docker image that you create on Day 1 of the workshop. 

.. Note::

	If you don't have ``simple-script`` you can use my image on docker hub - https://hub.docker.com/r/upendradevisetty/simple-script-auto

Here are the brief steps:

1. Go to `Docker hub <https://hub.docker.com/>`_ and look for the Dockerhub image that you built on Day 1

2. Use ``singularity pull`` command to pull the Docker image onto your working directory on the Atmosphere

3. Use ``singularity run`` command to launch a container from the Docker image and check to see if you get the same output that as you get from running ``docker run``

4.3 Running a container on HPC
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For running a container on HPC, you need to have Singularity module available on HPC. Let's first look to see if the Singularity module is available on HPC or not

.. warning::

	The following instructions are from running on UA HPC. It may or may not work on other HPC. Please refer to HPC documentation to find similar commands

.. code-block :: bash

	$ module avail singularity
	------------------------------------------ /cm/shared/uamodulefiles -------------------------------------------
	singularity/2/2.6.1  singularity/3/3.2  singularity/3/3.2.1  singularity/3/3.4.2  singularity/3/3.5.3  

You can see that there are three different versions of Singularity are available. For this workshop, we will use ``singularity/3/3.1``. Let's load it now

.. code-block:: bash

	$ module load singularity/3/3.1

.. |singularity| image:: ../img/singularity.png
  :height: 200
  :width: 200

.. |singularityflow| image:: http://singularity.lbl.gov/assets/img/diagram/singularity-2.4-flow.png
  :width: 800

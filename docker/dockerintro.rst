**Introduction to Docker**
--------------------------

|docker|

Prerequisites
=============

There are no specific skills needed for this tutorial beyond a basic comfort with the command line and using a text editor.

* Install Docker on your laptop:

  - `Mac <https://docs.docker.com/docker-for-mac/>`_
  - `Windows <https://docs.docker.com/docker-for-windows/>`_
  - `Ubuntu <https://docs.docker.com/install/linux/docker-ce/ubuntu/>`_

* Install Docker on a featured Atmosphere image:

.. code-block:: bash

	$ ezd	

1.0 Docker Run
==============

As we just covered in the `previous section <./findingcontainers.html>`_, containers can be found in registries (such as the Docker hub), or you can build your own (See `Advanced Section <./dockeradvanced.html>`_). 

When you're looking for the right container, you can search for images within a registry directly from the command line using ``docker search`` (after you've logged into that registry).

.. code-block:: bash

	$ docker search ubuntu
	  NAME                                                   DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
	  ubuntu                                                 Ubuntu is a Debian-based Linux operating sys…   7310                [OK]                
	  dorowu/ubuntu-desktop-lxde-vnc                         Ubuntu with openssh-server and NoVNC            163                                     [OK]
	  rastasheep/ubuntu-sshd                                 Dockerized SSH service, built on top of offi…   131                                     [OK]
	  ansible/ubuntu14.04-ansible                            Ubuntu 14.04 LTS with ansible                   90                                      [OK]
	  ubuntu-upstart                                         Upstart is an event-based replacement for th…   81                  [OK]                
	  neurodebian                                            NeuroDebian provides neuroscience research s…   43                  [OK]                
	  ubuntu-debootstrap                                     debootstrap --variant=minbase --components=m…   35                  [OK]                
	  1and1internet/ubuntu-16-nginx-php-phpmyadmin-mysql-5   ubuntu-16-nginx-php-phpmyadmin-mysql-5          26                                      [OK]
	  nuagebec/ubuntu                                        Simple always updated Ubuntu docker images w…   22                                      [OK]
	  tutum/ubuntu                                           Simple Ubuntu docker images with SSH access     18                                      
	  ppc64le/ubuntu                                         Ubuntu is a Debian-based Linux operating sys…   11                                      
	  i386/ubuntu                                            Ubuntu is a Debian-based Linux operating sys…   9                                       
	  1and1internet/ubuntu-16-apache-php-7.0                 ubuntu-16-apache-php-7.0                        7                                       [OK]
	  eclipse/ubuntu_jdk8                                    Ubuntu, JDK8, Maven 3, git, curl, nmap, mc, …   5                                       [OK]
	  darksheer/ubuntu                                       Base Ubuntu Image -- Updated hourly             3                                       [OK]
	  codenvy/ubuntu_jdk8                                    Ubuntu, JDK8, Maven 3, git, curl, nmap, mc, …   3                                       [OK]
	  1and1internet/ubuntu-16-nginx-php-5.6-wordpress-4      ubuntu-16-nginx-php-5.6-wordpress-4             2                                       [OK]
	  1and1internet/ubuntu-16-nginx                          ubuntu-16-nginx                                 2                                       [OK]
	  pivotaldata/ubuntu                                     A quick freshening-up of the base Ubuntu doc…   1                                       
	  smartentry/ubuntu                                      ubuntu with smartentry                          0                                       [OK]
	  pivotaldata/ubuntu-gpdb-dev                            Ubuntu images for GPDB development              0                                       
	  1and1internet/ubuntu-16-healthcheck                    ubuntu-16-healthcheck                           0                                       [OK]
	  thatsamguy/ubuntu-build-image                          Docker webapp build images based on Ubuntu      0                                       
	  ossobv/ubuntu                                          Custom ubuntu image from scratch (based on o…   0                                       
	  1and1internet/ubuntu-16-sshd                           ubuntu-16-sshd                                  0                                       [OK]

.. Note::

	Depending on how and where you've installed Docker, you may see a ``permission denied`` error after running the ``$ docker run helo-world`` command. If you're on Linux, you may need to prefix your Docker commands with ``sudo``. Alternatively to run docker command without ``sudo``, you need to add your user name (who has root privileges) to the docker "group". 

	Create the docker group::
	
	$ sudo groupadd docker
	
	Add your user to the docker group::
	
	$ sudo usermod -aG docker $USER

	Log out or close terminal and log back in and your group membership will be initiated

The single most common command that you'll use with Docker is ``docker run`` (`help manual <https://docs.docker.com/engine/reference/commandline/run/>`_). 

``docker run`` starts a container and executes the default entrypoint, or any other command line statement that follows ``run``. 

.. code-block:: bash

	$ docker run alpine ls -l
	total 52
	drwxr-xr-x    2 root     root          4096 Dec 26  2016 bin
	drwxr-xr-x    5 root     root           340 Jan 28 09:52 dev
	drwxr-xr-x   14 root     root          4096 Jan 28 09:52 etc
	drwxr-xr-x    2 root     root          4096 Dec 26  2016 home
	drwxr-xr-x    5 root     root          4096 Dec 26  2016 lib
	drwxr-xr-x    5 root     root          4096 Dec 26  2016 media
	........

.. Note::

	To find out more about a Docker images, run ``docker inspect hello-world``. 
	
In the demo above, you could have used the ``docker pull`` command to download the ``hello-world`` image first. 

When you executed the command ``docker run alpine``, Docker looked for the image, did not find it, and then ran a ``docker pull`` behind the scenes to download the ``alpine`` image with the ``:latest`` tag.

When you run ``docker run alpine``, you provided a command ``ls -l``, so Docker started the command specified and you saw the listing of the alpine file system.

You can use the ``docker images`` command to see a list of all the cached images on your system:

.. code-block:: bash

	$ docker images
	REPOSITORY              TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
	alpine                 	latest              c51f86c28340        4 weeks ago         1.109 MB
	hello-world             latest              690ed74de00f        5 months ago        960 B

Images need to have an ``ENTRYPOINT`` set in their Dockerfile recipe in order for them to return a result when they are run. The ``hello-world`` image echos out the statement that it is present when it executes.

You can change the entrypoint of a container by making a statement after the ``repository/container_name:tag``:

.. code-block:: bash

	$ docker run alpine echo "Hello world"
	Hello world

In this case, the Docker client dutifully ran the ``echo`` command in our ``alpine`` container and then exited. If you've noticed, all of that happened pretty quickly. Imagine booting up a virtual machine, running a command and then killing it. Now you know why they say containers are fast!

Now it's time to see the ``docker ps`` command which shows you all containers that are currently running.

.. code-block:: bash

	$ docker ps
	CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

Since no containers are running, you see a blank line. Let's try a more useful variant: ``docker ps --all``

.. code-block:: bash

	$ docker ps --all
	CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
	36171a5da744        alpine              "/bin/sh"                5 minutes ago       Exited (0) 2 minutes ago                        fervent_newton
	a6a9d46d0b2f        alpine             "echo 'hello from alp"    6 minutes ago       Exited (0) 6 minutes ago                        lonely_kilby
	ff0a5c3750b9        alpine             "ls -l"                   8 minutes ago       Exited (0) 8 minutes ago                        elated_ramanujan
	c317d0a9e3d2        hello-world         "/hello"                 34 seconds ago      Exited (0) 12 minutes ago                       stupefied_mcclintock

What you see above is a list of all containers that you ran. Notice that the STATUS column shows that these containers exited a few minutes ago. 

Try another command, this time to access the container as a shell:

.. code-block:: bash

	$ docker run alpine sh

Wait, nothing happened! Is that a bug? Well, no. 

The container will exit after running any scripted commands such as ``sh``, unless they are run in an "interactive" terminal (TTY) - so for this example to not exit, you need to add the ``-i`` for interactive and ``-t`` for TTY. You can run them both in a single flag as ``-it``, which is the more common way of adding the flag: 


.. code-block:: bash

	$ docker run -it alpine sh
	/ # ls
	bin    dev    etc    home   lib    media  mnt    proc   root   run    sbin   srv    sys    tmp    usr    var
	/ # uname -a
	Linux de4bbc3eeaec 4.9.49-moby #1 SMP Wed Sep 27 23:17:17 UTC 2017 x86_64 Linux

The prompt should change to something more like ``/ # `` -- You are now running a shell inside the container. Try out a few commands like ``ls -l``, ``uname -a`` and others. 

Exit out of the container by giving the ``exit`` command.

.. code-block:: bash

	/ # exit

.. Note::

	If you type ``exit`` your **container** will exit and is no longer active. To check that, try the following::

		$ docker ps --latest
		CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS                          PORTS                    NAMES
		de4bbc3eeaec        alpine                "/bin/sh"                3 minutes ago       Exited (0) About a minute ago                            pensive_leavitt

	If you want to keep the container active, then you can use keys ``ctrl +p`` ``ctrl +q``. To make sure that it is not exited run the same ``docker ps --latest`` command again::

		$ docker ps --latest
		CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS                         PORTS                    NAMES
		0db38ea51a48        alpine                "sh"                     3 minutes ago       Up 3 minutes                                            elastic_lewin

	Now if you want to get back into that container, then you can type ``docker attach <container id>``. This way you can save your container::

		$ docker attach 0db38ea51a48

1.1 House Keeping and Cleaning Up
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Docker images are cached on your machine in the location where Docker was installed. These image files are not visible in the same directory where you might have used ``docker pull <imagename>``.

Some Docker images can be large. Especially Data Science images with many libraries and packages pre-installed. 

.. Important::

	Pulling many images from the Docker Registries may fill up your hard disk!

To inspect your system and disk use:

.. code-block:: bash

	$ docker system info
	
	$ docker system df

To find out how many images are on your machine, type:

.. code-block:: bash

	$ docker images --help

To remove images that you no longer need, type:

.. code-block:: bash

	$ docker system prune --help

This is where it becomes important to differentiate between *images*, *containers*, and *volumes* (which we'll get to more in a bit). You can take care of all of the dangling images and containers on your system. Note, that ``prune`` will not removed your cached *images*

.. code-block:: bash

		$ docker system prune
	WARNING! This will remove:
	  - all stopped containers
	  - all networks not used by at least one container
	  - all dangling images
	  - all dangling build cache

	Are you sure you want to continue? [y/N]

If you add the ``-af`` flag it will remove "all" ``-a`` dangling images, empty containers, AND ALL CACHED IMAGES with "force" ``-f``.

2.0  Managing Docker images
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the previous example, you pulled the ``alpine`` image from the registry and asked the Docker client to run a container based on that image. To see the list of images that are available locally on your system, run the ``docker images`` command.

.. code-block:: bash

	$ docker images
	REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
	ubuntu                     bionic              47b19964fb50        4 weeks ago         88.1MB
	alpine                     latest              caf27325b298        4 weeks ago         5.53MB
	hello-world                latest              fce289e99eb9        2 months ago        1.84kB
	.........

Above is a list of images that I've pulled from the registry and those I've created myself (we'll shortly see how). You will have a different list of images on your machine. The **TAG** refers to a particular snapshot of the image and the **ID** is the corresponding unique identifier for that image.

For simplicity, you can think of an image akin to a Git repository - images can be committed with changes and have multiple versions. When you do not provide a specific version number, the client defaults to latest.

2.1 Pulling and Running a JupyterLab or RStudio-Server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In this section, let's find a Docker image which can run a Jupyter Notebook

Search for official images on Docker Hub which contain the string 'jupyter'

.. code-block:: bash

	$ docker search jupyter
	NAME                                    DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
	jupyter/datascience-notebook            Jupyter Notebook Data Science Stack from htt…   611                                     
	jupyter/all-spark-notebook              Jupyter Notebook Python, Scala, R, Spark, Me…   276                                     
	jupyterhub/jupyterhub                   JupyterHub: multi-user Jupyter notebook serv…   237                                     [OK]
	jupyter/scipy-notebook                  Jupyter Notebook Scientific Python Stack fro…   227                                     
	jupyter/tensorflow-notebook             Jupyter Notebook Scientific Python Stack w/ …   201                                     
	jupyter/pyspark-notebook                Jupyter Notebook Python, Spark, Mesos Stack …   142                                     
	jupyter/minimal-notebook                Minimal Jupyter Notebook Stack from https://…   96                                      
	jupyter/base-notebook                   Small base image for Jupyter Notebook stacks…   95                                      
	jupyterhub/singleuser                   single-user docker images for use with Jupyt…   30                                      [OK]
	jupyter/r-notebook                      Jupyter Notebook R Stack from https://github…   30                                      
	jupyter/nbviewer                        Jupyter Notebook Viewer                         22                                      [OK]
	mikebirdgeneau/jupyterlab               Jupyterlab based on python / alpine linux wi…   21                                      [OK]
	jupyter/demo                            (DEPRECATED) Demo of the IPython/Jupyter Not…   14                                      
	eboraas/jupyter                         Jupyter Notebook (aka IPython Notebook) with…   12                                      [OK]
	jupyterhub/k8s-hub                                                                      11                                      
	nbgallery/jupyter-alpine                Alpine Jupyter server with nbgallery integra…   9                                       
	jupyter/repo2docker                     Turn git repositories into Jupyter enabled D…   7                                       
	jupyterhub/configurable-http-proxy      node-http-proxy + REST API                      5                                       [OK]
	...

Search for images on Docker Hub which contain the string 'rstudio'

.. code-block:: bash

	$ docker search rstudio

	NAME                                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
	rocker/rstudio                            RStudio Server image                            289                                     [OK]
	opencpu/rstudio                           OpenCPU stable release with rstudio-server (…   29                                      [OK]
	rocker/rstudio-stable                     Build RStudio based on a debian:stable (debi…   16                                      [OK]
	dceoy/rstudio-server                      RStudio Server                                  8                                       [OK]
	rocker/rstudio-daily                                                                      6                                       [OK]
	rstudio/r-base                            Docker Images for R                             6                                       
	rstudio/r-session-complete                Images for sessions and jobs in RStudio Serv…   4                                       
	rstudio/rstudio-server-pro                Default Docker image for RStudio Server Pro     1                                       
	aghorbani/rstudio-h2o                     An easy way to start rstudio and H2O to run …   1                                       [OK]
	centerx/rstudio-pro                       NA                                              1                                       [OK]
	mobilizingcs/rstudio                      RStudio container with mz packages pre-insta…   1                                       [OK]
	calpolydatascience/rstudio-notebook       RStudio notebook                                1                                       [OK]	
	...

2.2 Interactive Containers
^^^^^^^^^^^^^^^^^^^^^^^^^^

Let's go ahead and run some basic Integraded Development Environment images from "trusted" organizations on the Docker Hub registry. 

When we want to run a container that runs on the open internet, we need to add a `TCP or UDP port number <https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers>`_ from which we can access the application in a browser using the machine's IP (Internet Protocol) address or DNS (Domain Name Service) location. 

Here are some examples to run basic RStudio and Jupyter Lab:

.. code-block:: bash

	$docker run --rm -p 8787:8787 -e PASSWORD=cc2020 rocker/rstudio 

.. code-block:: bash

	$docker run --rm -p 8888:888 jupyter/base-notebook

.. Note::
	
	We've added the ``--rm`` flag, which means the container will automatically removed from the cache when the container is exited. 
	
	When you start an IDE in a terminal, the terminal connection must stay active to keep the container alive. 

If we want to keep our window in the foreground  we can use the ``-d`` - the *detached* flag will run the container as a background process, rather than in the foreground. When you run a container with this flag, it will start, run, telling you the container ID: 

.. code-block:: bash
	
	$ docker run --rm -d -p 8888:8888 jupyter/base-notebook

	Unable to find image 'jupyter/base-notebook:latest' locally
	latest: Pulling from jupyter/base-notebook
	5c939e3a4d10: Pull complete 
	c63719cdbe7a: Pull complete 
	19a861ea6baf: Pull complete 
	651c9d2d6c4f: Pull complete 
	21b673dc817c: Pull complete 
	1594017be8ef: Pull complete 
	b392f2c5ed42: Pull complete 
	8e4f6538155b: Pull complete 
	7952536f4b86: Pull complete 
	61032726be98: Pull complete 
	3fc223ec0a58: Pull complete 
	23a29aed8d6e: Pull complete 
	25ed667252a0: Pull complete 
	434b2237517c: Pull complete 
	d33fb9062f74: Pull complete 
	fdc8c4d68c3d: Pull complete 
	Digest: sha256:3b8ec8c8e8be8023f3eeb293bbcb1d80a71d2323ae40680d698e2620e14fdcbc
	Status: Downloaded newer image for jupyter/base-notebook:latest
	561016e4e69e22cf2f3b5ff8cbaa229779c2bdf3bdece89b66957f3f3bc5b734
	$
	
Note, that your terminal is still active and you can use it to launch more containers. To view the running container, use the ``docker ps`` command 

.. code-block:: bash
	
	$ docker ps
	CONTAINER ID        IMAGE                   COMMAND                  CREATED              STATUS              PORTS                             NAMES
	561016e4e69e        jupyter/base-notebook   "tini -g -- start-no…"   About a minute ago   Up About a minute   8888/tcp, 0.0.0.0:8888->888/tcp   affectionate_banzai

What if we want a Docker container to `always (re)start <https://docs.docker.com/config/containers/start-containers-automatically/>`_, even after we reboot our machine?

.. code-block:: bash
	
	$ docker run --restart always 

3. Managing Data in Docker
==========================

It is possible to store data within the writable layer of a container, but there are some limitations:

- The data doesn’t persist when that container is no longer running, and it can be difficult to get the data out of the container if another process needs it.

- A container’s writable layer is tightly coupled to the host machine where the container is running. You can’t easily move the data somewhere else.

- Its better to put your data into the container **AFTER** it is build - this keeps the container size smaller and easier to move across networks. 

Docker offers three different ways to mount data into a container from the Docker host: 

  * **volumes** 

  * **bind mounts** 

  * **tmpfs volumes**
  
When in doubt, volumes are almost always the right choice.

3.1 Volumes 
~~~~~~~~~~~

|volumes|

Volumes are often a better choice than persisting data in a container’s writable layer, because using a volume does not increase the size of containers using it, and the volume’s contents exist outside the lifecycle of a given container. While bind mounts (which we will see later) are dependent on the directory structure of the host machine, volumes are completely managed by Docker. Volumes have several advantages over bind mounts:

- Volumes are easier to back up or migrate than bind mounts.
- You can manage volumes using Docker CLI commands or the Docker API.
- Volumes work on both Linux and Windows containers.
- Volumes can be more safely shared among multiple containers.
- A new volume’s contents can be pre-populated by a container.

.. Note::
  
	If your container generates non-persistent state data, consider using a ``tmpfs`` mount to avoid storing the data anywhere permanently, and to increase the container’s performance by avoiding writing into the container’s writable layer.

3.1.1 Choose the -v or –mount flag for mounting volumes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``-v`` or ``--volume``: Consists of three fields, separated by colon characters (:). The fields must be in the correct order, and the meaning of each field is not immediately obvious.

- In the case of named volumes, the first field is the name of the volume, and is unique on a given host machine.
- The second field is the path where the file or directory are mounted in the container.
- The third field is optional, and is a comma-separated list of options, such as ``ro``.

.. code-block:: bash

   -v /home/username/your_data_folder:/data

.. Note::

	Originally, the ``-v`` or ``--volume`` flag was used for standalone containers and the ``--mount`` flag was used for swarm services. However, starting with Docker 17.06, you can also use ``--mount`` with standalone containers. In general, ``--mount`` is more explicit and verbose. The biggest difference is that the ``-v`` syntax combines all the options together in one field, while the ``--mount`` syntax separates them. Here is a comparison of the syntax for each flag.

.. code-block:: bash
	
	$docker run --rm -v $(pwd):/work -p 8787:8787 -e PASSWORD=cc2020 rocker/rstudio 

In the Jupyter Lab example, we use the ``-e`` environmental flag to re-direct the URL of the container at the localhost

.. code-block:: bash
	
	$docker run --rm -v $(pwd):/work -p 8888:8888 -e REDIRECT_URL=http://localhost:8888 jupyter/base-notebook

Once you're in the container, you will see that the ``/work`` directory is mounted in the working directory.

Any data that you add to that folder outside the container will appear INSIDE the container. And any work you do inside the container saved in that folder will be saved OUTSIDE the container as well. 

Docker Commands
===============

+----------------+------------------------------------------------+
| Command        |          Usage                                 |
+================+================================================+
| docker pull    |  Download an image from Docker Hub             |
+----------------+------------------------------------------------+
| docker run     |  *Usage:* ``docker run -it user/image:tag``    |
|                |  starts a container with an entrypoint         |
+----------------+------------------------------------------------+
| docker build   | *Usage:* ``docker build -t user/image:tag .``  |
|                |  Builds a docker image from a Dockerfile in    |
|                |  current working directory. ``-t`` for tagname |
+----------------+------------------------------------------------+
| docker images  |  List all images on the local machine          |
+----------------+------------------------------------------------+
| docker tag     |  Add a new tag to an image                     |
+----------------+------------------------------------------------+
| docker login   |  Authenticate to the Docker Hub                |
|                |  requires username and password                |
+----------------+------------------------------------------------+
| docker push    |  *Usage:* ``docker push user/image:tag``       |
|                |  Upload an image to Docker Hub                 |
+----------------+------------------------------------------------+
| docker inspect |  *Usage:* ``docker inspect containerID``       |
|                |  Provide detailed information on constructs    |
|                |  controlled by Docker                          |
+----------------+------------------------------------------------+
| docker ps -a   |  List all containers on your system            |
+----------------+------------------------------------------------+
| docker rm      |  *Usage:* ``docker rm -f <container>``         |
|                |  Deletes a *container*                         |
|                |  ``-f`` remove running container               |
+----------------+------------------------------------------------+
| docker rmi     |  Deletes an *image*                            |
+----------------+------------------------------------------------+
| docker stop    |  *Usage:* ``docker stop <container>``          |
|                |  Stop a running container                      |
+----------------+------------------------------------------------+
| docker system  |  *Usage:* ``docker system prune``		  |
|                |  Remove old images and cached layers		  |
|                |  *Usage:* ``docker system df``		  |          	  
|                |  View system details (cache size)              |
+----------------+------------------------------------------------+

Getting more help with Docker
=============================

- The command line tools are very well documented:

.. code-block:: bash

   $ docker --help
   # shows all docker options and summaries

.. code-block:: bash

   $ docker COMMAND --help
   # shows options and summaries for a particular command

- Learn `more about docker <https://docs.docker.com/get-started/>`_
4. Extra Demos
==============

4.1 Portainer
~~~~~~~~~~~~~

`Portainer <https://portainer.io/>`_ is an open-source lightweight managment UI which allows you to easily manage your Docker hosts or Swarm cluster.

- Simple to use: It has never been so easy to manage Docker. Portainer provides a detailed overview of Docker and allows you to manage containers, images, networks and volumes. It is also really easy to deploy, you are just one Docker command away from running Portainer anywhere.

- Made for Docker: Portainer is meant to be plugged on top of the Docker API. It has support for the latest versions of Docker, Docker Swarm and Swarm mode.

4.1.1 Installation
^^^^^^^^^^^^^^^^^^

Use the following Docker commands to deploy Portainer. Now the second line of command should be familiar to you by now. We will talk about first line of command in the Advanced Docker session.

.. code-block:: bash

	# on CyVerse Atmosphere:
	$ ezd -p
	
	$ docker volume create portainer_data

	$ docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer

- If you are on mac, you'll just need to access the port 9000 (http://localhost:9000) of the Docker engine where portainer is running using username ``admin`` and password ``tryportainer``

- If you are running Docker on Atmosphere/Jetstream or on any other cloud, you can open ``ipaddress:9000``. For my case this is ``http://128.196.142.26:9000``

.. Note:: 
	
	The ``-v /var/run/docker.sock:/var/run/docker.sock`` option can be used in Mac/Linux environments only.

|portainer_demo|

4.2 Play-with-docker (PWD)
~~~~~~~~~~~~~~~~~~~~~~~~~~

`PWD <https://labs.play-with-docker.com/>`_ is a Docker playground which allows users to run Docker commands in a matter of seconds. It gives the experience of having a free Alpine Linux Virtual Machine in browser, where you can build and run Docker containers and even create clusters in `Docker Swarm Mode <https://docs.docker.com/engine/swarm/>`_. Under the hood, Docker-in-Docker (DinD) is used to give the effect of multiple VMs/PCs. In addition to the playground, PWD also includes a training site composed of a large set of Docker labs and quizzes from beginner to advanced level available at `training.play-with-docker.com <https://training.play-with-docker.com/>`_.

4.2.1 Installation
^^^^^^^^^^^^^^^^^^

You don't have to install anything to use PWD. Just open ``https://labs.play-with-docker.com/`` <https://labs.play-with-docker.com/>`_ and start using PWD

.. Note::

	You can use your Dockerhub credentials to log-in to PWD

|pwd|

.. |docker| image:: ../img/docker.png
  :width: 250


.. |static_site_docker| image:: ../img/static_site_docker.png
  :width: 500


.. |static_site_docker1| image:: ../img/static_site_docker1.png
  :width: 500

.. |portainer_demo| image:: ../img/portainer_demo.png
  :width: 500


.. |pwd| image:: ../img/pwd.png
  :width: 500

.. |catpic| image:: ../img/catpic-1.png
  :width: 500

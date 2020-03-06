**Advanced Docker**
-------------------

Now that we are *relatively* comfortable with Docker, lets look at some advanced Docker topics, such as: 

- Modifying a Dockerfile and creating a new container
- Push a Docker image to the Docker Hub Registry 

1.0 The Dockerfile
^^^^^^^^^^^^^^^^^^

.. Note::

	This is one of the official Docker images provided by the Jupyter project for you to build your own data science notebooks on:  
	
	https://jupyter-docker-stacks.readthedocs.io/en/latest/

Create a file called Dockerfile, and add content to it as described below, e.g. ``nano Dockerfile``

.. Important::

	``Dockerfile`` needs to be capitalized.

.. code-block:: bash

	# base image
	FROM jupyter/scipy-notebook:latest
	
	# reset user to root for installing additional packages
	USER root

	# Install a few dependencies for iCommands, text editing, and monitoring instances
	RUN apt-get update && apt-get install -y \
	    apt-transport-https \
	    gcc \
	    gnupg \
	    htop \
	    less \
	    libfuse2 \
	    libpq-dev \
	    libssl1.0 \
	    lsb \
	    nano \
	    nodejs \
	    python-requests \
	    software-properties-common \
	    vim
	    
	# Install iCommands
	RUN wget https://files.renci.org/pub/irods/releases/4.1.12/ubuntu14/irods-icommands-4.1.12-ubuntu14-x86_64.deb && \
    	dpkg -i irods-icommands-4.1.12-ubuntu14-x86_64.deb && \
    	rm irods-icommands-4.1.12-ubuntu14-x86_64.deb
    
	# reset container user to jovyan
	USER jovyan
	
	# set the work directory
	WORKDIR /home/jovyan
	
	# expose the public port we want to run on
	EXPOSE 8888

	# copy a file into the container
	COPY entry.sh /bin
	RUN mkdir -p /home/jovyan/.irods

	ENTRYPOINT ["bash", "/bin/entry.sh"]

.. Note::

   We use a code line escape character ``\`` to allow single line scripts to be written on multiple lines in the Dockerfile.
   
   We also use the double characters ``&&`` which essentially mean "if true, then do this" while executing the code. The ``&&`` can come at the beginning of a line or the end when used with ``\``

Now let's talk about what each of those lines in the Dockerfile mean.

**1** We'll start by specifying our base image, using the FROM keyword:

.. code-block:: bash

	FROM jupyter/scipy-notebook:latest

**2** Create and copy a file into the image by using ``COPY`` command.

create a new file called ``entry.sh`` -- use your preferred text editor, e.g. ``nano entry.sh`` 

.. code-block:: bash

	#!/bin/bash
	
	echo '{"irods_host": "data.cyverse.org", "irods_port": 1247, "irods_user_name": "$IPLANT_USER", "irods_zone_name": "iplant"}' | envsubst > $HOME/.irods/irods_environment.json

	exec jupyter lab --no-browser 

The ``entry.sh`` file creates an iRODS environment ``.json`` which has CyVerse Data Store configurations pre-written. It also tells Docker to start Jupter Lab and to not pop open a browser tab when doing so.

.. code-block:: bash

	COPY entry.sh /bin

**3** Specify the port number which needs to be exposed. Since Jupyter runs on 8888 that's what we'll expose.

.. code-block:: bash

	EXPOSE 8888

**4** What about ``CMD``?

Notice that unlike our previous Dockerfile this one does not end with a ``CMD`` command. This is on purpose.

Remember: The primary purpose of ``CMD`` is to tell the container which command it should run by default when it is started.

Can you guess what will happen if we don't specify our own 'entrypoint' using ``CMD``?

**5** Setting a new entrypoint 

When this container is run, it will use a different default ``ENTRYPOINT`` than the original container

.. code-block:: bash

	ENTRYPOINT ["bash", "/bin/entry.sh"]
	
This entrypoint runs the shell script ``entry.sh``	

.. _Build the image:

2.0 Build the image
^^^^^^^^^^^^^^^^^^^

.. Note::

	Remember to replace ``<DOCKERHUB_USERNAME>`` with your username. This username should be the same one you created when registering on Docker hub.

.. code-block:: bash

	DOCKERHUB_USERNAME=<YOUR_DOCKERHUB_USERNAME>

For example this is how I assign my dockerhub username

.. code-block:: bash

	DOCKERHUB_USERNAME=tswetnam

Now build the image using the following command:

.. code-block:: bash

	$ docker build -t $DOCKERHUB_USERNAME/jupyterlab-scipy:cyverse .
	Sending build context to Docker daemon  3.072kB
	Step 1/3 : FROM jupyter/minimal-notebook
	 ---> 36c8dd0e1d8f
	Step 2/3 : COPY model.py /home/jovyan/work/
	 ---> b61aefd7a735
	Step 3/3 : EXPOSE 8888
	 ---> Running in 519dcabe4eb3
	Removing intermediate container 519dcabe4eb3
	 ---> 7983fe164dc6
	Successfully built 7983fe164dc6
	Successfully tagged tswetnam/jupyterlab-scipy:cyverse

If everything went well, your image should be ready! Run ``docker images`` and see if your image ``$DOCKERHUB_USERNAME/jupyterlab-scipy:cyverse`` shows.

.. _Run your image:

3.0 Run your image
==================

When Docker can successfully build your Dockerfile, test it by starting a new container from your new image using the docker run command. Don’t forget to include the port forwarding options you learned about before.

.. code-block:: bash

	$ docker run --rm -it -p 8888:8888 $DOCKERHUB_USERNAME/jupyterlab-scipy:cyverse

You should see something like this:

.. code-block:: bash

	Executing the command: jupyter notebook
	[I 07:21:25.396 NotebookApp] Writing notebook server cookie secret to /home/jovyan/.local/share/jupyter/runtime/notebook_cookie_secret
	[I 07:21:25.609 NotebookApp] JupyterLab extension loaded from /opt/conda/lib/python3.7/site-packages/jupyterlab
	[I 07:21:25.609 NotebookApp] JupyterLab application directory is /opt/conda/share/jupyter/lab
	[I 07:21:25.611 NotebookApp] Serving notebooks from local directory: /home/jovyan
	[I 07:21:25.611 NotebookApp] The Jupyter Notebook is running at:
	[I 07:21:25.611 NotebookApp] http://(29a022bb5807 or 127.0.0.1):8888/?token=copy-your-own-token-not-this-one
	[I 07:21:25.611 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
	[C 07:21:25.612 NotebookApp] 
	    
	    Copy/paste this URL into your browser when you connect for the first time,
	    to login with a token:
	        http://(29a022bb5807 or 127.0.0.1):8888/?token=copy-your-own-token-not-this-one

Head over to http://localhost:8888 and your Jupyter notebook server should be running.

Note: Copy the token from your own ``docker run`` output and paste it into the 'Password or token' input box.

3.1 Dockerfile commands summary
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here's a quick summary of the few basic commands we used in our Dockerfiles.

- **FROM** starts the Dockerfile. It is a requirement that the Dockerfile must start with the FROM command. Images are created in layers, which means you can use another image as the base image for your own. The FROM command defines your base layer. As arguments, it takes the name of the image. Optionally, you can add the Dockerhub username of the maintainer and image version, in the format username/imagename:version.

- **RUN** is used to build up the Image you're creating. For each RUN command, Docker will run the command then create a new layer of the image. This way you can roll back your image to previous states easily. The syntax for a RUN instruction is to place the full text of the shell command after the RUN (e.g., RUN mkdir /user/local/foo). This will automatically run in a /bin/sh shell. You can define a different shell like this: RUN /bin/bash -c 'mkdir /user/local/foo'

- **COPY** copies local files into the container.

- **CMD** defines the commands that will run on the Image at start-up. Unlike a RUN, this does not create a new layer for the Image, but simply runs the command. There can only be one CMD per a Dockerfile/Image. If you need to run multiple commands, the best way to do that is to have the CMD run a script. CMD requires that you tell it where to run the command, unlike RUN. So example CMD commands would be:

.. code-block:: bash

	CMD ["python", "./app.py"]

	CMD ["/bin/bash", "echo", "Hello World"]

- **EXPOSE** creates a hint for users of an image which ports provide services. It is included in the information which can be retrieved via ``$ docker inspect <container-id>``.

.. Note::

	The ``EXPOSE`` command does not actually make any ports accessible to the host! Instead, this requires publishing ports by means of the ``-p`` flag when using ``docker run``.

- **PUSH** pushes your image to Docker Cloud, or alternately to a private registry

.. Note::

	If you want to learn more about Dockerfiles, check out `Best practices for writing Dockerfiles <https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/>`_.

3.2 Tagging images
~~~~~~~~~~~~~~~~~~

The notation for associating a local image with a repository on a registry is ``username/repository:tag``. The tag is optional, but recommended, since it is the mechanism that registries use to give Docker images a version. Give the repository and tag meaningful names for the context, such as ``get-started:part2``. This will put the image in the ``get-started`` repository and tag it as ``part2``.

.. Note::

	By default the docker image gets a ``latest`` tag if you don't provide one. Thought convenient, it is not recommended for reproducibility purposes.

Now, put it all together to tag the image. Run docker tag image with your username, repository, and tag names so that the image will upload to your desired destination. For our docker image since we already have our Dockerhub username we will just add tag which in this case is ``1.0``

.. code-block:: bash

	$ docker tag jupyterlab-scipy:cyverse $DOCKERHUB_USERNAME/jupyterlab-scipy:cyverse

4.0 Publishing your image
=========================

4.1 Log into the Docker Hub Registry 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Note::

	If you don’t have an account, sign up for one at `Docker Cloud <https://cloud.docker.com/>`_ or `Docker Hub <https://hub.docker.com/>`_. Make note of your username -- it may or may not be the same as your email, GitHub, or CyVerse username. There are several advantages to registering with registries like DockerHub which we will see later on in the session.

	If you want to authenticate to a different Registry, type the name of the registry after ``login``:

.. code-block:: bash

	$ docker login <registry-name>
	Authenticating with existing credentials...
	WARNING! Your password will be stored unencrypted in /home/tswetnam/.docker/config.json.
	Configure a credential helper to remove this warning. See
	https://docs.docker.com/engine/reference/commandline/login/#credentials-store

	Login Succeeded

If it is your first time logging in you will be queried for your ``username`` and ``password``. 

Login with your Docker ID to push and pull images from Docker Hub or private registry. 

If you don't have a Docker ID, head over to https://hub.docker.com to create one.   
Upload your tagged image to the Dockerhub repository

.. code-block:: bash

	$ docker push $DOCKERHUB_USERNAME/jupyterlab-scipy:cyverse

Once complete, the results of this upload are publicly available. If you log in to Docker Hub, you will see the new image there, with its pull command.

|docker_image|

Congrats! You just made your first Docker image and shared it with the world!

4.2 Pull and run the image from the remote repository
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now run the following command to run the docker image from Dockerhub

.. code-block:: bash

	$ docker run -p 8888:8888 --name notebooktest $DOCKERHUB_USERNAME/jupyterlab-scipy:cyverse

.. Note::

	You don't have to run ``docker pull`` since if the image isn’t available locally on the machine, Docker will pull it from the repository.

Head over to ``http://<vm-address>:8888`` and your app should be live. 

1.2 Private repositories
~~~~~~~~~~~~~~~~~~~~~~~~

In an earlier part, we had looked at the Docker Hub, which is a public registry that is hosted by Docker. While the Dockerhub plays an important role in giving public visibility to your Docker images and for you to utilize quality Docker images put up by others, there is a clear need to setup your own private registry too for your team/organization. For example, CyVerse has it own private registry which will be used to push the Docker images.

1.2.1 Pull down the Registry Image
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You might have guessed by now that the registry must be available as a Docker image from the Docker Hub and it should be as simple as pulling the image down and running that. You are correct!

A Dockerhub search on the keyword ``registry`` brings up the following image as the top result:

|private_registry|

Run a container from ``registry`` Dockerhub image

.. code-block:: bash

	$ docker run -d -p 5000:5000 --name registry registry:2

Run ``docker ps --latest`` to check the recent container from this Docker image

.. code-block:: bash

	$ docker ps --latest
	CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
	6e44a0459373        registry:2          "/entrypoint.sh /e..."   11 seconds ago      Up 10 seconds       0.0.0.0:5000->5000/tcp   registry

1.2.2 Tag the image that you want to push
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Next step is to tag your image under the registry namespace and push it there

.. code-block:: bash

	$ REGISTRY=localhost:5000

	$ docker tag $DOCKERHUB_USERNAME/jupyterlab-scipy:cyverse $REGISTRY/$(whoami)/mynotebook:1.0

1.2.2 Publish the image into the local registry
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Finally push the image to the local registry

.. code-block:: bash

	$ docker push $REGISTRY/$(whoami)/mynotebook:1.0
	The push refers to a repository [localhost:5000/julianp/mynotebook]
	64436820c85c: Pushed 
	831cff83ec9e: Pushed 
	c3497b2669a8: Pushed 
	1c5b16094682: Pushed 
	c52044a91867: Pushed 
	60ab55d3379d: Pushed 
	1.0: digest: sha256:5095dea8b2cf308c5866ef646a0e84d494a00ff0e9b2c8e8313a176424a230ce size: 1572

1.2.3 Pull and run the image from the local repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can also pull the image from the local repository similar to how you pull it from Dockerhub and run a container from it

.. code-block:: bash

	$ docker run -P --name=mynotebooklocal $REGISTRY/$(whoami)/jupyterlab-scipy:cyverse

2. Automated Docker image building from GitHub
==============================================

An automated build is a Docker image build that is triggered by a code change in a GitHub or Bitbucket repository. By linking a remote code repository to a Dockerhub automated build repository, you can build a new Docker image every time a code change is pushed to your code repository.

A build context is a Dockerfile and any files at a specific location. For an automated build, the build context is a repository containing a Dockerfile.

Automated Builds have several advantages:

- Images built in this way are built exactly as specified.
- The Dockerfile is available to anyone with access to your Docker Hub repository.
- Your repository is kept up-to-date with code changes automatically.
- Automated Builds are supported for both public and private repositories on both GitHub and Bitbucket.

2.1 Prerequisites
~~~~~~~~~~~~~~~~~

To use automated builds, you first must have an account on `Docker Hub <https://hub.docker.com>`_ and on the hosted repository provider (`GitHub <https://github.com/>`_ or `Bitbucket <https://bitbucket.org/>`_). While Docker Hub supports linking both GitHub and Bitbucket repositories, here we will use a GitHub repository. If you don't already have one, make sure you have a GitHub account. A basic account is free

.. Note::

	- If you have previously linked your Github or Bitbucket account, you must have chosen the Public and Private connection type. To view your current connection settings, log in to Docker Hub and choose Profile > Settings > Linked Accounts & Services.

	- Building Windows containers is not supported.

2.2 Link your Docker Hub account to GitHub
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1.	Log into Docker Hub.

2.	Click "Create Repository+"

|dockerhub_create|

3.	Click the Build Settings and select ``GitHub``.

|dockerhub_createrepo|

The system prompts you to choose between **Public and Private** and **Limited Access**. The **Public** and **Private** connection type is required if you want to use the Automated Builds.

4.	Press ``Select`` under **Public and Private** connection type.
	If you are not logged into GitHub, the system prompts you to enter GitHub credentials before prompting you to grant access. After you grant access to your code repository, the system returns you to Docker Hub and the link is complete.

|dockerhub_buildsettings|

After you grant access to your code repository, the system returns you to Docker Hub and the link is complete. For example, github linked hosted repository looks like this:

|dockerhub_autobuild|

2.3 Automated Container Builds
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Automated build repositories rely on the integration with a version control system (GitHub or Gitlab) where your ``Dockerfile`` is kept.

Let's create an automatic build for our container using the instructions below:

1. Initialize git repository for the `mynotebook` directory you created for your ``Dockerfile``

.. code-block:: bash

	$ git init
	Initialized empty Git repository in /home/julianp/mynotebook/.git/
	
	$ git status
	On branch master

	Initial commit

	Untracked files:
  	(use "git add <file>..." to include in what will be committed)

		Dockerfile
		model.py

	nothing added to commit but untracked files present (use "git add" to track) 

	$ git add * && git commit -m "Add files and folders"
	[master (root-commit) a4f732a] Add files and folders
	 2 files changed, 10 insertions(+)
	 create mode 100644 Dockerfile
	 create mode 100644 model.py


2. Create a new repository on github by navigating to this URL - https://github.com/new

|create_repo|

.. Note::

	Don't initialize the repository with a README and don't add a license.

3. Push the repository to github

|create_repo2|

.. code-block:: bash

	$ git remote add origin https://github.com/<your-github-username>/mynotebook.git

	$ git push -u origin master
	Counting objects: 7, done.
	Delta compression using up to 8 threads.
	Compressing objects: 100% (5/5), done.
	Writing objects: 100% (7/7), 1.44 KiB | 0 bytes/s, done.
	Total 7 (delta 0), reused 0 (delta 0)
	To https://github.com/<your-github-username>/mynotebook.git
	 * [new branch]      master -> master
	Branch master set up to track remote branch master from origin.

4.	Select ``Create`` > ``Create Automated Build`` from Docker Hub.

- The system prompts you with a list of User/Organizations and code repositories. 

- For now select your GitHub account from the User/Organizations list on the left. The list of repositories change.

- Pick the project to build. In this case ``mynotebook``. Type in "Jupyter Test" in the Short Description box.

- If you have a long list of repos, use the filter box above the list to restrict the list. After you select the project, the system displays the Create Automated Build dialog.

|dockerhub_autobuilds|

.. Note::

	The dialog assumes some defaults which you can customize. By default, Docker builds images for each branch in your repository. It assumes the Dockerfile lives at the root of your source. When it builds an image, Docker tags it with the branch name.

5.	Customize the automated build by pressing the ``Click here to customize`` behavior link.

|auto_build-2.1|

Specify which code branches or tags to build from. You can build by a code branch or by an image tag. You can enter a specific value or use a regex to select multiple values. To see examples of regex, press the Show More link on the right of the page.

- Enter the ``master`` (default) for the name of the branch.

- Leave the Dockerfile location as is.

- Recall the file is in the root of your code repository.

- Specify ``1.0`` for the Tag Name.

6.	Click ``Create``.

.. important::

	During the build process, Docker copies the contents of your Dockerfile to Docker Hub. The Docker community (for public repositories) or approved team members/orgs (for private repositories) can then view the Dockerfile on your repository page.

	The build process looks for a README.md in the same directory as your Dockerfile. If you have a README.md file in your repository, it is used in the repository as the full description. If you change the full description after a build, it’s overwritten the next time the Automated Build runs. To make changes, modify the README.md in your Git repository.

.. warning:: 

	You can only trigger one build at a time and no more than one every five minutes. If you already have a build pending, or if you recently submitted a build request, Docker ignores new requests.

It can take a few minutes for your automated build job to be created. When the system is finished, it places you in the detail page for your Automated Build repository.

7. Manually Trigger a Build

Before you trigger an automated build by pushing to your GitHub ``mynotebook`` repo, you'll trigger a manual build. Triggering a manual build ensures everything is working correctly.

From your automated build page choose ``Build Settings``

|auto_build-5|

Press ``Trigger`` button and finally click ``Save Changes``.

.. Note::

	Docker builds everything listed whenever a push is made to the code repository. If you specify a particular branch or tag, you can manually build that image by pressing the Trigger. If you use a regular expression syntax (regex) to define your build branch or tag, Docker does not give you the option to manually build.

|auto_build-6|

8. Review the build results

The Build Details page shows a log of your build systems:

Navigate to the ``Build Details`` page.

Wait until your image build is done.

You may have to manually refresh the page and your build may take several minutes to complete.

|auto_build-7|

Exercise 1 (5-10 mins): Updating and automated building
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- ``git add``, ``commit`` and ``push`` to your GitHub or Gitlab repo
- Trigger automatic build with a new tag (2.0) on Docker Hub
- Pull your Docker image from Docker Hub to a new location.
- Run the instance to make sure it works


3.2 Bind mounts
^^^^^^^^^^^^^^^

When you run a container, you can bring a directory from the host system into the container, and give it a new name and location using the ``-v`` or ``--volume`` flag.

.. code-block:: bash

  $ mkdir -p ~/local-data-folder
  $ echo "some data" >> ~/local-data-folder/data.txt
  $ docker run -v ${HOME}/local-data-folder:/data $YOUR_DOCKERHUB_USERNAME/mynotebook:latest cat /data/data.txt
  
In the example above, you can mount a folder from your localhost, in your home user directory into the container as a new directory named ``/data``. 


3.3 Create and manage volumes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Unlike a bind mount, you can create and manage volumes outside the scope of any container.

A given volume can be mounted into multiple containers simultaneously. When no running container is using a volume, the volume is still available to Docker and is not removed automatically. You can remove unused volumes using ``docker volume prune`` command. 

When you create a Docker volume, it is stored within a directory on the Docker Linux host (``/var/lib/docker/`` 

.. Note::
  File location on Mac OS X is a bit different: https://timonweb.com/posts/getting-path-and-accessing-persistent-volumes-in-docker-for-mac/
  
Let's create a volume

.. code-block:: bash

	$ docker volume create my-vol

List volumes:

.. code-block:: bash

	$ docker volume ls

	local               my-vol

Inspect a volume by looking at the Mount section in the `docker volume inspect`

.. code-block:: bash

	$ docker volume inspect my-vol
	[
	    {
	        "Driver": "local",
	        "Labels": {},
	        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
	        "Name": "my-vol",
	        "Options": {},
	        "Scope": "local"
	    }
	]

Remove a volume

.. code-block:: bash

	$ docker volume rm my-vol
	$ docker volume ls


3.3.1 Populate a volume using a container
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This example starts an ``alpine`` container and populates the new volume ``output-vol`` with the some output created by the container.

.. code-block:: bash

	docker volume create output-vol
	docker run --name=data-app --mount source=output-vol,target=/data alpine sh -c 'env >> /data/container-env.txt'

Use ``docker inspect output-vol`` to see where the volume data lives on your host, and then use ``cat`` to confirm that it contains the output created by the container.

.. code-block:: bash
	
	docker volume inspect output-vol
	sudo cat /var/lib/docker/volumes/output-vol/_data/container-env.txt

You should see something like:

.. code-block::

	HOSTNAME=790e13bba28a
	SHLVL=1
	HOME=/root
	PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
	PWD=/

After running either of these examples, run the following commands to clean up the container and volume.

.. code-block:: bash

	docker rm data-app
	docker volume rm output-vol

3.4 Bind mounts
~~~~~~~~~~~~~~~

**Bind mounts:** When you use a bind mount, a file or directory on the host machine is mounted into a container. 

.. tip::

	If you are developing new Docker applications, consider using named **volumes** instead. You can’t use Docker CLI commands to directly manage bind mounts.

|bind_mount|

.. Warning:: 

	One side effect of using bind mounts, for better or for worse, is that you can change the host filesystem via processes running in a container, including creating, modifying, or deleting important system files or directories. This is a powerful ability which can have security implications, including impacting non-Docker processes on the host system.

	If you use ``--mount`` to bind-mount a file or directory that does not yet exist on the Docker host, Docker does not automatically create it for you, but generates an error.

3.2.1 Start a container with a bind mount
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a ``bind-data`` directory in your home directory.

.. code-block:: bash

	cd ~
	mkdir -p ~/bind-data

Run a container, mounting this directory inside the container, and the container should create some data in there.

.. code-block:: bash

	docker run --mount type=bind,source="$(pwd)"/bind-data,target=/data alpine sh -c 'env >> /data/container-env.txt'

Check that the output looks right.

.. code-block:: bash
	
	cat ~/bind-data/container-env.txt


3.4.1 Use a read-only bind mount
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For some development applications, the container needs to write into the bind mount, so changes are propagated back to the Docker host. At other times, the container only needs read access.

This example modifies the one above but mounts the directory as a read-only bind mount, by adding ``ro`` to the (empty by default) list of options, after the mount point within the container. Where multiple options are present, separate them by commas.

.. code-block:: bash

	docker run --mount type=bind,source="$(pwd)"/bind-data,target=/data,readonly alpine sh -c 'ls -al /data/ && env >> /data/container-env.txt'

You should see an error message about not being able to write to a read-only file system.

.. code-block:: bash

	sh: can't create /data/container-env.txt: Read-only file system


4. Docker Compose for multi-container apps
==========================================

**Docker Compose** is a tool for defining and running your multi-container Docker applications. 

Main advantages of Docker compose include:

- Your applications can be defined in a YAML file where all the options that you used in ``docker run`` are now defined (Reproducibility).
- It allows you to manage your application as a single entity rather than dealing with individual containers (Simplicity).

Let's now create a simple web app with Docker Compose using Flask (which you already seen before) and Redis. We will end up with a Flask container and a Redis container all on one host.

.. Note::

	The code for the above compose example is available `here <https://github.com/upendrak/compose_flask>`_

1. You’ll need a directory for your project on your host machine:

.. code-block:: bash

	$ mkdir compose_flask && cd compose_flask

2. Add the following to `requirements.txt` inside `compose_flask` directory:

.. code-block:: bash

	flask
	redis

3. Copy and paste the following code into a new file called `app.py` inside `compose_flask` directory:

.. code-block:: bash

	from flask import Flask
	from redis import Redis

	app = Flask(__name__)
	redis = Redis(host='redis', port=6379)

	@app.route('/')
	def hello():
	    redis.incr('hits')
	    return 'This Compose/Flask demo has been viewed %s time(s).' % redis.get('hits')

	if __name__ == "__main__":
	    app.run(host="0.0.0.0", debug=True)


4. Create a Dockerfile with the following code inside ``compose_flask`` directory:

.. code-block:: bash

	FROM python:2.7
	ADD . /code
	WORKDIR /code
	RUN pip install -r requirements.txt
	CMD python app.py

5. Add the following code to a new file, ``docker-compose.yml``, in your project directory:

.. code-block:: bash

	version: '2'
	services:
	    web:
	        restart: always
	        build: .
	        ports:
	            - "8888:5000"
	        volumes:
	            - .:/code
	        depends_on:
	            - redis
	    redis:
	        restart: always
	        image: redis

A brief explanation of ``docker-compose.yml`` is as below:

- ``restart: always`` means that it will restart whenever it fails.
- We define two services, **web** and **redis**.
- The web service builds from the Dockerfile in the current directory.
- Forwards the container’s exposed port (5000) to port 8888 on the host.
- Mounts the project directory on the host to /code inside the container (allowing you to modify the code without having to rebuild the image).
- ``depends_on`` links the web service to the Redis service.
- The redis service uses the latest Redis image from Docker Hub.

.. Note::

	Docker for Mac and Docker Toolbox already include Compose along with other Docker apps, so Mac users do not need to install Compose separately.
	Docker for Windows and Docker Toolbox already include Compose along with other Docker apps, so most Windows users do not need to install Compose separately.

	For Linux users 

	.. code-block:: bash

		sudo curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

		sudo chmod +x /usr/local/bin/docker-compose

5. Build and Run with ``docker-compose up -d`` command

.. code-block:: bash

	$ docker-compose up -d

	Building web
	Step 1/5 : FROM python:2.7
	2.7: Pulling from library/python
	f49cf87b52c1: Already exists
	7b491c575b06: Already exists
	b313b08bab3b: Already exists
	51d6678c3f0e: Already exists
	09f35bd58db2: Already exists
	f7e0c30e74c6: Pull complete
	c308c099d654: Pull complete
	339478b61728: Pull complete
	Digest: sha256:8cb593cb9cd1834429f0b4953a25617a8457e2c79b3e111c0f70bffd21acc467
	Status: Downloaded newer image for python:2.7
	 ---> 9e92c8430ba0
	Step 2/5 : ADD . /code
	 ---> 746bcecfc3c9
	Step 3/5 : WORKDIR /code
	 ---> c4cf3d6cb147
	Removing intermediate container 84d850371a36
	Step 4/5 : RUN pip install -r requirements.txt
	 ---> Running in d74c2e1cfbf7
	Collecting flask (from -r requirements.txt (line 1))
	  Downloading Flask-0.12.2-py2.py3-none-any.whl (83kB)
	Collecting redis (from -r requirements.txt (line 2))
	  Downloading redis-2.10.6-py2.py3-none-any.whl (64kB)
	Collecting itsdangerous>=0.21 (from flask->-r requirements.txt (line 1))
	  Downloading itsdangerous-0.24.tar.gz (46kB)
	Collecting Jinja2>=2.4 (from flask->-r requirements.txt (line 1))
	  Downloading Jinja2-2.10-py2.py3-none-any.whl (126kB)
	Collecting Werkzeug>=0.7 (from flask->-r requirements.txt (line 1))
	  Downloading Werkzeug-0.14.1-py2.py3-none-any.whl (322kB)
	Collecting click>=2.0 (from flask->-r requirements.txt (line 1))
	  Downloading click-6.7-py2.py3-none-any.whl (71kB)
	Collecting MarkupSafe>=0.23 (from Jinja2>=2.4->flask->-r requirements.txt (line 1))
	  Downloading MarkupSafe-1.0.tar.gz
	Building wheels for collected packages: itsdangerous, MarkupSafe
	  Running setup.py bdist_wheel for itsdangerous: started
	  Running setup.py bdist_wheel for itsdangerous: finished with status 'done'
	  Stored in directory: /root/.cache/pip/wheels/fc/a8/66/24d655233c757e178d45dea2de22a04c6d92766abfb741129a
	  Running setup.py bdist_wheel for MarkupSafe: started
	  Running setup.py bdist_wheel for MarkupSafe: finished with status 'done'
	  Stored in directory: /root/.cache/pip/wheels/88/a7/30/e39a54a87bcbe25308fa3ca64e8ddc75d9b3e5afa21ee32d57
	Successfully built itsdangerous MarkupSafe
	Installing collected packages: itsdangerous, MarkupSafe, Jinja2, Werkzeug, click, flask, redis
	Successfully installed Jinja2-2.10 MarkupSafe-1.0 Werkzeug-0.14.1 click-6.7 flask-0.12.2 itsdangerous-0.24 redis-2.10.6
	 ---> 5cc574ff32ed
	Removing intermediate container d74c2e1cfbf7
	Step 5/5 : CMD python app.py
	 ---> Running in 3ddb7040e8be
	 ---> e911b8e8979f
	Removing intermediate container 3ddb7040e8be
	Successfully built e911b8e8979f
	Successfully tagged composeflask_web:latest

And that’s it! You should be able to see the Flask application running on ``http://localhost:8888`` or ``<ipaddress>:8888``

|docker-compose|


.. code-block:: bash

	$ cat output.txt 
	Prediction of DecisionTreeClassifier:['apple' 'orange' 'apple']

.. |docker_image| image:: ../img/docker_image.png
  :width: 500
  
.. |dockerhub_create| image:: ../img/dockerhub_create.png
  :width: 800
  
.. |dockerhub_createrepo| image:: ../img/dockerhub_createrepo.png
  :width: 800
  
.. |dockerhub_autobuild| image:: ../img/dockerhub_autobuild.png
  :width: 800  

.. |dockerhub_autobuilds| image:: ../img/dockerhub_autobuilds.png
  :width: 800 
  
.. |dockerhub_buildsettings| image:: ../img/dockerhub_buildsettings.png
  :width: 800  

.. |private_registry| image:: ../img/private_registry.png
  :width: 500

.. |create_repo| image:: ../img/create_repo.png
  :width: 500 

.. |create_repo2| image:: ../img/create_repo2.png
  :width: 500 

.. |auto_build-1| image:: ../img/auto_build-1.png
  :width: 500

.. |auto_build-2| image:: ../img/auto_build-2.png
  :width: 500

.. |auto_build-2.1| image:: ../img/auto_build-2.1.png
  :width: 500

.. |auto_build-3| image:: ../img/auto_build-3.png
  :width: 500

.. |auto_build-4| image:: ../img/auto_build-4.png
  :width: 500

.. |auto_build-5| image:: ../img/auto_build-5.png
  :width: 500

.. |auto_build-6| image:: ../img/auto_build-6.png
  :width: 500

.. |auto_build-7| image:: ../img/auto_build-7.png
  :width: 500

.. |volumes| image:: ../img/volumes.png
  :width: 500

.. |bind_mount| image:: ../img/bind_mount.png
  :width: 500

.. |tmpfs| image:: ../img/tmpfs.png
  :width: 500

.. |docker-compose| image:: ../img/dc-1.png
  :width: 500

.. |jn_ss| image:: ../img/jn_ss.png
  :width: 500

.. |jn_login| image:: ../img/jn_login.png
  :width: 500

.. |jn_login2| image:: ../img/jn_login2.png
  :width: 500

.. |jn_login3.5| image:: ../img/jn_login3.5.png
  :width: 500

.. |jn_login3| image:: ../img/jn_login3.png
  :width: 500
 
.. |jn_login4| image:: ../img/jn_login4.png
  :width: 500

.. |rstudio_ss| image:: ../img/rstudio_ss.png
  :width: 500

.. |rstudio_login2| image:: ../img/rstudio_login2.png
  :width: 500
 
.. |rstudio_login| image:: ../img/rstudio_login.png
  :width: 500

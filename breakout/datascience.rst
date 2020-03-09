**Docker for Data Science**
--------------------------

For domain scientists (and budding data scientists), running a container already equipped with the libraries and tools needed for a particular analysis eliminates the need to spend hours debugging packages across different environments or configuring custom environments.

.. raw:: html

	<div style="width: 640px; height: 480px; margin: 10px; position: relative;"><iframe allowfullscreen frameborder="0" style="width:640px; height:480px" src="https://www.lucidchart.com/documents/embeddedchart/108d5703-5fc1-438b-bf7a-2be10662d1ab" id="wv5EGfbRDaCG"></iframe></div>

.. admonition:: Discussion Question

	Why Set Up a Data Science Software Environment in a Container?
	
	.. admonition:: answers

		- Speed. Docker containers allow a Jupyter or RStudio session to launch in seconds to minutes
		- Configuring environments can be a pain.
		- Standardize how data scientists work, and ensure that old code doesn’t stop running because of environment changes.
		- Containerization benefits both data science and IT/technical operations teams. 
		- Containers solve a lot of common problems associated with doing data science work at the enterprise level. 

Dealing with inconsistent package versions, having to dive through obscure error messages, and having to wait hours for packages to compile can be frustrating. This makes it hard to get started with data science in the first place, and is a completely arbitrary barrier to entry.

Thanks to the rich ecosystem of Docker users, there are readily available images for the common components in data science pipelines. 

Here are some Docker Images that may help you quickly configure your own data science pipeline:

- `MySQL <https://hub.docker.com/_/mysql/>`_
- `Postgres <https://hub.docker.com/_/postgres/>`_
- `Redmine <https://hub.docker.com/_/redmine/>`_
- `MongoDB <https://hub.docker.com/_/mongo/>`_
- `Hadoop <https://hub.docker.com/r/sequenceiq/hadoop-docker/>`_
- `Spark <https://hub.docker.com/r/sequenceiq/spark/>`_
- `Zookeeper <https://hub.docker.com/r/wurstmeister/zookeeper/>`_
- `Kafka <https://github.com/spotify/docker-kafka>`_
- `Cassandra <https://hub.docker.com/_/cassandra/>`_
- `Storm <https://github.com/wurstmeister/storm-docker>`_
- `Flink <https://github.com/apache/flink/tree/master/flink-contrib/docker-flink>`_
- `R <https://github.com/rocker-org/rocker>`_

How does all this stuff fit together??

.. raw:: html

	<div style="width: 640px; height: 480px; margin: 10px; position: relative;"><iframe allowfullscreen frameborder="0" style="width:640px; height:480px" src="https://www.lucidchart.com/documents/embeddedchart/41c634cf-f619-4556-847b-6cb894776aae" id="Ww5EhIAJKdHs"></iframe></div>

**Motivation:** Say you want to play around with some cool data science libraries in Python or R but what you don’t want to do is spend hours on installing Python or R, working out what libraries you need, installing each and every one and then messing around with the tedium of getting things to work just right on your version of Linux/Windows/OSX/OS9 — well this is where Docker comes to the rescue! With Docker we can get a Jupyter ‘Data Science’ notebook stack up and running in no time at all. Let’s get started! We will see few examples of these in the following sections...

1. Launch a Jupyter notebook conatiner
======================================

Docker allows us to run a ‘ready to go’ Jupyter data science stack in what’s known as a container:

.. code-block:: bash

	$ docker run --rm -p 8888:8888 jupyter/minimal-notebook

Once you’ve done that you should be greeted by your very own containerised Jupyter service!

|jn_login|

To create your first notebook, drill into the work directory and then click on the ‘New’ button on the right hand side and choose ‘Python 3’ to create a new Python 3 based Notebook. 

|jn_login2|

Now you can write your python code. Here is an example

|jn_login3|

|jn_login3.5|

To mount the host directory inside the Jupyter notebook container, you must first grant the within-container notebook user or group (NB_UID or NB_GID) write access to the host directory

.. code-block:: bash

	sudo chown 1000 <host directory>

you can run the command as below

.. code-block:: bash

	$ docker run --rm -p 8888:8888 -v $PWD:/work -w /home/jovyan/work jupyter/minimal-notebook

.. Tip::

	If you want to run `Jupyter-lab` instead of the default Jupyter notebook, you can do so by adding `jupyter-lab` at the end of the command.

More options for Datascience jupyter notebook - https://github.com/Paperspace/jupyter-docker-stacks/tree/master/datascience-notebook

To shut down the container once you’re done working, simply hit Ctrl-C in the terminal/command prompt. Your work will all be saved on your actual machine in the path we set in our Docker compose file. And there you have it — a quick and easy way to start using Jupyter notebooks with the magic of Docker.

2. Launch a RStudio container
=============================

Next, we will see a Docker image from Rocker which will allow us to run RStudio inside the container and has many useful R packages already installed.

|rstudio_ss|

.. code-block:: bash

	$ docker run --rm -d -e PASSWORD=rstudio1 -p 8787:8787 rocker/rstudio

The command above will lead RStudio-Server to launch invisibly. To connect to it, open a browser and enter http://localhost:8787, or <ipaddress>:8787 on cloud.

|rstudio_login2|

.. Tip::

	For the current Rstudio container, the default username is `rstudio` and the password is `rstudio1`. However you can override the disable the log-in with `-e DISABLE_AUTH=true` in place of `-e PASSWORD=rstudio1`. 

|rstudio_login|

If you want to mount the host directory inside the Rstudio container, you can do as below

.. code-block:: bash

	$ docker run -v $PWD:/data -w /data -p 8787:8787 -e DISABLE_AUTH=true --rm rocker/rstudio:3.6.2

And navigate to the `/data` inside the container using the file browser option in Rstudio.

An excellent R tutorial for reproducible research can be found `here <https://ropenscilabs.github.io/r-docker-tutorial/>`_

.. |jn_login| image:: ../img/jn_login.png
	:width: 700

.. |jn_login2| image:: ../img/jn_login2.png
	:width: 700

.. |jn_login3| image:: ../img/jn_login3.png
	:width: 700

.. |jn_login3.5| image:: ../img/jn_login3.5.png
	:width: 700

.. |rstudio_ss| image:: ../img/rstudio_ss.png
	:width: 700

.. |rstudio_login2| image:: ../img/rstudio_login2.png
	:width: 700

.. |rstudio_login| image:: ../img/rstudio_login.png
	:width: 700

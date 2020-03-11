**Containerized Workflows**
---------------------------

Workflow Management Using Snakemake
===================================

|snakemake|

In this breakout session you'll learn about `snakemake <https://snakemake.readthedocs.io/en/stable/>`_, a workflow management system consisting of a text-based workflow specification language and a scalable execution environment. You will be introduced to the Snakemake workflow definition language and how to use the execution environment to scale workflows to compute servers and clusters while adapting to hardware specific constraints. 

Snakemake is designed specifically for computationally intensive and/or complex data analysis pipelines. The name is a reference to the programming language Python, which forms the basis for the Snakemake syntax. 


SETUP
-----

.. Note:: 

- Right-Click the button below and login to CyVerse Discovery Environment for a quick launch of Snakemake VICE Jupyter lab app.
	
	|smake-vice|_

- To run Snakemake inside a docker container, run the following on your instance with docker installed:

.. code::

  docker run -it --entrypoint bash cyversevice/jupyterlab-snakemake
  
- Clone tutorial repository

.. code::  
  
  git clone https://github.com/NBISweden/workshop-reproducible-research.git
  
  cd workshop-reproducible-research/docker/
  
  git checkout devel
  
  ls
  
  snakemake -n



- Click `here <https://nbis-reproducible-research.readthedocs.io/en/latest/snakemake/>`_ for a Snakemake tutorial by `NBISweden <https://nbis-reproducible-research.readthedocs.io/en/latest/snakemake/>`_.

.. |snakemake| image:: ../img/snakemake.png
  :width: 700

.. |smake-vice| image:: https://de.cyverse.org/Powered-By-CyVerse-blue.svg
.. _smake-vice: https://de.cyverse.org/de/?type=quick-launch&quick-launch-id=7a62a49e-7fee-4822-b128-a1b2485e2941&app-id=9e989f50-6109-11ea-ab9d-008cfa5ae621

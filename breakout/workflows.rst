**Containerized Workflows**
---------------------------

Workflow Management Using Snakemake
===================================

|snakemake|

In this breakout session you'll learn about `snakemake <https://snakemake.readthedocs.io/en/stable/>`_, a workflow management system consisting of a text-based workflow specification language and a scalable execution environment. You will be introduced to the Snakemake workflow definition language and how to use the execution environment to scale workflows to compute servers and clusters while adapting to hardware specific constraints. 

Snakemake is designed specifically for computationally intensive and/or complex data analysis pipelines. The name is a reference to the programming language Python, which forms the basis for the Snakemake syntax. 

See `Snakemake Slides <https://slides.com/johanneskoester/snakemake-tutorial#/>`_

SETUP
-----

.. Note:: 

- Right-Click the button below and login to CyVerse Discovery Environment for a quick launch of Snakemake VICE Jupyter lab app.
	
	|smake-vice|_

- To run Snakemake inside a docker container, run the following on your instance with docker installed:

.. code::

  docker run -it --entrypoint bash cyversevice/jupyterlab-snakemake
  
- Clone RNAseq Snakemake tutorial repository

.. code::  
  
  git clone https://github.com/NBISweden/workshop-reproducible-research.git
  
  cd workshop-reproducible-research/docker/
  
  git checkout devel
  
  ls
  
- Dry-Run RNAseq Snakefile   
.. code::  
  
  snakemake -n

- Run RNAseq Snakefile   
.. code::  
  
  snakemake


**Why Snakemake**
-----------------

From where and how to get data for your analysis, to where and how to treat the outputs, workflow managers can help you achieve better scientific reproducibility and scalability. Once you learn to properly use Snakemake (or similar workflow management tools), keeping track of and sharing your work becomes second nature, not only saving you time whenever you need to re-run all or part of an analysis but helping you reduce errors that naturally get introduced whenever a non-automated activity is done (i.e., as part of the human condition of doing computational science and not being a bot!).

**Other Workflow Managers**
---------------------------

- `CCTools <https://cctools.readthedocs.io/en/latest/>`_ offers `Makeflow <https://cctools.readthedocs.io/en/latest/makeflow/>`_ a workflow management system similar to Snakemake and also `WorkQueue <https://cctools.readthedocs.io/en/latest/work_queue/>`_ for scalin-up through Distributed Computing. Read more `here <http://ccl.cse.nd.edu/software/tutorials/acic19/>`_.

- Click `here <https://nbis-reproducible-research.readthedocs.io/en/devel/snakemake/>`_ for a Snakemake tutorial by `NBISweden <https://nbis-reproducible-research.readthedocs.io/en/devel/>`_.

.. |snakemake| image:: ../img/snakemake.png
  :width: 700

.. |smake-vice| image:: https://de.cyverse.org/Powered-By-CyVerse-blue.svg
.. _smake-vice: https://de.cyverse.org/de/?type=quick-launch&quick-launch-id=7a62a49e-7fee-4822-b128-a1b2485e2941&app-id=9e989f50-6109-11ea-ab9d-008cfa5ae621

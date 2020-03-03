|CyVerse_logo2|_

**Deploying apps in CyVerse Discovery Environment**
---------------------------------------------------

The CyVerse `Discovery Environment (DE) <https://de.cyverse.org>`_ provides a simple yet powerful web portal for managing data, analyses, and workflows. The DE uses containers (both Docker and Singularity) to support customizable, non-interactive, interactive reproducible workflows using data stored in the CyVerse Data Store.

This `paper <https://f1000research.com/articles/5-1442/v3>`_ will guide you to bring your dockerized tools into CyVerse DE. 

|f1000|

.. Important ::

	Significant changes have been made as to how you can bring your tools into DE and so we are working on a separate paper that will show all those changes. Meanwhile you can follow the below tutorial for integrating your tools.

Here are the basic steps for deploying Docker images as apps in DE. For this tutorial I am going to show an example of `Tensor image classifier <https://github.com/upendrak/tensorflow_image_classifier>`_

- `Build and test your Docker images`_

- `Push your Docker image to Dockerhub`_

- `Add Docker images as tool in DE`_

- `Create an App UI for the tool in DE`_

- `Test the app using appropriate test data in DE`_

.. warning::

	If you already have your own Docker image or a Docker image of interest is already hosted on a public registry(s) (Dockerhub or quay.io or some other public repository), then you can skip to Step 3 

.. _Build and test your Docker images:

**1. Build and test your Docker images**

The first step is to dockerize your tool or software of interest. Detailed steps of how to dockerize your tool and test your dockerized images can be found in sections `intro to docker <../docker/dockerintro.html>`_ and `advanced docker <../docker/dockeradvanced.html>`_. 

For this tutorial I will use the ``tensorflow image classifier`` docker image that I built using this `code <https://github.com/upendrak/tensorflow_image_classifier>`_. 

**Building the Docker image from the Dockerfile**

.. code-block:: bash
	
	$ git clone https://github.com/upendrak/tensorflow_image_classifier && cd tensorflow_image_classifier

	$ docker build -t tensorflow_up:1.0 .

**Testing Docker image with test data**

.. code-block:: bash

	$ docker run --rm -v $(pwd):/data -w /data tensorflow_up:1.0 sample_data/16401288243_36112bd52f_m.jpg

This generates a file called `16401288243_36112bd52f_m.out` that consits of classification percentages such as 

.. code-block :: bash

	daisy (score = 0.99785)
	bee (score = 0.00009)
	speedboat (score = 0.00008)
	mitten (score = 0.00006)
	sulphur butterfly, sulfur butterfly (score = 0.00004)

.. _Push your Docker image to Dockerhub:

**2. Push your Docker image to public repositories**

Once the Docker image works as expected then either you set-up an automated build (recommended) or directly push the build Docker image to `dockerhub <http://hub.docker.com>`_. Here are the brief steps for automated build. See `Advanced Docker <../docker/dockeradvanced.html>`_ section for more details. 

**2.1.** Login to hub.docker.com and select Create Repository

|auto-1|

**2.2.** Give a name to the repository. In here, I have given `tensorflow_image_classifier` as the name

|auto-2|

**2.3.** Use the default visibility (Public in this case). Under Build settings, click the github octocat symbol which will ask you to authenticate github. Upon authentication, you'll be able to select the `tensorflow_image_classifier` github repo. Under Build rules, keep the source type as Branch, source as master, Docker Tag as 1.0 and the rest as defaults. Finally click "Create and Build" to start the building process

|auto-3|

**2.4.** It takes few minutes to hours (depending on the size of the image) and finally when everything works well, you'll see the **SUCCESS** message as shown here

|auto-4|

Here is the docker image built using automated build for the tensorflow image classifier on `Dockerhub <https://hub.docker.com/r/upendradevisetty/tensorflow_image_classifier>`_

.. _Add Docker images as tool in DE:

**3. Add Docker images as tool in DE**

All tools now run installed as Docker images in the DE. Once the software is dockerized and available as Docker images on dockerhub then you can add those docker images as a tool in DE.

.. warning::

	Check if the tool and correct version are already installed in the DE by following the steps below.

	- Log in to the Discovery Environment by going to https://de.cyverse.org/de/, entering your CyVerse username and password, and clicking LOGIN. If you have not already done so, you will need to sign up for a CyVerse account.
	- Click the ``Apps`` window to open the Apps window.
	- Click the ``Manage Tools`` button on the top-right of the Apps window.
	- In the search tools field, enter the first few letters of the tool name and then click enter.
	- If the tool is available then you can skip to skip to step 3 for creating a UI for that tool.

If the tool is not available in DE then do the following:

- Click open the ``Tools`` tab in ``Manage Tools`` window and then click ``Add tools`` button

- Then enter the fields about your tool and then click "Ok". 
	
	- Tool Name: It should be the name of the tool. For example "tensorflow_image_classifier".
	- Description: A short Description about the tool. For example "Tensorflow image classifier".
	- Version: What is the version number of the tool. For example "1.0".
	- Image name: Name of the Docker image on dockerhub or quay.io. For example "upendradevisetty/tensorflow_image_classifier".
	- Tag: What is the tag of your Docker image. This is optional but is highly recommended. If non specified, it will pull the default tag ``latest``. If the ``latest`` tag is not avaiable the tool integration will fail. For example "1.0"
	- Entrypoint: Do you want a entrypoint for your Docker image? This optional. 
	- Docker Hub URL: URL of the Dockerhub docker image. Option but is recommended. In this example "".

|img_building_1|

- If there is no error message, you have successfully integrated the tool.

.. _Create an App UI for the tool in DE:

**4. Create an App UI for the tool in DE**

Once the Dockerized tool is added, you can create the app UI for the tool. The ``Create App`` window consists of four distinct sections:

- The first section contains the different app items that can be added to your interface. To add an app item, select the one to use (hover over the object name for a brief description) and drag it into position in the middle section.
- The second section is the landing place for the objects you dragged and dropped from the left section, and it updates to display how the app will look when presented to a user.
- The third section (Details) displays all of the available properties for the selected item. As you customize the app in this section, the middle section updates dynamically so you can see how it will look and act.
- Finally, the fourth section at the bottom (Command line view) contains the command-line commands for the current item’s properties. As you update the properties in the Details section, the command-line view updates as well to let you make sure that you are passing the correct arguments in the correct order.

|img_building_4|

.. Note::

	Creating a new app interface requires that you know how to use the tool. With that knowledge, you create the interface according to how you want options to be displayed to a user. 

Here is an example of the ``Tensorflow image classifier - 1.0`` app UI in DE

|img_building_3|

.. _Test the app using appropriate test data in DE:

**5. Test the app using appropriate test data in DE**

After creating the new app according to your design, test your app in the Your Apps under development folder in the DE using appropriate test data to make sure it works properly.

For testing, we'll use the the same image that we used earlier.

|img_building_9|

1. First open the ``Tensorflow image classifier - 1.0`` app in the app window

|img_building_5|

2. Next browse the test file in the app and click launch analysis

|img_building_6|

3. After the analysis is completed, open the folder and check to see if the image classifier correctly predicts

|img_building_8|

Congrats!!! It works. The image classifier correctly predicts that the image is a daisy..

- If your app works the way you expect it to you can share your app or make the app public
- If your app doesn't work, then you may need to make changes to the app UI or you need to make changes to your Docker image. If you make changes to the Docker image, then you don't need to create a new app UI again as the Docker image updates will be propagated automatically.

.. |CyVerse_logo2| image:: ../img/cyverse_learning.png
    :width: 500
.. _CyVerse_logo2: http://learning.cyverse.org/

.. |f1000| image:: ../img/f1000.png
  :width: 700

.. |img_building_1| image:: ../img/img_building_1.png
  :width: 700

.. |img_building_4| image:: ../img/img_building_4.png
  :width: 700

.. |img_building_3| image:: ../img/img_building_3.png
  :width: 700

.. |img_building_5| image:: ../img/img_building_5.png
  :width: 700

.. |img_building_6| image:: ../img/img_building_6.png
  :width: 700

.. |img_building_8| image:: ../img/img_building_8.png
  :width: 700

.. |img_building_9| image:: ../img/img_building_9.png
  :width: 300

.. |auto-1| image:: ../img/auto-1.png
  :width: 700

.. |auto-2| image:: ../img/auto-2.png
  :width: 700

.. |auto-3| image:: ../img/auto-3.png
  :width: 700

.. |auto-4| image:: ../img/auto-4.png
  :width: 700

Composing AI Inference pipelines with Node-RED & Flowfuse
=========================================================

.. dropdown:: :fab:`youtube;youtube-icon` ㅤCreate a pipeline with Flowfuse

   .. raw:: html

      <div style="position: relative; padding-bottom: 56.25%; margin-bottom: 2em; height: 0; overflow: hidden; max-width: 100%; height: auto;">
        <iframe src="https://www.youtube.com/embed/9a019SA5GW4" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
      </div>

   :material-outlined:`error;1.5em` Please, be aware that video demos can become quickly outdated. In case of doubt, always refer to the written documentation.

.. admonition:: Deprecation notice
    :class: info

    *  **OSCAR v3.6.5** now provides support for deploying a Node-RED instance. More info regarding this new OSCAR feature can be found `here <https://docs.oscar.grycap.net/latest/integration-node-red/>`__ .
    * The Flowfuse instance of the project is **no longer available**. You can test the examples directly in OSCAR, deploying your own Node-RED instance with OSCAR nodes already integrated.
    * The current version of the videotutorial will be updated soon. Stay tuned!


In this document, we will learn about Composing AI Inference pipelines based on OSCAR
services with Node-RED, specifically:

* how to create inference services in OSCAR,
* how to create a new "Flows" instance (based on Node-RED) in the OSCAR platform for AI4EOSC,
* how to access the instance and create our first application workflow,
* how to deploy a workflow to call inference services deployed in OSCAR,
* how to delete the Node-RED instance.

For a complete overview of the Node-RED examples, please refer to the `GitHub README <https://github.com/ai4os/ai4-compose/blob/main/node-red/README.md>`__.

First of all, let's understand the key technologies involved in this tutorial: 

* `Node-RED <https://nodered.org/>`__ is an open-source visual programming tool.
  Built on Node.js, it allows users to create event-driven systems by connecting nodes
  representing different functionalities. With a user-friendly web interface and a rich
  library of pre-built nodes, Node-RED simplifies the visual composition of pipelines.
* `OSCAR <https://oscar.grycap.net/>`__ is an open-source serverless platform to support
  scalable event-driven computations. From **OSCAR v3.6.5**, it provides support for deploying
  a Node-RED instance. More info regarding this new OSCAR feature can be found `here <https://docs.oscar.grycap.net/latest/integration-node-red/>`__ .

In AI4OS, we use Node-Red to visually compose AI model inference pipelines.
Specific custom nodes have been created to perform AI model inference on remote
OSCAR clusters.

1. Creating inference services in OSCAR
---------------------------------------
Let's start by creating an OSCAR service to use it from Node-RED.

1.1. Deploy YOLOv8 service
^^^^^^^^^^^^^^^^^^^^^^^^^^

Go to ``OSCAR Dashboard`` <https://dashboard.oscar.grycap.net/>`__ and,
in the ``Services`` panel, select ``Create service -> FDL``. Use the
following configuration:

FDL:

.. code:: yaml

   functions:
     oscar:
     - oscar-cluster:
         name: yolov8-node-red
         memory: 4Gi
         cpu: '2.0'
         image: ai4oshub/ai4os-yolov8-torch:latest
         script: script.sh
         log_level: CRITICAL

Script:

.. code:: bash

   #!/bin/bash
   RENAMED_FILE="${INPUT_FILE_PATH}.png"
   mv "$INPUT_FILE_PATH" "$RENAMED_FILE"
   OUTPUT_FILE="$TMP_OUTPUT_DIR/output.png"
   deepaas-cli --deepaas_method_output="$OUTPUT_FILE" predict --files "$RENAMED_FILE" --accept image/png 2>&1
   echo "Prediction was saved in: $OUTPUT_FILE"

See the our documentation on :doc:`how to make an inference in OSCAR </howtos/deploy/oscar>`
for more information about creating OSCAR service.

2. Creating our Node-RED instance in OSCAR
------------------------------------------
1. In the ```OSCAR dashboard`` <https://inference.cloud.ai4eosc.eu/ui/>`__,
   go to the ``Flows`` panel and then click ``New``.

   .. image:: /_static/images/flows/node-red-deployed.png
      :alt: node-red-deployed.png

2. Enter the **admin** password that you will be asked to access later
   on this instance of Node-RED, and select or create a Bucket.

   .. image:: /_static/images/flows/node-red-dashboard.png
      :alt: node-red-dashboard.png

3. After deploying Node-RED, we access its user interface.

   .. image:: /_static/images/flows/node-red-dashboard-visit.png
      :alt: node-red-dashboard-visit.png


3. Accessing the Node-RED instance
----------------------------------
Once the Node-RED instance is up and running, you can log in with your credentials
(the user is always **admin**).

.. image:: /_static/images/flows/node-red-login.png
   :alt: node-red-login.png


4. Creating a workflow in Node-Red
----------------------------------

4.1. Designing and creating the workflow
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Let’s create a workflow that fetches an image from the Internet, makes a
request to the YOLO service and visualizes the result.

We need the following list of components from the Node-RED sidebar menu:

- **Common** → ``inject`` node
- **Network** → ``HTTP request`` node
- **Output** → ``image`` node
- **OSCAR** → ``OSCAR YOLO8`` node

.. figure:: /_static/images/flows/node-red-nodes.png
   :alt: node-red-nodes.png

Drag and drop the boxes to the canvas and then connect the components as
shown:

.. figure:: /_static/images/flows/node-red-workflow.png
   :alt: node-red-workflow.png

Now we need to configure the components. To configure the *HTTP request
node* double-click on it:

- **URL**: URL of an image you want to analyze with YOLO (for example,
  you can use this
  ```image`` <https://upload.wikimedia.org/wikipedia/commons/thumb/1/15/Cat_August_2010-4.jpg/640px-Cat_August_2010-4.jpg>`__)
- **Payload**: *Send as request body*
- **Return**: *A binary buffer*

.. figure:: /_static/images/flows/node-red-node-http-request.png
   :alt: node-red-http-request.png

Configure the ``OSCAR YOLO8`` node:

- **Server**: URL of the OSCAR cluster. You can get it from
  ```OSCAR dashboard`` <https://dashboard.oscar.grycap.net/>`__ → *Info*
  (Sidebar panel) → *Endpoint*
- **Service** Name: *yolov8-node-red*
- **Token**: Obtain the token from
  ```OSCAR dashboard`` <https://dashboard.oscar.grycap.net/>`__ → *Info*
  (Sidebar panel) → *Access token*

.. figure:: /_static/images/flows/node-red-node-oscar-yolo.png
   :alt: node-red-node-oscar-yolo.png

4.2. Testing the workflow
^^^^^^^^^^^^^^^^^^^^^^^^^
After configuring your workflow, you can test it in the Node-RED Editor:

Click *Deploy* (top right corner) and then click on the *inject* node:

.. figure:: /_static/images/flows/node-red-workflow-run.png
   :alt: node-red-workflow-run.png

You should see the result as indicated below.

.. figure:: /_static/images/flows/node-red-workflow-result.png
   :alt: node-red-workflow-result.png


5. How to delete a Node-Red instance
------------------------------------

To delete an instance, you have to click on the ``Delete`` button and confirm the operation.

   .. image:: /_static/images/flows/node-red-dashboard-delete.png
      :alt: node-red-dashboard-delete.png

   .. image:: /_static/images/flows/node-red-delete.png
      :alt: node-red-delete.png

The MinIO bucket remains available; however, it is a good practice to ensure that you have backed up any important data or configurations before
deleting an instance. 

6. Other application examples
-----------------------------

6.1 Toy workflow: OSCAR Cowsay
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We have now seen how to create an application, deploy a Node-RED instance,
and connect to it. Next, we will proceed to create a workflow to demonstrate the
functionality of the Node-RED tool.

For this first  toy example, we will use a module that takes text as input and returns an
ASCII art of a cow repeating the same text as output.

To set up this example, we will essentially need three nodes:
the Inject node, the OSCAR Cowsay node, and the Debug node.
The Debug node is used to visualize the result in the debug log.

To place the modules in the workspace, simply drag them from the left-hand side menu.
And finally, we connect the inputs and outputs of the modules as shown in the figure.

.. image:: /_static/images/flows/20.png

Once we have deployed the workflow, we need to configure each module.

For the Inject node, as shown in the figure, there are default parameters.
For the cowsay example, it is necessary to remove the topic since it will not be used.
Additionally, change the type of `msg.payload` to string and enter the desired text in
the box, in this case: ``Hello World!``

.. image:: /_static/images/flows/21.png
   :width: 800px

For the OSCAR Cowsay node, we need to select the endpoint of the OSCAR cluster we will
use and enter it in the ``Server`` section.
Additionally, we will select the name of the service in the cluster and enter the token.

.. image:: /_static/images/flows/22.png
   :width: 800px

For this example, we will use the endpoint ``https://inference.cloud.ai4eosc.eu``.
Additionally, to locate the service token, we just need to expand the details of
the service. (Remember: to access the platform, you need to have an :doc:`EGI account </reference/user-access-levels>`.)

.. image:: /_static/images/flows/23.png

Finally, the Debug node does not require any additional configurations,
so we can click on the ``Deploy`` button.
This will save the workflow, and it will be possible to start it.

.. image:: /_static/images/flows/24.png

Now, to start the workflow after deploying, you need to click on the small square next
to the Inject node on the left side. This will initiate the workflow and input the
string into the next node. After invoking the cowsay service, it will return the
modified cowsay string as output, which can be viewed in the debug window thanks to
the Debug node.

We have finished implementing the first workflow using an OSCAR node.

.. image:: /_static/images/flows/25.png

6.2 Plant Classification workflow with input preprocessing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In this section, we will compose an example workflow for AI inference where
we will convert the color image of a plant to black and white and then classify
the plant to determine its species.

.. image:: /_static/images/flows/26.png
   :width: 600px

If we have started an instance with the OSCAR Node-RED template, we can use the
preconfigured modules of some OSCAR services.
To find them, we just need to go to the OSCAR section in the left side menu of Node-RED.

* **Node HTTP Request** is designed to execute an HTTP request to retrieve an image
  from a specified URL, which is provided as input. Once the image is downloaded,
  it becomes the output of this node.
* **Node OSCAR Grayify**, receives the image from the previous node as its input.
  Its primary function is to process the image to convert it into grayscale.
  After this, the processed image is sent to the OSCAR cluster for appropriate processing.
  The result from this node is the original image converted to grayscale, which is provided as output.
* **Node OSCAR Plants Classification** takes the grayscale image processed by Node 2 as
  input. This node is responsible for classifying the plant in the image using the OSCAR
  cluster. After processing, the node produces an output in JSON format, containing
  detailed information about the plant classification.

This processing sequence ensures a coherent and efficient workflow, optimizing image
classification through the integration of advanced technologies in each node.

Once the pipeline is organized, we will start configuring the components. To begin:

* The inject node does not need to be modified, since it is used to start the pipeline.
* The image preview nodes and the debug node should also not be modified.
* The http request node: set the method to GET, enter the image URL (for
  example: ``https://blog.agroterra.com/wp-content/uploads/2013/09/trigo-570x288.jpg``),
  configure the payload to be sent as a request body, and set the return to be a binary buffer.

.. image:: /_static/images/flows/27.png
   :width: 600px

Finally, we need to edit the OSCAR nodes, which have three fields, in the same way
we did in the Cowsay example.

.. image:: /_static/images/flows/28.png

If the result of Plant Classification appears as a buffer, you just need to select
the option to view the result in raw, allowing you to read the information correctly.

.. image:: /_static/images/flows/29.png
   :width: 600px


6.3 Importing flows from Github
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now we will explain how to, step by step, recreate usage examples for OSCAR by
importing them from the GitHub repository.
In this case we will look up for the cowsay example.

First install the dependencies `described here <https://github.com/ai4os/ai4-compose/tree/main/node-red>`__.
Then, access to the `node-red repo <https://github.com/ai4os/ai4-compose/tree/main/node-red/modules>`__ and,
in this example, look for the ``grayify.json``.

.. image:: /_static/images/flows/30.png

.. image:: /_static/images/flows/31.png

.. image:: /_static/images/flows/32.png

.. image:: /_static/images/flows/33.png

Then, to import flows/subflows/nodes/examples in our node red instance, we can expand
the hamburger menu located in the top right corner and look for the fourth option:
``Import``.
Once this option is selected, a floating menu will appear where we can paste the JSON.

.. image:: /_static/images/flows/34.png

6.4 Importing modules via the Node-RED palette
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the case of importing other types of modules or nodes, we can expand the same menu,
but now we will go to the ``Manage palette`` option, which allows us to import from
the module installation menu.

Once in the ``Manage palette`` menu, you can search for the desired modules or nodes
and install them directly.
Ensure that the modules or nodes you're installing are compatible with your version of
Node-RED and come from trusted sources to maintain the integrity and security of your
environment.

After installation, it's good practice to test the new modules or nodes to ensure they
work as expected.

.. image:: /_static/images/flows/35.png

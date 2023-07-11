.. _apps_deploy:

Deploying An Application
===============================
We're going to take some of what we've learned from best practices and put it into, well, practice.
To deploy an app, we'll build a docker container, push it to dockerhub,
publish an app definition to TAPIS.
First we'll build a container:
::
  docker build -t $DOCKER_USERNAME/$DOCKER_REPO:$DOCKER_TAG -f Dockerfile
  docker push $DOCKER_USERNAME/$DOCKER_REPO:$DOCKER_TAG
  # or use buildx for macs with an M1 chip:
  docker buildx build --platform linux/amd64 --push -t $DOCKER_USERNAME/$DOCKER_REPO:$DOCKER_TAG -f Dockerfile . 



Setup Tapipy Client
---------------------
If you haven't already, let's go ahead and install the TAPIS CLI on your host system:
::
  pip install tapipy

And the open up python and instantiate a client:
::
  # Import the Tapis object
  from tapipy.tapis import Tapis

  # Log into you the Tapis service by providing user/pass and the base url of your tenant. For example, to interact with the tacc tenant --
  t = Tapis(base_url='https://tacc.tapis.io',
            username='myuser',
            password='mypass')
      
  # Get tokens that will be used for authentication function calls
  t.get_tokens()
 
  # Push your credentials to the public "frontera" system
  t.systems.createUserCredential(systemId='frontera', userName='username', password='password')
  

To check Tapis is setup correctly, you can run:
::
  t.systems.getSystems()
  t.files.listFiles(systemId='frontera', path='.')

and you should see the frontera system, and files present in the root directory.


Copy an Application from Github
---------------------
Let's create a Tapis app to perform some analysis.
For this example we'll create a fastqc application that is triggered when ``.fastq``
files are uploaded to a certain directory, but you can use any application or file type
for this.

You can clone the fastqc example app from here:
::
  git clone https://github.com/JoshuaUrrutia/automation_resources_2023.git
  cd automation_resources_2023/fastqc_app



Build the Container
----------------------------
Remember the storage system we created last week?

.. literalinclude:: assets/storage.json
  :linenos:
  :emphasize-lines: 14,15


Build and push the container:
::
  docker build -t $USERNAME/fastqc_app:3.0.0 .
  docker push $USERNAME/fastqc_app:3.0.0
  # or on a M1 chip
  docker buildx build --platform linux/amd64 -t $USERNAME/fastqc_app:3.0.0 --push .

And look for the ``"homeDir"`` key in the json response:
::
  "homeDir": "/work/05369/urrutia/stampede2/"

Ok and now we'll create a directory called `apps` where we'll store all our app bundles.
::
  # tapis files mkdir agave://urrutia.stampede2.storage/work/05369/urrutia/stampede2/ apps
  tapis files mkdir agave://$USERNAME.stampede2.storage/$HOME_DIR apps

Edit the app.ini file
----------------------------
Replace the docker username and ``storage_path`` in the ``app.ini``, with your docker username
and your ``homeDir`` (the location on your storage system where you have write access).

.. literalinclude:: assets/app.ini
   :linenos:
   :emphasize-lines: 7,15

The contents of the ``app.ini`` file will be injected into your app definition (``app.json``):

.. literalinclude:: assets/app.json
   :linenos:
   :emphasize-lines: 7,38

Deploy the Application
----------------------------
All that's left now is to deploy the application from the FastQC repo:
::
  tapis apps deploy

Which should print out a table like this:
::
  +--------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | stage  | message                                                                                                                                                                                     |
  +--------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
  | build  | Step 1/4 : FROM python:3.8                                                                                                                                                                  |
  | build  |  ---> ea8c3fb3cd86                                                                                                                                                                          |
  |        |                                                                                                                                                                                             |
  | build  | Step 2/4 : RUN apt-get update     && apt-get upgrade -y     && apt-get install wget -y     && apt-get install zip -y     && apt-get install default-jre -y                                  |
  | build  |  ---> Using cache                                                                                                                                                                           |
  |        |                                                                                                                                                                                             |
  | build  |  ---> f0f2bd1f3194                                                                                                                                                                          |
  |        |                                                                                                                                                                                             |
  | build  | Step 3/4 : RUN wget https://www.bioinformatics.babraham.ac.uk/projects/fastqc/fastqc_v0.11.9.zip     && unzip fastqc_v0.11.9.zip     && rm fastqc_v0.11.9.zip     && chmod +x FastQC/fastqc |
  | build  |  ---> Using cache                                                                                                                                                                           |
  |        |                                                                                                                                                                                             |
  | build  |  ---> 3bea8add49b6                                                                                                                                                                          |
  |        |                                                                                                                                                                                             |
  | build  | Step 4/4 : ENV PATH "/FastQC/:$PATH"                                                                                                                                                        |
  | build  |  ---> Using cache                                                                                                                                                                           |
  |        |                                                                                                                                                                                             |
  | build  |  ---> cfafe349377a                                                                                                                                                                          |
  |        |                                                                                                                                                                                             |
  | build  | Successfully built cfafe349377a                                                                                                                                                             |
  |        |                                                                                                                                                                                             |
  | build  | Successfully tagged jurrutia/fastqc_app:0.11.9                                                                                                                                              |
  |        |                                                                                                                                                                                             |
  | push   | The push refers to repository [docker.io/jurrutia/fastqc_app]                                                                                                                               |
  | push   | 0.11.9: digest: sha256:4ee48dae892538f83b69d6a1a7dbf099c51d3d032e44d0241518984897b5274f size: 2642                                                                                          |
  | upload | assets/runner-template.sh                                                                                                                                                                   |
  | upload | assets/tester.sh                                                                                                                                                                            |
  | upload | assets/_lib/CONTAINER_IMAGE                                                                                                                                                                 |
  | upload | assets/_lib/extend-runtime.sh                                                                                                                                                               |
  | create | Created Tapis app urrutia-fastqc-0.11.9 revision 1                                                                                                                                          |
  +--------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

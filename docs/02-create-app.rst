.. _apps_deploy:

Deploying An Application
===============================
We're going to take some of what we've learned from best practices and put it into, well, practice.
Apps deploy is a CLI command that will build a docker container, push it to dockerhub,
upload your app asset bundle to a deploymentSystem, and register your app on an executionSystem.
Apps deploy is a single command that replaces:
::
  docker build -t $DOCKER_USERNAME/$DOCKER_REPO:$DOCKER_TAG -f Dockerfile
  docker push $DOCKER_USERNAME/$DOCKER_REPO:$DOCKER_TAG
  tapis files upload agave://$DEPLOYMENT_SYSTEM/$DEPLOYMENT_PATH/ runner.sh
  tapis apps create -F app.json

And we plan on adding even more in the future! Namely an automatic upload to GitHub
so there's a source controlled snapshot of each deployment.

Setup Tapis CLI
---------------------
Let's go ahead and install the TAPIS CLI on your host system:
::
  pip install tapis-cli

And re-run:
::
  tapis auth init

Or, if you want to be clever, move over the authentication directory we created last week:
::
  cp -R ~/.tapis ~/.agave

`Docker CLI Tangent <02-docker-tangent.html>`_
---------------------

You might be wondering: "Can I just re-use the TAPIS container the same way we did last week?".
And yes, you can, but there are some caveats. `See this tangent <02-docker-tangent.html>`_ for more info.

Copy an Application from Github
---------------------
Let's create a Tapis app to perform some analysis.
For this example we'll create a fastqc application that is triggered when .fastq
files are uploaded to a certain directory, but you can use any application or file type
for this.

You can clone the fastqc example app from here:
::
  git clone https://github.com/JoshuaUrrutia/fastqc_app.git
  cd fastqc_app

Or, if you'd like, you're welcome to use application that was created last week:
https://tacc.github.io/summer-institute-2020-tapis/block2/apps/

And later modify the reactor in the later steps to trigger off of a image file, instead
of a .fastq file.

Find deploymentPath
----------------------------
Remember we created a storage system last week?

.. literalinclude:: assets/storage.json
  :linenos:
  :emphasize-lines: 14,15

Our write operations will be on to the ``rootDir`` above. If you plan on deploying
lots of apps, it's a good idea to redefine the rootDir on your system to be a directory
where you have write access, for example replacing the rootDir with your
homeDir: ``/work/dir../UPDATEUSERNAME/stampede2``. This will simplify the
structure of your app.ini file, and you won't have to lookup
or remember your directory number when listing and uploading files.

But, to save time, since this system is already created, we'll just grab the
absolute path to the directory where we have write access.

To get a full listing of your system, run:
::
  tapis systems show -f json $USERNAME.stampede2.storage

And look for the ``"homeDir"`` key in the json response:
::
  "homeDir": "/work/05369/urrutia/stampede2/"

Ok and now we'll create a directory called `apps` where we'll store all our app bundles.
::
  #tapis files mkdir agave://urrutia.stampede2.storage/work/05369/urrutia/stampede2/ apps
  tapis files mkdir agave://$USERNAME.stampede2.storage/$HOME_DIR apps

Edit the app.ini file
----------------------------
Replace the docker username and storage_path in the app.ini, with your docker username
and your ``homeDir`` (the location on your storage system where you have write access).

.. literalinclude:: assets/app.ini
   :linenos:
   :emphasize-lines: 7,15

The contents of the app.ini file will be injected into your app definition (``app.json``):

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

Running Apps Deploy
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

Alternatively you can re-use the TAPIS container the same way we did last week:
::
 docker run --rm -it \
            -v ${PWD}:/work \
            -v ${HOME}/.agave:/root/.agave \
            -v /var/run/docker.sock:/var/run/docker.sock \
            tacc/tapis-cli:latest \
            bash

--rm                             Automatically remove the container when it exits
-i, --interactive                    Keep STDIN open even if not attached
-t, --tty                            Allocate a pseudo-TTY
-v, --volume list                    Bind mount a volume

But I wouldn't recommend this for several reasons.
First you will be limited to writing only to subdirectories under your current directory $PWD,
because of the local:container volume mount specified in ``-v ${PWD}:/work``.

Moreover, because part of the deploy process is to build a docker container, we would
be building a container inside of a container. Which could be useful in some cases, but
is somewhat contrived.
`Docker in Docker <https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/>`_
To avoid this we can just mount our system docker into our container, so we're
using the same docker engine to build the container inside our container.
Notice there is an extra volume mount ``/var/run/docker.sock:/var/run/docker.sock``.
So our system docker engine is mounted inside the container.

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
structure of your project.ini file, and you won't have to lookup
or remember your directory number.

But, to save time, since this system is already created, we'll just grab the
absolute path to the directory where we have write access.

To get a full listing of your system, run:
::
  tapis systems show -f json $USERNAME.stampede2.storage

And look for the ``"homeDir"`` key in the json response:
::
  "homeDir": "/work/05369/urrutia/stampede2/"

Ok and now we'll create a directory called `apps` where we'll store all our app bundles.
```
# tapis files mkdir agave://urrutia.stampede2.storage/work/05369/urrutia/stampede2/ apps
tapis files mkdir agave://$USERNAME.stampede2.storage/$HOME_DIR apps

Edit the .ini file
----------------------------
Replace the docker username and storage_path in the project.ini, with your docker username
and your `homeDir` (the location on your storage system where you have write access).

.. literalinclude:: assets/project.ini
   :linenos:
   :emphasize-lines: 7,15

Deploy the Application
----------------------------
Deploy the application from the fastqc repo:
::
  tapis apps deploy

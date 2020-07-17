Deploying an Actor
=======================

What is a reactor? See more info in our documentation:
  * `Abaco documentation <https://tacc-cloud.readthedocs.io/projects/abaco/en/latest/>`_
  * `Abaco swagger guide <https://tacc.github.io/abaco-live-docs/>`_

It's a script, that lives in the cloud, and does something for you. It's not
for compute intensive jobs, that's what apps are for, it's designed to be quick,
responsive, and lightweight.

We're going deploy a reactor that will receive a notification when a file is uploaded,
create a Tapis job.json, and submit it to our FastQC application.

Copy a Reactor from Github
----------------------------

Clone a Abaco reactor I created to submit FastQC jobs:
::
  git clone https://github.com/JoshuaUrrutia/fastqc_router_reactor.git

Vignette: create a reactor from scratch with ``tapis abaco init`` (not implemented)


Edit actor.ini and config.yml
---------------------------------
We'll need to make edits to actor.ini so that it points to your dockerhub username:

.. literalinclude:: assets/actor.ini
   :linenos:
   :emphasize-lines: 7

And change the name of the app in config.yml, so it matches your app id:

.. literalinclude:: assets/config.yml
  :linenos:
  :emphasize-lines: 6,16

Now we create an empty secrets.json file. It's just empty in this example, but
if you had passwords or credentials you wanted to be available in your actor, you
could add those to the secrets.json.
::
  cp secrets.json.sample secrets.json

Deploy the Actor
---------------------------------
All that's left is to deploy our reactor:
::
  tapis actors deploy

You should see a response like:
::
  Building jurrutia/fastqc_router:0.1
  Finished (27932 msec)
  Pushing jurrutia/fastqc_router:0.1
  Finished (9354 msec)
  +--------+-------------------------------------------------------------------------------------------------+
  | stage  | message                                                                                         |
  +--------+-------------------------------------------------------------------------------------------------+
  | build  | Step 1/1 : FROM jurrutia/reactors:python2-edge                                                  |
  | build  | # Executing 5 build trigger                                                                     |
  | build  | s                                                                                               |
  | build  |  ---> Running in 34bf66e2e455                                                                   |
  |        |                                                                                                 |
  | build  | You must give at least one requirement to install (see "pip help install")                      |
  |        |                                                                                                 |
  | build  | Removing intermediate container 34bf66e2e455                                                    |
  |        |                                                                                                 |
  | build  |  ---> ca9ae97aef39                                                                              |
  |        |                                                                                                 |
  | build  | Successfully built ca9ae97aef39                                                                 |
  |        |                                                                                                 |
  | build  | Successfully tagged jurrutia/fastqc_router:0.1                                                  |
  |        |                                                                                                 |
  | push   | The push refers to repository [docker.io/jurrutia/fastqc_router]                                |
  | push   | 0.1: digest: sha256:844f0ce2de5e03f1f15fedb64b7f5354bf64da453a18c87c6cb5c9981e6e8991 size: 6978 |
  | create | Created Tapis actor X4blX3Ez65qQZ                                                               |
  | cache  | Cached actor identifier to disk                                                                 |
  +--------+-------------------------------------------------------------------------------------------------+

Copy your actor id (``X4blX3Ez65qQZ`` in the above example).
If you forget the id, you can always list out your actors with ``tapis actors list``.

Create a FastQC Folder
---------------------------------
Now we'll create the ``fastqc`` folder on our storage system. After we create our
notification, any file that is uploaded here will be analyzed automatically by
our FastQC app!
::
  #tapis files mkdir agave://urrutia.stampede2.storage/work/05369/urrutia/stampede2 fastqc
  tapis files mkdir agave://$USERNAME.stampede2.storage/$HOME_DIR fastqc

Create File System Notifications
---------------------------------
Now you’re ready to create a file system notification.
This notification will pass a message to the fastqc_router_reactor when a file
is uploaded to the fastqc directory on your storage system. The fastqc_router_reactor takes
this notification, crafts a job.json, and submits a job to the fastqc_app.
We’ve created a python wrapper to help setup the file system notifications,
you can download the python scripts here:
::
  git clone https://github.com/JoshuaUrrutia/abaco_notifications.git

From the abaco_notifications directory, you can run add_notify_reactor.py to
setup a notification. For example:
::
  #python add_notify_reactor.py urrutia.stampede2.storage /work/05369/urrutia/stampede2/fastqc X4blX3Ez65qQZ
  python add_notify_reactor.py $AGAVE_SYSTEM_NAME $PATH_TO_DIRECTORY $ACTOR_ID

If it runs successfully your response should look like:
::
  assocationIds = 8216966626126028310-242ac112-0001-002
  notification id: 18251060861323945066-242ac116-0001-011
  notification url: https://portals-api.tacc.utexas.edu//actors/v2/X4blX3Ez65qQZ/messages?x-nonce=PORTALS_basEq8g5oylx


Upload and Test
---------------------------------
Now the only thing left to do is to test and see if our
``upload -> notification -> reactor -> app``
chain is functioning.

Upload a fastq file to your FastQC directory (you can find a copy of this file in
the ``fastqc_app/test/`` repo):
::
  tapis files upload agave://$SYSTEM/$PATH/ $FILE
  tapis files upload agave://urrutia.stampede2.storage/work/05369/urrutia/stampede2/fastqc reads1.fastq.gz

Now that we've uploaded lets see if our actor was triggered:
::
  tapis actors exces list $ACTOR_ID

The response should look like:
::
  urrutia$ tapis actors execs list X4blX3Ez65qQZ
  +---------------+----------+
  | executionId   | status   |
  +---------------+----------+
  | AqDao7YgEYZ6Z | COMPLETE |
  +---------------+----------+

If you want to see the logs from your actor execution you can run:
::
  tapis actors execs logs $ACTOR_ID $EXECUTION_ID

Finally, let's check to see if a job was submitted to our application:
::
  tapis jobs list
  +------------------------------------------+--------------------------------+----------+
  | id                                       | name                           | status   |
  +------------------------------------------+--------------------------------+----------+
  | 485458bc-335d-4d05-ae30-70de2583b6d5-007 | fastqc_test                    | FINISHED |
  +------------------------------------------+--------------------------------+----------+
And go ahead and download the outputs of that job:
::
  #tapis jobs outputs download 485458bc-335d-4d05-ae30-70de2583b6d5-007
  #cd 485458bc-335d-4d05-ae30-70de2583b6d5-007
  tapis jobs outputs download $JOB_ID
  cd $JOB_ID
  open reads1_fastqc.html

Congratulations, you successfully automated part of your workflow with Tapis!
But there's not reason to stop here, you can add a notification to your FastQC jobs
to trigger a new reactor (to perform an alignment maybe?), and build an entirely
automated workflow by chaining together reactors and apps.

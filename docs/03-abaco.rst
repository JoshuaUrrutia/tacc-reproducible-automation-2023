Deploying an Abaco Reactor
=======================

What is a reactor?
* `abaco docs <https://tacc-cloud.readthedocs.io/projects/abaco/en/latest/>`_
* `abaco swagger guide <https://tacc.github.io/abaco-live-docs/>`_

It's a script, that lives in the cloud, and does something for you. It's not
for compute intensive jobs, that's what apps are for, it's designed to be quick,
responsive, and lightweight.

We're going deploy a reactor that will recieve a notification when a file is uploaded,
create a Tapis job.json, and submit it to our FastQC application.

Copy a Reactor from Github
----------------------------

Clone a Abaco reactor I created to submit FastQC jobs:

::
  git clone https://github.com/JoshuaUrrutia/fastqc_router_reactor.git

Vignette: create a reactor from scratch with ``tapis abaco init`` (not implemented)



Edit reactor.rc and config.yml
---------------------------------
We'll need to make edits to reactor.rc so that it points to your dockerhub username:

The header section
++++++++++++++++++

.. literalinclude:: assets/reactor.rc
   :linenos:
   :emphasize-lines: 5

And change the name of the app in config.yml, so it matches your app id:
.. literalinclude:: assets/config.yml
  :linenos:
  :emphasize-lines: 6

Now we create an empty secrets.json file.
::
  cp secrets.json.sample secrets.json

::
  tapis actors deploy

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

Copy your actor id ``X4blX3Ez65qQZ`` in the above example. You can always list out your actors with ``tapis actors list``

::
  tapis files mkdir agave://urrutia.stampede2.storage/work/05369/urrutia/stampede2 fastqc

Create File System Notifications
---------------------------------
Now you’re ready to create a file system notification.
This notification will pass a message to the fastqc_router_reactor when a file
is uploaded to a particular directory in agave. The fastqc_router_reactor takes
this notification, crafts a job.json, and submits a job to the fastqc_app.
We’ve created a python wrapper to help setup the file system notifications,
you can download the python scripts here:
::
  git clone https://github.com/JoshuaUrrutia/abaco_notifications.git

From the abaco_notifications directory, you can run add_notify_reactor.py to
setup a notification. For example:

::
  python add_notify_reactor.py $AGAVE_SYSTEM_NAME $PATH_TO_DIRECTORY $ACTOR_ID

For my reactor and folder, I would use:
::
  python add_notify_reactor.py data.iplantcollaborative.org urrutia/fastqc GPrgrggl5ler4
  python add_notify_reactor.py urrutia.stampede2.storage /work/05369/urrutia/stampede2/fastqc X4blX3Ez65qQZ

::
  python add_notify_reactor.py urrutia.stampede2.storage /work/05369/urrutia/stampede2/fastqc X4blX3Ez65qQZ
  assocationIds = 8216937226126028310-242ac112-0001-002
  notification id: 1825106086161945066-242ac116-0001-011
  notification url: https://portals-api.tacc.utexas.edu//actors/v2/X4blX3Ez65qQZ/messages?x-nonce=PORTALS_bV5g0q8g5oylx

This will send a notification to the fastqc_router_reactor whenever a file is
uploaded the fastqc directory in my personal storage space on
data.iplantcollaborative.org.

Now the only thing left to do is to test and see if your
notification -> reactor -> app chain is functioning.

::
  tapis files upload agave://$SYSTEM/$PATH/ $FILE
  tapis files upload agave://urrutia.stampede2.storage/work/05369/urrutia/stampede2/fastqc reads1.fastq.gz


::
  tapis actors exces list $ACTOR_ID
  tapis actors execs logs $ACTOR_ID $EXECUTION_ID

  urrutia$ tapis actors execs list X4blX3Ez65qQZ
  +---------------+----------+
  | executionId   | status   |
  +---------------+----------+
  | AqDao7YgEYZ6Z | COMPLETE |
  +---------------+----------+
  urrutia$ tapis actors execs logs X4blX3Ez65qQZ AqDao7YgEYZ6Z
  Logs for execution XqDao7YgEYZ6Z:

::
  tapis jobs list -l 5



::
  python add_notify_requestbin.py $AGAVE_SYSTEM_NAME $PATH_TO_DIRECTORY
  python add_notify_requestbin.py urrutia.stampede2.storage /work/05369/urrutia/stampede2/fastqc2


::
  tapis actors deploy -I X4blX3Ez65qQZ 

Deploying an Actor
=======================

What is an actor? See more info in our documentation:
  * `Actor <https://tapis.readthedocs.io/en/latest/technical/actors.html>`_
  * `Actors LiveDocs <https://tapis-project.github.io/live-docs/?service=Actors>`_

Basically a Tapis actor is a script, that lives in the cloud, and does something for you. It's not
for compute intensive jobs, that's what apps are for, it's designed to be quick,
responsive, and lightweight.

We're going to deploy an actor that will detect when a file is uploaded,
create a Tapis ``job.json``, and submit that job to our FastQC application.

Create an Upload Folder
---------------------------------
Now we'll create the ``uploads`` folder on our storage system. After we create our
notification, any file that is uploaded here will be analyzed automatically by
our FastQC app! Let's ssh into frontera and create the folder w/ bash:
::
  ssh $USERNAME@frontera.tacc.utexas.edu
  cdw
  mkdir uploads
  cd uploads
  pwd 

Make sure to copy this path, ex ``/work2/05369/urrutia/frontera/uploads`` we'll need it when we deploy the actor.


Deploying an Actor
----------------------------

We can move the the `fastqc_actor` directory in that same repository we downloaded earlier:
::
  cd ../fastqc_actor


Edit reactor.py
---------------------------------
Reactor.py is the script that runs when an actor is invoked.
We'll edit the input directory here to match the input directory that you'll be using to upload data:

.. literalinclude:: assets/reactor.py
   :linenos:
   :emphasize-lines: 10

And change the name of the app in ``job.json``, so it matches your app id:

.. literalinclude:: assets/job.json
  :linenos:
  :emphasize-lines: 3

Deploy the Actor
---------------------------------
All that's left is to deploy our reactor:
::
  import json
  actor = {
    "image": "jurrutia/test_actor:0.1",
    "stateless": True,
    "token": True#,
    "cron": True,
    "cron_schedule": "now + 5 minutes"
  }
  t.actors.create_actor(**actor)  # type: ignore
  #t.actors.update_actor(actor_id='B741py883o7Y8', image='$USERNAME/$REPO:$TAG')
  #t.actors.send_message(actor_id='B741py883o7Y8', message={"test":"message"})

You should see a response like:
::
  _links: 
  executions: https://tacc.tapis.io//v3/actors/B741py883o7Y8/executions
  owner: https://tacc.tapis.io//v3/oauth2/profiles/urrutia
  create_time: 2023-07-11T15:58:27.624476
  cron_next_ex: None
  cron_on: False
  cron_schedule: None
  default_environment: 

  description: 
  gid: None
  hints: []
  id: B741py883o7Y8
  image: jurrutia/test_actor:0.1
  last_update_time: 2023-07-11T15:58:27.624476
  link: 
  log_ex: None
  max_cpus: None
  max_workers: None
  mem_limit: None
  mounts: [
  container_path: /home/tapis/runtime_files/_abaco_data1
  host_path: /home/apim/prod/runtime_files/data1
  mode: ro, 
  container_path: /home/tapis/runtime_files/_abaco_data2
  host_path: /home/apim/prod/runtime_files/data2/tacc/urrutia
  mode: rw]
  name: None
  owner: urrutia
  privileged: False
  queue: default
  revision: 1
  run_as_executor: False
  state: 

  stateless: True
  status: SUBMITTED
  status_message: 
  tasdir: None
  token: True
  type: none
  uid: None
  use_container_uid: False
  webhook: 

Copy your actor id (``B741py883o7Y8`` in the above example).
If you forget the id, you can always list out your actors with ``tapis actors list``.

Upload and Test
---------------------------------
Now the only thing left to do is to test and see if our
``upload -> notification -> reactor -> app``
chain is functioning.

Upload a fastq file to your FastQC directory (you can find a copy of this file in
the ``fastqc_app/tests/`` repo):
::
  # tapis files upload agave://urrutia.stampede2.storage/work/05369/urrutia/stampede2/fastqc reads1.fastq.gz
  tapis files upload agave://$SYSTEM/$PATH/ $FILE

Now that we've uploaded lets see if our actor was triggered:
::
  tapis actors execs list $ACTOR_ID

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
  # tapis jobs outputs download 485458bc-335d-4d05-ae30-70de2583b6d5-007
  # cd 485458bc-335d-4d05-ae30-70de2583b6d5-007
  tapis jobs outputs download $JOB_ID
  cd $JOB_ID
  open reads1_fastqc.html

Congratulations, you successfully automated part of your workflow with Tapis!
But there is no reason to stop here, you can add a notification to your FastQC jobs
to trigger a new reactor (and perform an alignment maybe?), and build an entirely
automated workflow by chaining together reactors and apps.

Troubleshooting
=======================

Notifications
------------
If the reactor never executed, you can check the notifications
are working by posting notifications to requestbin using the
``add_notify_requestbin.py`` script in the ``abaco_notifications`` directory:
::
  python add_notify_requestbin.py $AGAVE_SYSTEM_NAME $PATH_TO_DIRECTORY
  assocationIds = 344770698063965720-242ac112-0001-002
  notification id: 213876312678002200-242ac11a-0001-011
  notification url: https://requestbin.agaveapi.co/10ymcld1

You can reupload the file and check the requestbin url to see if it recieves the notification:
::
  #tapis files upload agave://urrutia.stampede2.storage/work/05369/urrutia/stampede2/fastqc reads1.fastq.gz
  tapis files upload agave://$SYSTEM/$PATH/ $FILE

Add ``?inspect`` onto the end of the notification url that was returned by
the add_notify_requestbin.py script, ex: ``https://requestbin.agaveapi.co/10ymcld1?inspect``

Reactor
---------
If the reactor executed, but did not launch your app, you can check the reactor logs:
::
  tapis actors execs logs $ACTOR_ID $EXECUTION_ID

You can then edit your reactor.py or config.yml as needed, and redeploy the actor.
If you want to redeploy your reactor but don't want to re-create
the notification, you can deploy your reactor to the same actor id with:
::
  #tapis actors deploy -I X4blX3Ez65qQZ
  tapis actors deploy -I $ACTOR_ID


Application
-----------
If the app launched, but you are not getting the output you expect,
you can check the app logs. Run jobs-list to find the relevant job_ID, then you can run:
::
  #tapis jobs outputs download 485458bc-335d-4d05-ae30-70de2583b6d5-007
  tapis jobs show $JOB_ID
  # and check the lastStatusMessage
  tapis jobs outputs download $JOB_ID
  # and check the .err and .our files

Introduction
============

This tutorial will teach you how to automate your data analysis using TAPIS.

Module Learning Objectives
--------------------------
In this example, we will show you how to setup automated analysis that is triggered when a file is uploaded to TACC.


Participants are **strongly encouraged** to follow along on the command line.
After completing this module, participants should be able to:

* Register your credentials to a TAPIS system
* Build and deploy an Application
* Trigger an Actor on a regular schedule
* Use an Actor to submit an job to an Application

And we're going to work backwards, first creating the app, then the actor, and finally uploading a file to submit a job.

Why is this important?
----------------------

As you develop your computational skills, you will find that these skill are
in high demand.
Basic operations like moving files, interpreting metadata, initiating scripts,
and formatting outputs will take up and inordinate amount of your time and are boring.
If you can standardize your process for data ingest, you can automate the boring parts of your work.
And can instead devote more time to interpreting your analysis and working on a new, improved version of your pipeline.
Moreover, automating analysis will standardize the processing of your data, so in 6 months from now when
your computational results have been verified experimentally
you can go look back at what version of the application was run, what
the parameters were, and write your methods section accordingly. Instead of
having to guess or remember what you did, you can just check the records.

Don't underestimate the time-saving value of automation! Check out this informative chart from `XKCD <http://xkcd.com>`_

.. image:: https://imgs.xkcd.com/comics/is_it_worth_the_time.png

This module is about 90 minutes, so if this process shaves 30 seconds off of something
you do once a week, or shaves 30 minutes off something you do once a year, then
it's worth the time investment!

Polling Vs Event-Driven Automation
------------
The two main automation paradigms are polling and event-driven automation. 
Polling queries the state at regular intervals, providing consistency and ease of monitoring.
Polling is particularly useful when dealing with systems that do not have built-in event mechanisms. 
However, it may unnecessarily consume resources and is less responsive than an event-driven pipeline. 
On the other hand, event-driven automation allows for instant responses to specific events,  minimizing latency and allowing for real-time processing.  
This makes event-driven systems highly efficient for handling time-sensitive or critical tasks. 
However, event-driven systems typically have a more involved setup process, and logging/debuging might be difficult for end users that don't have administrative access to the event handler.
Additionally, event-driven systems require careful design and management to ensure proper event handling and maintain system integrity.

In summary, polling provides a consistent and reliable approach to data retrieval, while event-driven automation pipelines offer real-time responsiveness and adaptability. 
Choosing between the two depends on the specific requirements of the automation task at hand, considering factors such as data frequency, system characteristics, and the importance of real-time processing.

In this example we use a polling-based automation system, but if you're interested in seeing an event-driven example, see `this guide <https://tacc-reproducible-automation.readthedocs.io/en/latest/>`_` (which uses Tapis V2):


Requirements
------------

* Accounts

  * `GitHub <https://github.com/>`_
  * `Docker Hub Account <https://hub.docker.com/>`_
  * `TACC Account <https://hub.docker.com/>`_

* Software

  * Python 3
  * git
  * python pip
  * `Tapipy <https://github.com/tapis-project/tapipy/tree/main>`_
  * `Docker CE <https://www.docker.com/community-edition>`_


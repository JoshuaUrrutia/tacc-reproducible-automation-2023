Introduction
============

This tutorial will teach you how to automate your data analysis using TAPIS.

Module Learning Objectives
--------------------------
In this example, we will show you how to setup automated analysis that is triggered when a file is uploaded to agave.


Participants are **strongly encouraged** to follow along on the command line.
After completing this module, participants should be able to:

* Create a TAPIS notification when files are uploaded to a specific location
* Trigger an Abaco reactor in response to a notification
* Use an Abaco reactor to submit an job to an Application

And we're going to work backwards, create the App, create the reactor,
then create the notification.

Why is this important?
----------------------

As you develop your computational skills, you will find that these skill are
in high demand.
Basic operations like moving files, interpreting metadata, initiating scripts,
and formatting outputs will take up and inordinate amount of your time and are boring.
If you can standardize your process for data ingest, you can automate the boring parts of your work.
And instead devote more time to interpreting your analysis and working on a new, improved version of your pipeline.
Moreover, automating analysis will standardize the processing of your data, so in 6 months from now when
your computational results have been verified experimentally
you can go look back at what version of the application was run, what
the parameters were, and write your methods section accordingly, without
having to guess or remember what you did, you can just check the records.

Don't underestimate the time-saving value of automation! Check out this informative chart from `XKCD <http://xkcd.com>`_

.. image:: https://imgs.xkcd.com/comics/is_it_worth_the_time.png

This module is about 90 minutes so if this process shaves 30 seconds off of something
you do once a week, or shaves 30 minutes off something you do once a year, then
it's worth the time investment!

Requirements
------------

* Accounts

  * `GitHub <https://github.com/>`_
  * `Docker Hub Account <https://hub.docker.com/>`_

* Software

  * Python 3
  * git
  * python pip
  * `Tapis CLI <https://tapis-cli.readthedocs.io/en/latest/getting-started/installing.html>`_
  * `Docker CE <https://www.docker.com/community-edition>`_
  * Storage and Execution systems setup from the `TAPIS module <https://tacc.github.io/summer-institute-2020-tapis/block1/tapis-systems>`_

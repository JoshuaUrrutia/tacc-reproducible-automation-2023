.. _apps_deploy:

Creating an Application
---------------------

Building A Container
===============================
We're going to take some of what we've learned from best practices and put it into, well, practice.
To deploy an app, we'll build a docker container, push it to dockerhub,
publish an app definition to TAPIS.

Clone this repo locally to get a copy of the app and actor we'll be deploying:
::
  git clone https://github.com/JoshuaUrrutia/automation_resources_2023.git
  cd automation_resources_2023/fastqc_app

Now we can build and push a container that includes fastqc:
::
  docker build -t $USERNAME/fastqc_app:3.0.0 .
  docker push $USERNAME/fastqc_app:3.0.0
  # or on a M1 chip
  docker buildx build --platform linux/amd64 -t $USERNAME/fastqc_app:3.0.0 --push .

Setup Tapipy Client and Push Credentials to Execution System
===============================
If you haven't already, let's go ahead and install the ``tapipy`` on your local machine:
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


Deploy an Application
===============================
First lets update the application ID and the name of the container you just built.
If you don't want to build the container, you can just use the one that I built.

.. literalinclude:: assets/app.json
  :linenos:
  :emphasize-lines: 2,10

Now we have our container and our Tapis app definition, we're ready to register the app with Tapis.
::
  import json
  # read our app definition into python as a dictionary
  with open('app.json', 'r') as openfile:
    app_def = json.load(openfile)
  # Deploy the App
  t.apps.createAppVersion(**app_def)
  # In the future if you need to update and redeploy the app, you can use the .patchApp method
  #t.apps.patchApp(appId=app_def['id'],appVersion=app_def['version'],jobAttributes=app_def['jobAttributes'])

If everything goes smoothly you should get a printout of your app url, ex:
::
  url: http://tacc.tapis.io/v3/apps/urrutia-fastqc/3.0.0

Congratulations, you've deployed a Tapis application!
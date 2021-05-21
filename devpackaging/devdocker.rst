.. _devJenkins:

.. title:: Installing Jenkins and Setting up a Pipeline

++++++++++++++++++++++++++++++++++++++++++++++++++++
Lab 3: Working with Jenkins to build Docker Images
++++++++++++++++++++++++++++++++++++++++++++++++++++

Welcome to the second lab of the DevOps Bootcamp! 

In this lab you will install Jenkins and configure a pipeline. 

.. note::

	Estimated time to complete this lab is 45 minutes


Lab Agenda
+++++++++++

- Install Jenkins
- Configure a Pipeline
  

Prerequisites
++++++++++++++

- You will have completed the Lab 1 of this bootcamp
- We will be using the Ubuntu container created in the first lab for the developer workstation


Working with Jenkins to build Docker Images
++++++++++++++++++++++++++++++++++++++++++++

Now that we are all set with Jenkins, we will learn how to configure Jenkins to build Docker Images based on a Dockerfile. Below are the steps of how you can use Docker within a CI/CD pipeline, using Images as a build artifact that can be promoted to different environments and finally production.

Step #1 : Launch Jenkins
-------------------------
We have Jenkins running on Docker container, if you run the docker ps command it would show you the status of the container. 
Let's launch the Jenkins Dashboard, for this navigate to your webrowser and login with the admin user id that you have created.

localhost:[YOUR PORT]


Step #2 : Configure the plugins and start building Docker Images
------------------------------------------------------------------
Our 1st step is to configure Docker plugin. Whenever a Jenkins build requires Docker, it will create a “Cloud Agent” via the plugin. The agent will be a Docker Container configured to talk to our Docker Daemon.The Jenkins build job will use this container to execute the build and create the image before being stopped. The Docker Image will be stored on the configured Docker Daemon. The Image can then be pushed to a Docker Registry ready for deployment.
  - Once you are inside the Jenkins Dashboard, select Manage Jenkins on the left.
  - On the Configuration page, select Manage Plugins.
   
  -   .. figure:: images/manage_jenkins.png
  -   
  - Manage Plugins page will give you a tabbed interface. Click Available to view all the Jenkins plugins that can be installed.
  - Using the search box, search for Docker plugin. There are multiple Docker plugins, select Docker plugin using the checkbox.
  
  -   .. figure:: images/  install_docker_plugin.png

  - Click Install without Restart at the bottom.
  - The plugin will now be downloaded and installed. Once complete, click the link Go back to the top page.

Once the Docker & GIT plugins have been installed, now we can go ahead and configure how they launch the Docker Containers.

Step #3 : Configure Docker agent
---------------------------------

Navigate back to the Jenkins Dashboard
  -  On the main screen under the section titled Set up a distributed build, there is a link called Configure a cloud. Click on it and select Docker from the list.


  .. figure:: images/configure_cloud.png

You can now configure the container options. Set the name of the agent to docker-agent.
  - The “Docker URL” is where Jenkins launches the agent container. In our case, we’ll use the same daemon as running Jenkins, but in real world scenario it should be separate instance so that it can scale.
  - Use Test Connection to verify Jenkins can talk to the Docker Daemon. You should see the Docker version number returned.

Now the plugin can communicate with Docker,next step would be to configure how to launch the Docker Image for the agent.
  - Using the Images dropdown, select Add Docker Template dropdown.
  - For the Docker Image, use sample one which has Docker client benhall/dind-jenkins-agent. This image is configured with a Docker client and available at https://hub.docker.com/r/benhall/dind-jenkins-agent/
  - To enable builds to specify Docker as a build agent, set a label of docker-agent.
  - Jenkins uses SSH to communicate with agents. Add a new set of “Credentials”. The username is jenkins and the password is jenkins.
  - Finally, expand the Container Settings section by clicking the button. In the “Volumes” text box enter /var/run/docker.sock:/var/run/docker.sock
  - Click Save.

Step #4 : Test the setup
-------------------------

On the Jenkins dashboard, select Create new jobs of type Freestyle project & create new job eg. Jenkins Docker Demo.

- The build will depend on having access to Docker. Using the “Restrict where this project can be run” we can define the label we set of our configured Docker agent. The set “Label Expression” to docker-agent. You should have a configuration of “Label is serviced by no nodes and 1 cloud”.
- Select the Repository type as Git and set the Repository. 
- Use the repository that you created in Lab 1
- We can now add a new Build Step using the dropdown. Select Execute Shell.

Build Step: In the shell screen that you are shown, enter following commands -

.. code-block:: bash
  
  ls 

  docker info 

  docker build -t jenkins-demo:${BUILD_NUMBER} . 

  docker tag jenkins-demo:${BUILD_NUMBER} jenkins-demo:latest 

  docker images

The first command lists all the files in the directory which will be built. When calling docker build we use the Jenkins build number as the image tag. This allows us to version our Docker Images. We also tag the build with latest.

Docker File:

.. code-block:: bash
  
  FROM scratch
  EXPOSE 80
  COPY http-server /
  CMD ["/http-server"]

On the left-hand side, select Build Now. 
You should see a build getting scheduled with a message “(pending — Waiting for next available executor)”.

Jenkins is launching the container and connecting to it via SSH. Sometimes this can take a moment or two.
You can see the progress using command -

.. code-block:: bash
  
  docker logs --tail=10 jenkins

Once the build has completed you should see the Image and Tags using the Docker CLI

.. code-block:: bash
  
  docker images

Working with Jenkins Pipelines
+++++++++++++++++++++++++++++++

We have already created a Git repo in our Lab #1. Since Jenkins needs to push tags to the origin repo, it will need a basic Git configuration. Let’s do that now. 
  - Go to Jenkins > Manage Jenkins > Configure System > Git plugin. 
  - Enter an username of your choice and also your email

Create a new job
-----------------
- We will start by creating a new freestyle project with a name of your choosing (for example, “Ntnx-staging”)
- Under General, check “This project is parameterized”. 
- Add two parameters, as shown below. 
- 
- 
.. figure:: images/
- 
- The “Default Value” of COMMIT_HASH is set to “refs/heads/master” for convenience since we just want to make sure the job has a valid commit to work with. 
- In the future, you may wish to set this to something more relevant like “refs/heads/sprint1”, or clear this field entirely.
- Under Source Code Management, choose ‘Git’. 
  - Add the URL of the repository and the credentials. (Jenkins will attempt to authenticate against this URL as a test, so it should give you an error promptly if the authentication fails.) 
  - Use the commit hash given when the job was started by typing ${COMMIT_HASH} in the Branch Specifier field.
- Under Post-build Actions, add an action with the type “Git Publisher”. 
- Choose “Add Tag” and set the options as shown below. 
- <.. image:: path>
- We check both boxes, because we want Jenkins to do whatever it needs to do in the tagging process (create or update tags as needed). 
- ${TAG} is the second parameter given when the job was started.

When you run the job, you’ll be prompted to enter a commit hash and tag name. Here, you can see that I’ve kicked off two builds: The first build checked out and tagged the latest commit on master.

Make a couple of builds. Check if the first build, the HEAD of the master branch, succeeded. 
- It should be tagged with “0.0.1” and pushed to the origin repo. 
- The second build, should get tagged as well





  




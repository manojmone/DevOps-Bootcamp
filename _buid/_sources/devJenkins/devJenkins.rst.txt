.. _devJenkins:

.. title:: Installing Jenkins and Setting up a Pipeline

++++++++++++++++++++++++++++++++++++++++++++++++++++
Lab 2: Installing Jenkins and Setting up a Pipeline
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

Installing Jenkins
+++++++++++++++++++
We will again use Docker to create a new Jenkins instamce. Before we get started, you’ll need to ensure that you have installed Docker on your machine. 

After installing Docker, download the latest stable Jenkins image by running:

docker image pull jenkins/jenkins:lts 

The main benefit of using Docker containers for hosting Jenkins is that you will be able to persist the state of your Jenkins server using Docker volumes. How does that help?
  - You will retain all your projects and configurations even after restarting your computer (local machine)
  - You don't need to run the whole Jenkins setup again
  - You may remove your container instance and still able to recover the state of your Jenkins server

You can create a volume by running the command below:

docker volume create [YOUR VOLUME]

For example, if you wish to name your docker volume name as jenkinsvol:

docker volume create jenkinsvol

Run the container by attaching the volume and assigning the targeted port. In this example, we'll also run it in detached mode. Here is the command to run your Docker container:

docker container run -d \
    -p [YOUR PORT]:8080 \
    -v [YOUR VOLUME]:/var/jenkins_home \
    --name jenkins-local \
    jenkins/jenkins:lts

    Here is what each argument means:

    -d: detached mode
    -v: attach volume
    -p: assign port target
    —name: name of the container

For example, the command below will create a new container named jenkins-local that uses docker volume named jenkinsvol.

docker container run -d -p 8082:8080 \
    -v jenkinsvol:/var/jenkins_home \
    --name jenkins-local \
    jenkins/jenkins:lts

If you were to run the docker ps command now, you should see two containers one would be the ubuntu container created in lab 1 and second will be the jenkins container.

Now that we know that our container is running, we will use the browser to access our Jenkins instance.

localhost:[YOUR PORT] (localhost:8082 based on my docker container run example above) You can replace 8082 to the port of your choice.

You will be shown a scree like the one below -

<.. image:: path
>

So where's this password? As a part of the Jenkins setup, the password is kept inside the container instance. In order to do this, we need to use the CONTAINER ID (or the name) and run docker exec.

Here is the full command to access it -
    docker container exec \
    [CONTAINER ID or NAME] \
    sh -c "cat /var/jenkins_home/secrets/initialAdminPassword"

So to find the password for my container named jenkins-local, the command will be:
    docker container exec \
    jenkins-local \
    sh -c "cat /var/jenkins_home/secrets/initialAdminPassword"

You will be shown an alpha-numeric code as an output, Copy the code and paste it on the webpage to unlock Jenkins. 
After unlocking, click on Install suggested plugins tile on the Customize Jenkins page. 

Wait until the installation of suggested plugins is complete and then you can proceed in creating your first admin user.
After creating the admin user, setup the Instance configuration. Since you are only using Jenkins locally, leave the URL to your localhost URL. 
Click on Save and Finish to start using Jenkins.


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
  - Manage Plugins page will give you a tabbed interface. Click Available to view all the Jenkins plugins that can be installed.
  - Using the search box, search for Docker plugin. There are multiple Docker plugins, select Docker plugin using the checkbox.
  - While on this page, install the Git plugin for obtaining the source code from a Git repository.
  - Click Install without Restart at the bottom.
  - The plugins will now be downloaded and installed. Once complete, click the link Go back to the top page.

<.. image:: path

Once the Docker & GIT plugins have been installed, now we can go ahead and configure how they launch the Docker Containers.

Step #3 : Configure Docker agent
---------------------------------

Navigate back to the Jenkins Dashboard
  - Select Manage Jenkins.
  - Select Configure System to access the main Jenkins settings.
  - At the bottom, there is a dropdown called Add a new cloud. Select Docker from the list.

  - <.. image:: path

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

ls 

docker info 

docker build -t jenkins-demo:${BUILD_NUMBER} . 

docker tag jenkins-demo:${BUILD_NUMBER} jenkins-demo:latest 

docker images

The first command lists all the files in the directory which will be built. When calling docker build we use the Jenkins build number as the image tag. This allows us to version our Docker Images. We also tag the build with latest.

Docker File:

FROM scratch
EXPOSE 80
COPY http-server /
CMD ["/http-server"]

On the left-hand side, select Build Now. 
You should see a build getting scheduled with a message “(pending — Waiting for next available executor)”.

Jenkins is launching the container and connecting to it via SSH. Sometimes this can take a moment or two.
You can see the progress using command -

docker logs --tail=10 jenkins

Once the build has completed you should see the Image and Tags using the Docker CLI

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
- <.. image:: path>
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





  




Installing Docker on Linux
These steps listed below are for Ubuntu Desktop LTS. You can find the full official docs and steps for other Linux distributions here:

Ubuntu: https://docs.docker.com/install/linux/docker-ce/ubuntu/

CentOS: https://docs.docker.com/install/linux/docker-ce/centos/
(Note: Students have encountered issues when using CentOS or RHEL as the host related to Docker container communication. You may need to research some workaround for the errors you run into or search the QA for already posted solutions)

Debian: https://docs.docker.com/install/linux/docker-ce/debian/

Installation Steps
Create Dockerhub account
https://hub.docker.com/signup

Install Docker
The Docker docs suggest setting up a Docker repository to install and update from.
This is where you should start: https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository

Login to the Dockerhub
In order to push and pull images, you will need to login to the Dockerhub. In your terminal, run docker login and then enter your Dockerhub account's username and password.

Test Docker installation
After completing the installation steps, test out Docker by running sudo docker run hello-world. This should download and run the test container printing "hello world" to your console.

Installing Docker Compose
Navigate to the installation page:
https://docs.docker.com/compose/install/#install-compose-as-standalone-binary-on-linux-systems
Click the Linux Standalone Binary Tab:




Then, CTL+F to search for the Install Compose as standalone binary on Linux systems section on the page. Follow the instructions provided there to install the latest Docker Compose binary.

Testing Docker Compose
After completing, test your installation by running docker-compose -v. This should print the version and build numbers to your console.

Run without Sudo
Follow these instructions to run Docker commands without sudo:
https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user

Start on Boot
Follow these instructions so that Docker and its services start automatically on boot:
https://docs.docker.com/install/linux/linux-postinstall/#configure-docker-to-start-on-boot

You may need to restart your system before starting the course material.

--------------------------------------------------
Buildkit for Docker Desktop
Most students who have the most recent versions of Docker will now have Buildkit enabled by default. If so, you will notice a slightly different output in your terminal when building from a Dockerfile.

The main difference for students will be the final step in the build process. As shown in the lecture the last step would say:

---> fc60771eaa08
Successfully Built fc60771eaa08
 
Now, with Buildkit, the final step would say:

 => => exporting layers                                                      
0.0s => => writing image sha256:ee59c34ada9890ca09145cc88ccb25d32b677fc3b61e921  0.0s
 
Both fc60771eaa08 and ee59c34ada9890ca09145cc88ccb25d32b677fc3b61e921 are the resulting image ID's that you would use to run a container.

eg:

docker run fc60771eaa08
or

docker run ee59c34ada9890ca09145cc88ccb25d32b677fc3b61e921


Disabling Buildkit to match course output

If you wish to disable the Buildkit feature so that you can more accurately follow along with the course, do the following:

Click the Docker Icon in the system tray (Windows) or menu bar (macOS)

Select Preferences

Select Docker Engine

Change buildkit from true to false

{
  "features": {
    "buildkit": false
  },
  "experimental": false
}
Apply and Restart.

Buildkit Features and Documentation

If you want to learn more about features Buildkit has to offer, please check out the following pages:

https://docs.docker.com/develop/develop-images/build_enhancements/

https://docs.docker.com/engine/reference/commandline/build/#specifying-external-cache-sources

https://www.docker.com/blog/advanced-dockerfiles-faster-builds-and-smaller-images-using-buildkit-and-multistage-builds/
--------------------------------------------------
Quick Note for Windows Users
In the upcoming lecture, we will be running a command to create a new image using docker commit with this command:

docker commit -c 'CMD ["redis-server"]' CONTAINERID

If you are a Windows user you may get an error like "/bin/sh: [redis-server]: not found" or "No Such Container"

Instead, try running the command like this:

docker commit -c "CMD 'redis-server'" CONTAINERID
--------------------------------------------------
Required Node Base Image Version
In the upcoming lecture, we will be adding a Node base image to our Dockerfile. To properly follow along with the lectures, please add this specific version:

Change this:

FROM node:alpine

to this:

FROM node:14-alpine

If you do not specify a Node version
If you do not specify a version, you will meet a number of errors caused by changes in the newest versions of Node:

npm ERR! idealTree already exists

This can be resolved by adding a WORKDIR right after the FROM instruction: (we will be adding this in the Specifying a Working Directory lecture anyway):

FROM node:alpine
 
WORKDIR /usr/app

--------------------------------------------------

Important Info About Travis and Account Registration
In the upcoming lecture, we will be signing up for an account with Travis. Since travis-ci.org is no longer operational, you will be redirected to sign up at:

https://www.travis-ci.com/

Along with this change, they have also updated their terms regarding free accounts and credits due to crypto mining abuse:

https://blog.travis-ci.com/2021-10-20-mining

Because of this, you will need to select a plan after registering.

Select Monthly Plans and then the free Trial Plan. This will give you 10,000 free credits to use within 30 days.



After this trial period, the plan will expire. It is possible that you will not finish the course by the time the credits expire.

If you run out of credits, are unable to register for a Travis account, or, if you simply do not wish to use Travis - We have provided guidance to use Github Actions instead (scroll past the lecture video to see the featured question):

https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/learn/lecture/11437142#questions/17673926


-------------------------------------------
Required Travis Script Updates
In the upcoming lecture, we will be adding a script to our .travis.yml file. Due to a change in how the Jest library works with Create React App, we need to make a small modification:

script:
  - docker run USERNAME/docker-react npm run test -- --coverage
 
instead should be:

script:
  - docker run -e CI=true USERNAME/docker-react npm run test
Additionally, you may want to set the following property to the top of your .travis.yml file:

language: generic 
You can read up on the CI=true variable here:

https://facebook.github.io/create-react-app/docs/running-tests#linux-macos-bash

and environment variables in Docker here:

https://docs.docker.com/engine/reference/run/#env-environment-variables

-----------------------------------------------

Required Updates for Amazon Linux 2 Platform - DO NOT SKIP
updated 8-12-2021

When creating our Elastic Beanstalk environment in the next lecture, we need to select Docker running on 64bit Amazon Linux 2 and make a few changes to our project:

This new AWS platform will conflict with the project we have built since it will look for a docker.compose.yml file to build from by default instead of a Dockerfile.

To resolve this, please do the following:

1. Rename the development Compose config file

Rename the docker-compose.yml file to docker-compose-dev.yml. Going forward you will need to pass a flag to specify which compose file you want to build and run from:
docker-compose -f docker-compose-dev.yml up
docker-compose -f docker-compose-dev.yml up --build
docker-compose -f docker-compose-dev.yml down

2. Create a production Compose config file

Create a docker-compose.yml file in the root of the project and paste the following:

version: '3'
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - '80:80'
AWS EBS will see a file named docker-compose.yml and use it to build the single container application.
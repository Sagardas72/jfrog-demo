## Getting started
This document will help you setup your own demo environment on your local workstation

### Set up Artifactory & Jenkins Servers with Docker

#### Prerequisites
1. Have docker installed locally on your workstation. Follow [this link](https://docs.docker.com/get-docker/) to help install docker locally if not already done. 
2. Have a trial license for JFrog available. It can be gotten from https://jfrog.com/start-free/

#### Setup Jenkins in Docker
1. Create a bridge network in Docker
   ```docker network create jfrog```
2. Run the **docker:dind** docker image. This image will be used to execute docker commands inside of Jenkins.
   ```
   docker run \
   --name jenkins-docker \
   --rm \
   --detach \
   --privileged \
   --network jenkins \
   --network-alias docker \
   --env DOCKER_TLS_CERTDIR=/certs \
   --volume jenkins-docker-certs:/certs/client \
   --volume jenkins-data:/var/jenkins_home \
   --publish 2376:2376 \
   --storage-driver overlay2 \
   docker:dind
   ```
3. Customize the official Docker image
   - Create a Dockerfile with the following content
     ```
	 FROM jenkins/jenkins:2.332.3-jdk11
	 USER root
	 RUN apt-get update && apt-get install -y lsb-release
	 RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
	  https://download.docker.com/linux/debian/gpg
	 RUN echo "deb [arch=$(dpkg --print-architecture) \
	  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
	  https://download.docker.com/linux/debian \
	  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
	 RUN apt-get update && apt-get install -y docker-ce-cli
	 USER jenkins
	 RUN jenkins-plugin-cli --plugins "blueocean:1.25.3 docker-workflow:1.28"
	 ```
4. Run the newly created docker image
   ```
   docker run \
   --name jenkins-blueocean \
   --restart=on-failure \
   --detach \
   --network jfrog \
   --env DOCKER_HOST=tcp://docker:2376 \
   --env DOCKER_CERT_PATH=/certs/client \
   --env DOCKER_TLS_VERYFY=1 \
   --publish 8080:8080 \
   --publish 50000:50000 \
   --volume jenkins-data:/var/jenkins_home \
   --volume jenkins-docker-certs:/certs/client:ro \
   jenkins-blueocean:2.332.3-1
   ```
5. Access the jenkins container to get the inital administrator password
   ```docker container exec -it jenkins-blueocean bash```
6. Get the inital administrator password
   ```cat /var/jenkins_home/secrets/initialAdminPassword```
7. Browse to http://localhost:8080 on the workstation browser to access the Jenkins UI, and follow the steps from unlocking jenkins to setting up the default plugins. The initial administrator password will be used in this step to unlock jenkins. 
8. Exit out of the jenkins-blueocean container.
   ```exit```
	
#### Setup Artifactory Server in DOcker
1. Create a docker volume
   ```docker volume create artifactory-data```
2. Run the artifactory container
   ```
   $ docker run \
   --detach \
   --name artifactory \
   --publish 8082:8082 \
   --publish 8081:8081 \
   --volume artifactory-data:/var/opt/jfrog/artifactory \
   --network jfrog \
   docker.bintray.io/jfrog/artifactory-pro:latest
   ```
3. Browse to http://localhost:8081/artifactory on the workstation browser to access the Artifactory UI. 
   > **NOTE**: The initial setup will require the entry of the jfrog license.
   
   > **NOTE**: Make sure to attach the artifactory container to the same network that Jenkins server is running on to facilitate communication between Jenkins server and Artifactory.
4. Set a new password for the admin user. Default is admin/password.
5. When the UI asks to **Create Repositories**, select the **Quick Setup** option and then select **Maven** as the repository type.
6. Add a meaningful name as the repository prefix. This will add the same prefix to all the five repositories that will be created for the maven repository. These names (with the prefix) will be used to upload the artifacts from Jenkins to Artifactory. 

#### Configure Jenkins Server
1. Login to Jenkins
2. On the left tab, click on **Manage Jenkins -> Manage Plugins**
3. Click on the Available tab and search for **Artifactory Plugin**
4. Select the plugin and from the bottom selections, click on **Download now and install after restart**. This will lead the UI to the plugin installation page where it will start displaying the installation progress of the plugin and all of its dependencies. At the bottom of the screen is an option to restart Jenkins once the installation is finished. Select that option. This will make sure that Jenkins restarts as soon as the installation finishes. 
   
README
======

### BUILD Section

#### Steps to setup Jenkins build server(Master) on an Ubuntu server.
---------------------------------------------------------------------
> Install Jenkins
  
  wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -  
  echo "deb http://pkg.jenkins-ci.org/debian binary/" | sudo tee -a /etc/apt/sources.list.d/jenkins.list  
  sudo apt-get update  
  sudo apt-get install jenkins

> Install Apache
  
  sudo apt-get install apache2  
  sudo a2enmod proxy  
  sudo a2enmod proxy_http

> Open /etc/apache2/sites-available/jenkins.conf and add the following to enable proxy to port
  
  <VirtualHost *:80>
      ServerName HOSTNAME
      ProxyRequests Off
      <Proxy *>
          Order deny,allow
          Allow from all
      </Proxy>
      ProxyPreserveHost on
      ProxyPass / http://localhost:8080/
  </VirtualHost>

> Enable it
  
  sudo a2ensite jenkins  
  sudo service apache2 reload

> Installing dependencies for our build server : Java / Maven / Git
  
  sudo add-apt-repository ppa:webupd8team/java  
  sudo apt-get update  
  sudo apt-get install oracle-java7-installer maven git-core

####Tasks explained
-------------------
1. ##### The ability to trigger a build in response to a git commit via a git hook.

> To configure Github to trigger a build in response to a push made by the user,
  go to the settings page of the project. Select the "jenkins (git plugin)" from the
  "Webhooks and services" tab. Configure the plugin to set the jenkins URL to the
  Jenkins master. In our case "http://ec2-54-148-38-238.us-west-2.compute.amazonaws.com/"
  and select the checkbox "Active". This configuration can be seen below-

![GitHook_configuration](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/task1_githook.JPG)
  
> To demonstrate this task, the screenshot below is of the **console output**
  that reads the following line "Started by an SCM change" (which means it was triggered by a 
  git push)

![SCM change](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/buildbyscm_task1.JPG)

> The "Jenkins (git plugin)" in github further affirms that the last delivery was successful.

![Success message](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/lastsuccess_task1.png)


2. ##### The ability to setup dependencies for the project and restore to a clean state.
>  The screenshots below demonstrate the task. When the project is first loaded, the dependencies are
   installed, target files are created and the project is built.

 ![Build is successful](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/nuildsuccess.JPG)

> For restoring to a clean state:  
  mvn clean

3. ##### The ability to execute a build script (e.g., shell, maven)
> The screenshots below demonstrate the task.

![BUILD SUCCESS](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/buildsuccess.png)

4. ##### The ability to run a build on multiple nodes (e.g. jenkins slaves, go agents, or a spawned droplet/AWS.).
>  To create and configure slave nodes (in the Jenkins master) that would be ready to accept the tasks delegated
   by the master. The following steps are for setup:
 
- Create an amazon instance (Ubuntu- same as that of the master) for the slave (Jenkins Slave)
- Install ssh server (To connect to the slaves via SSH)
- Create a slave user and generate rsa public and private keys for it.
- Copy the private key to Jenkins (under manage credentials using "Ssh slaves plugin") for that particular slave.
- Installed all the required tools for building the target project(Maven, Jenkins and Git).

The commands in use are as follows:

1. Installing required dependencies:
   sudo add-apt-repository ppa:webupd8team/java
   sudo apt-get update
   sudo apt-get install oracle-java7-installer maven git-core
   
2. Creating slave user
   sudo adduser theslave

3. sudo apt-get install openssh-server
   sudo vim /etc/ssh/sshd_config

4. Appending to the end of file:
   AllowUsers theslave

5. Restarting ssh server:
   restart ssh server
   sudo restart ssh

6. Switch user:
   sudo su theslave

7. Creating ssh directory and giving permissions:
   mkdir ~/.ssh
   chmod 700 ~/.ssh

8. Run the ssh-keygen command:
   ssh-keygen -t rsa

9. Copy the contents of id_rsa public key. 
   cd .ssh
   cat id_rsa.pub >> authorized_keys

10.Copy the id_rsa private key into jenkins manage credentials for "theslave" 
   
> In our project, we have created two slaves 'theslave' and 'jenkins' to demonstrate 
  the ability to run build on multiple nodes. For the purpose of this task, we have 
  changed the number of executors to 0. So that the incoming build request will be directly
  delegated by the master to the online slaves. This is as shown below:

  ![Number of executors in Master](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/master_0.JPG) 
  
> Before jenkins master delegates job to slaves, the build queue have two items as shown below (because the items 'testslavemaven' and 'workspacewipeout' have the same github project:
  ![Trigger tasks](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/trigger_task4.png)
  
> After the jobs are delegated to the two slaves "theslave" and "jenkins", the build executor status depicts that both the slaves are executing build.
  ![Multiple slaves](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/multipleslaves_task4.png)
  
> The slave "jenkins" executed one job as shown below:
![Jenkins slave working](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/task1_consolescm.JPG)

> The slave "theslave" executed another job as shown below:
![theslave working](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/theslave_console.JPG)

> SSH authentication of 'theslave'
![slaves authenticated](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/theslave_auth.JPG)

> The slaves are idle as they don't have any task right now. 
![Idle slaves](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/slavesidle.JPG)

> The slaves are online.
![Online slaves](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/after_theslavelaunched.JPG)

5. ##### The ability to retrieve the status of the build via http.
> The status of the build can be retrieved by using the following 
  command: curl -i -H "Accept: application/json" -H "Content-Type: application/json" http://ec2-54-148-38-238.us-west-2.compute.amazonaws.com/job/testslavemaven/lastBuild/api/json
  
![Status response](https://github.com/mahasanath/Firsttask/blob/master/milestone1_devops_screenshots/task5_consolehttp.JPG)

> The configuration file (config.xml) for the Jenkins master and the job configuration file
  have been uploaded on the github.

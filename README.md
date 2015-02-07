# Firsttask

README
======

### BUILD Section

##### The ability to trigger a build in response to a git commit via a git hook.

> To configure Github to trigger a build in response to a push made by the user,
  go to the settings page of the project. Select the "jenkins (git plugin)" from the
  "Webhooks and services" tab. Configure the plugin to set the jenkins URL to the
  Jenkins master. In our case "http://ec2-54-148-38-238.us-west-2.compute.amazonaws.com/"
  and select the checkbox "Active". This configuration can be seen below-

  ![Alt text][t1id1]
  [t1id1]: ./.png 
  
> To demonstrate this task, the screenshot below is of the **console output**
  that reads the following line "Started by an SCM change" (which means it was triggered by a 
  git push)

  ![Alt text][t1id2]
  [t1id2]: ./.png 

> The "Jenkins (git plugin)" in github further affirms that the last delivery was successful.

  ![Alt text][t1id3]
  [t1id3]: ./.png 


##### The ability to setup dependencies for the project and restore to a clean state.
>  The screenshots below demonstrate the task. When the project is first loaded, the dependencies are
   installed, target files are created and the project is built.

  ![Alt text][t2id1]
  [t2id1]: ./.png 

> For restoring to a clean state:  
  mvn clean

  ![Alt text][t2id2]
  [t2id2]: ./.png 

##### The ability to execute a build script (e.g., shell, maven)
> The screenshots below demonstrate the task.

![Alt text][t3id3]
  [t3id3]: ./.png  


##### The ability to run a build on multiple nodes (e.g. jenkins slaves, go agents, or a spawned droplet/AWS.).
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
  delegated by the master to the online slaves. 

![Alt text][t4id1]
  [t4id1]: ./.png 
  
![Alt text][t4id2]
  [t4id2]: ./.png 
  
![Alt text][t4id3]
  [t4id3]: ./.png 
  
![Alt text][t4id4]
  [t4id4]: ./.png 
  
##### The ability to retrieve the status of the build via http.
> The status of the build can be retrieved by using the following 
  command: curl -i -H "Accept: application/json" -H "Content-Type: application/json" http://ec2-54-148-38-238.us-west-2.compute.amazonaws.com/job/testslavemaven/lastBuild/api/json
  
  ![Alt text][t5id1]
  [t5id1]: ./.png 
  
  ![Alt text][t5id2]
  [t5id2]: ./.png 


> The configuration file (config.xml) for the Jenkins master and the job configuration file
  have been uploaded on the github.

#Spark/hdfs Cluster Setup
##VM setup
Currently, 3 VM's setup with 20gb volume, 2 VCPUS with 2 GB ram.

###LAN IP's:
* Node 0: 192.168.2.216
* Node 1: 192.168.2.58
* Node 2: 192.168.2.10

###Floating IP's:
* Node 0: 130.238.29.209
* Node 1: 130.238.29.139
* Node 2: 130.238.29.7

##Hadoop Setup
**Step 1:** As always when launching a new linux instance, we need to run the commands below to update all installed packages on the machine. Do this on all instances.10
- ```sudo apt-get update```
- ```sudo apt-get upgrade ```

**Step 2:** From the master node (node 0), run the below commands. The following commands setup so that the master node have passwordless login (ssh) to the worker nodes.
- ```ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa``` - Generates new key.
- ```cat .ssh/id_rsa.pub >> ~/.ssh/authorized_keys``` - Append the new key to the authorized keys.
- ```scp -i team_19.pem .ssh/authorized_keys ubuntu@192.168.2.58:/home/ubuntu/.ssh/authorized_keys``` - Copy the new key to node 1
- ```scp -i team_19.pem .ssh/authorized_keys ubuntu@192.168.2.10:/home/ubuntu/.ssh/authorized_keys``` - Copy the new key to node 2

**Step 3:** Add all nodes to /etc/hosts.
- ```sudo nano /etc/hosts``` Edit and add the text below:
 - ```192.168.2.216 namenode.socal.rr.com```
 - ```192.168.2.58 datanode1.socal.rr.com```
 - ```192.168.2.10 datanode2.socal.rr.com```

**Step 4:** Install JDK1.8 on all nodes.
- ```sudo apt-get -y install openjdk-8-jdk-headless``` - Installs JDK1.8
- ```java -version``` - Checks if the install was successful. 
##Spark Setup
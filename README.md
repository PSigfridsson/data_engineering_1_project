# Data Engineering 1 - Project
Repository for the Data Engineering 1 project.

## Spark/hdfs Cluster Setup
### VM setup
Currently, 3 VM's setup with 20gb volume, 2 VCPUS with 2 GB ram.

#### LAN IP's:
* Node 0: 192.168.2.216
* Node 1: 192.168.2.58
* Node 2: 192.168.2.10

#### Floating IP's:
* Node 0: 130.238.29.209
* Node 1: 130.238.29.139
* Node 2: 130.238.29.7

### Hadoop Setup
**Step 1:** As always when launching a new linux instance, we need to run the commands below to update all installed packages on the machine. Do this on all instances.
- ```sudo apt-get update```
- ```sudo apt-get upgrade ```

**Step 2:** From the master node (node 0), run the below commands. The following commands setup so that the master node have passwordless login (ssh) to the worker nodes.
- ```ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa``` - Generates new key.
- ```cat .ssh/id_rsa.pub >> ~/.ssh/authorized_keys``` - Append the new key to the authorized keys.
- ```scp -i team_19.pem .ssh/authorized_keys ubuntu@192.168.2.58:/home/ubuntu/.ssh/authorized_keys``` - Copy the new key to node1
- ```scp -i team_19.pem .ssh/authorized_keys ubuntu@192.168.2.10:/home/ubuntu/.ssh/authorized_keys``` - Copy the new key to node2

**Step 3:** Add all nodes to /etc/hosts on the master node.
- ```sudo nano /etc/hosts``` Edit and add the text below:
  - ```192.168.2.216 master```
  - ```192.168.2.58 worker-node1```
  - ```192.168.2.10 worker-node2```

**Step 4:** Install JDK1.8 on all nodes.
- ```sudo apt-get -y install openjdk-8-jdk-headless``` - Installs JDK1.8
- ```java -version``` - Checks if the install was successful. 

**Step 5:** Install Hadoop on all nodes.
- ```sudo wget -P ~ https://mirrors.sonic.net/apache/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz```
- ```tar -xzf hadoop-3.2.1.tar.gz```
- ```mv hadoop-3.2.1 hadoop```
- ```sudo rm hadoop-3.2.1.tar.gz```

**Step 6:** Hadoop configuration on all nodes.
- ```nano ~/.bashrc``` - Edits file, add the text below.
```
export HADOOP_HOME=/home/ubuntu/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export HADOOP_HDFS_HOME=${HADOOP_HOME}
export YARN_HOME=${HADOOP_HOME}
```
- ```source ~/.bashrc``` - Reloads the env variables.

**Step 7:** Hadoop cluster configuration, do this on the master node then copy to the worker nodes.
- ```nano ~/hadoop/etc/hadoop/hadoop-env.sh``` - Add/Edit the below text.

```export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64```

- ```nano ~/hadoop/etc/hadoop/core-site.xml``` - Edit file, add text below.
```
<configuration>
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://192.168.2.216:9000</value>
	</property>
</configuration>
```
- ```nano ~/hadoop/etc/hadoop/hdfs-site.xml``` - Edit file, add text below.
```
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///usr/local/hadoop/hdfs/data</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///usr/local/hadoop/hdfs/data</value>
    </property>
</configuration>
```
- ```nano ~/hadoop/etc/hadoop/yarn-site.xml``` - Edit file, add text below.
```
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
       <name>yarn.resourcemanager.hostname</name>
       <value>192.168.2.216</value>
    </property>
</configuration>
```
- ```scp ~/hadoop/etc/hadoop/hdfs-site.xml ubuntu@LAN_IP_FOR_NODE:~/hadoop/etc/hadoop/hdfs-site.xml ``` - Copy to the files to the nodes.
- ```scp ~/hadoop/etc/hadoop/core-site.xml ubuntu@LAN_IP_FOR_NODE:~/hadoop/etc/hadoop/core-site.xml ``` - Copy to the files to the nodes.
- ```scp ~/hadoop/etc/hadoop/yarn-site.xml ubuntu@LAN_IP_FOR_NODE:~/hadoop/etc/hadoop/yarn-site.xml ``` - Copy to the files to the nodes.
- ```nano ~/hadoop/etc/hadoop/mapred-site.xml``` - ONLY ON MASTER, edit file, add text below.
```
<configuration>
	<property>
		<name>mapreduce.jobtracker.address</name>
		<value>192.168.2.216:54311</value>
	</property>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
</configuration>
```

***Step 8:*** Create data folder on ALL nodes.
- ```sudo mkdir -p /usr/local/hadoop/hdfs/data```
- ```sudo chown ubuntu:ubuntu -R /usr/local/hadoop/hdfs/data```
- ```chmod 700 /usr/local/hadoop/hdfs/data```

***Step 9:*** Create master and worker files on ALL nodes, i created them on the master node then using SCP to transfer to the worker nodes.
- ```nano ~/hadoop/etc/hadoop/masters``` - Edit and add the LAN IP of the master node.
- ```nano ~/hadoop/etc/hadoop/workers``` - Edit and add the LAN IP's of all worker nodes.

***Step 10:*** Format HDFS and Start Hadoop Cluster, run these commands on the master node.
- ```hdfs namenode -format``` - Formatting
- ```start-dfs.sh``` - Starting hdfs

Now run the following command on all nodes to verifiy that the hdfs start was successful.
- ```jps``` - The master node should be running NameNode and SecondaryNameNode. The workers should run DataNode.

Now the hdfs should be properly setup. To connect to the web gui edit your ~/.ssh/config file. Add the following:

	Host 130.238.29.209
  		User ubuntu
  		# modify this to match the name of your key
  		IdentityFile ~/.ssh/team_19.pem
  		# HDFS namenode web gui
  		LocalForward 9870 192.168.2.216:50070

Now you should be able to connect to the hdfs web gui by typing http://localhost:9870 in your browser, on your local machine. (This assumes the proper security group is added to the master.)
### Spark Setup

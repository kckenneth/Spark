|Title |  Spark Installation |
|-----------|----------------------------------|
|Author | Kenneth Chen |
|Utility | IBM, Softlayer, Spark |
|Date | 10/08/2018 |

# Apache Spark Setup

# Setup 3 virtual servers in the cloud. 
Provision CENTOS 7 (os), 2 (CPU), 4GB (RAM), 100GB (local disk) 
```
$ slcli vs create --datacenter=sjc01 --hostname=spark1 --domain=mids.com --billing=hourly --cpu=2 --memory=4096 --disk=100 --os=CENTOS_7_64
$ slcli vs create --datacenter=sjc01 --hostname=spark2 --domain=mids.com --billing=hourly --cpu=2 --memory=4096 --disk=100 --os=CENTOS_7_64
$ slcli vs create --datacenter=sjc01 --hostname=spark3 --domain=mids.com --billing=hourly --cpu=2 --memory=4096 --disk=100 --os=CENTOS_7_64
```
Check the servers provisioned
```
$ slcli vs list

:..........:..........:...............:..............:............:........:
:    id    : hostname :   primary_ip  :  backend_ip  : datacenter : action :
:..........:..........:...............:..............:............:........:
: 63442913 :  spark1  : 50.97.252.101 : 10.53.47.13  :   sjc01    :   -    :
: 63442935 :  spark2  : 50.97.252.103 : 10.53.47.34  :   sjc01    :   -    :
: 63442941 :  spark3  : 50.97.252.102 : 10.53.47.30  :   sjc01    :   -    :
:..........:..........:...............:..............:............:........:
```
# Setup DNS (on all nodes)
To easily ssh with the name instead of the IP addresses, we will setup the DNS. 

```
# vi /etc/hosts

127.0.0.1     localhost.localdomain localhost
50.97.252.101  spark1
50.97.252.103  spark2
50.97.252.102  spark3
```

# Setup passwordless ssh
The idea is to `ssh` without password between nodes. `spark1` must be able to `ssh spark1`, `ssh spark2` and `ssh spark3`. You already know by now that to ssh using the name requires you to set up at `/etc/hosts`. Without password requires you to setup `ssh-keygen` generation. 

```
# ssh-keygen -f ~/.ssh/id_rsa -b 2048 -t rsa 
# cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys 
# chmod 600 ~/.ssh/authorized_keys

# scp ~/.ssh/* root@50.97.252.103:/root/.ssh/
# scp ~/.ssh/* root@50.97.252.102:/root/.ssh/

# for i in spark1 spark2 spark3; do ssh-copy-id $i; done
```
`ssh-copy-id` will copy `id_rsa.pub` key to `authorized_keys` file (will be created) in other nodes. So when it tries to establish the connection, it will first ask the password of the node it's sshing into. 

To test the ssh passwordless for all nodes 

```
# vi test.sh
```
Copy the following script 
```
#!/bin/bash

# Edit node list
nodes="spark1 spark2 spark3"

# Test ssh configuration
for i in $nodes
 do echo -n "Testing ssh ${i}: "
 ssh ${i} "ssh ${i} date"
done
```
Run the script 
```
# chmod 755 test.sh
# ./test.sh

Testing ssh spark1: Sun Oct 14 17:27:07 CDT 2018
Testing ssh spark2: Sun Oct 14 17:27:08 CDT 2018
Testing ssh spark3: Sun Oct 14 17:27:09 CDT 2018
```

# Install Java, SBT and Spark (on all nodes)

Install SBT
```
# curl https://bintray.com/sbt/rpm/rpm | sudo tee /etc/yum.repos.d/bintray-sbt-rpm.repo
# yum install -y java-1.8.0-openjdk-devel sbt git
```
Set Jave path in `~/.bash_profile`. After you set the export path, you need to `source` it so that the path becomes activated. To test later, check the java version.
```
# echo export JAVA_HOME=\"$(readlink -f $(which java) | grep -oP '.*(?=/bin)')\" >> /root/.bash_profile
# source /root/.bash_profile

# $JAVA_HOME/bin/java -version

openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)
```
Install Spark
```
# curl https://d3kbcqa49mib13.cloudfront.net/spark-2.1.1-bin-hadoop2.7.tgz | tar -zx -C /usr/local --show-transformed --transform='s,/*[^/]*,spark,'
```
Set Spark path
```
# echo export SPARK_HOME=\"/usr/local/spark\" >> /root/.bash_profile
# source /root/.bash_profile
```
# Configure Spark (on spark1)

```
# cd $SPARK_HOME/conf/
# vi slaves

spark1
spark2
spark3
```

# Copy files to spark1
We will copy moby10b.txt and src directory into our spark1 node and check their directory in our node. 
```
# cd /root
# git clone https://github.com/MIDS-scaling-up/coursework.git
```
moby10b.txt and src directory
```
/root/coursework/week6/hw/apache_spark_introduction/moby10b.txt
/root/coursework/week6/hw/apache_spark_introduction/src/spark
```

# Start Spark
we will start spark from spark1 as master node. There are a few script you should be familiar with.  
```
sbin/start-master.sh - Starts a master instance on the machine the script is executed on
sbin/start-slaves.sh - Starts a slave instance on each machine specified in the conf/slaves file
sbin/start-all.sh - Starts both a master and a number of slaves as described above
sbin/stop-master.sh - Stops the master that was started via the bin/start-master.sh script
sbin/stop-slaves.sh - Stops all slave instances on the machines specified in the conf/slaves file
sbin/stop-all.sh - Stops both the master and the slaves as described above
```

Start the master and check if it's working at `http://<master_ip>:8080/` = `http://50.23.91.125:8080`
```
# $SPARK_HOME/sbin/start-master.sh

starting org.apache.spark.deploy.master.Master, logging to /usr/local/spark/logs/spark-root-org.apache.spark.deploy.master.Master-1-spark1.mids.com.out
```
Start the slaves, refresh the browser and check if all workers are now active. 
```
# $SPARK_HOME/sbin/start-slaves.sh
```

# Calculate Pi
```
# $SPARK_HOME/bin/run-example SparkPi
```

Question 1: What value of PI to you get? Why is the value not "exact"? For a hint, see `$SPARK_HOME/examples/src/main/python/pi.py`







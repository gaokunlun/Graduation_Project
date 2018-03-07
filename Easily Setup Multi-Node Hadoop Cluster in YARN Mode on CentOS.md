## 1. Install CentOS:
I will be using 3 CentOS 6.5 minimal machines on my virtual box and lets name them as nn1(consisting of namenode and resourcemanager), dn1(datanode1 and nodemanager1), dn2(datanode2 and nodemanager2). You can also use ubuntu here instead of CentOS

You can make your network adapter as Bridge or Host-Only depending upon your requirement. Here are my 3 machines
```shell
nn1–192.168.1.200 HDFS — Namenode and YARN — Resourcemanager
dn1–192.168.1.201 HDFS — Datanode and YARN — Nodemanager
dn2–192.168.1.202 HDFS — Datanode and YARN — Nodemanager
```

## 2. Edit /etc/hosts :
Now lets ssh into nn1, dn1, dn2 and change the /etc/hosts file, so that we can use hostname instead of IP everytime we wish to use or ping any of these machines
https://cdn-images-1.medium.com/max/1600/1*oef3tNXeAhHqYqxsjPQHrw.png

## 3. Download and Install Java:
```
[root@nn1 /]yum install wget
[root@nn1 /]wget http://download.oracle.com/otn-pub/java/jdk/8u161-b12/2f38c3b165be4555a1fa6e98c45e0808/jdk-8u161-linux-x64.rpm
[root@nn1 /]# rpm -ivh jdk-7u45-linux-x64.rpm
```

## 4. Install Hadoop 2.7.5
```
[root@nn1 ~]# wget https://archive.apache.org/dist/hadoop/core/hadoop-2.7.5/hadoop-2.7.5.tar.gz
[root@nn1 ~]# tar -xzvf /root/hadoop-2.7.5.tar.gz -C /home/hadoop/
```

## 5. Set Environment variables in ~/.bash_profile(centos) or .bashrc(ubuntu)
```shell
export JAVA_HOME=/usr/java/jdk1.7.0_45
PATH=$PATH:$JAVA_HOME/bin:/home/hadoop/hadoop-2.7.3/bin:$PATH
export PATH
# Set Hadoop-related environment variables
export HADOOP_HOME=/home/hadoop/hadoop-2.7.3
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
# Add Hadoop bin/ directory to PATH
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
```
source it to reflect changes

## 6. Modify core-site.xml
```
cd /home/hadoop/hadoop-2.7.5/etc/hadoop
vim core-site.xml
```

```shell
<configuration>
<property>
<name>fs.default.name</name>
<value>hdfs://192.168.1.200:9000</value>
</property>
</configuration>
```

## 7. Modify hdfs-site.xml
```shell
<configuration>
<property>
<name>dfs.namenode.name.dir</name>
<value>file:/home/hadoop/hadoop-2.7.5/hadoop_store/hdfs/namenode2</value>
</property>
<property>
<name>dfs.datanode.data.dir</name>
<value>file:/home/hadoop/hadoop-2.7.5/hadoop_store/hdfs/datanode2
</value>
</property>
</configuration>
```

## 8. Modify mapred-site.xml
cp mapred-site-template.xml mapred-site.xml
```shell
<configuration>
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
</property>
</configuration>
```

## 9. Modify yarn-site.xml
```shell
<configuration>
<! — Site specific YARN configuration properties — >
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
<property>
<name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
<value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>
<property>
<name>yarn.resourcemanager.resource-tracker.address</name>
<value>192.168.1.200:8025</value>
</property>
<property>
<name>yarn.resourcemanager.scheduler.address</name>
<value>192.168.1.200:8030</value>
</property>
<property>
<name>yarn.resourcemanager.address</name>
<value>192.168.1.200:8050</value>
</property>
</configuration>
```

## 10. Create Namenode directory in nn1
mkdir -p /home/hadoop/hadoop-2.7.3/hadoop_store/hdfs/namenode2

## 11. Modify slaves files and add both datanodes ip’s

## 12. Copy all modified files to both the datanodes dn1 and dn2

## 13. Make datanode directory on both dn1 and dn2 and give permissions
```
[root@dn1 hadoop] mkdir -p /home/hadoop/hadoop-2.7.5/hadoop_store/hdfs/datanode2/
[root@dn1 hadoop] chmod 755 /home/hadoop/hadoop-2.7.5/hadoop_store/hdfs/datanode2/
```

## 14. Disable Firewall
service iptables stop
service ip6tables stop

## 15. Format Namenode on nn1
[root@nn1 hadoop] hdfs namenode -format

## 16. Start Namenode, Datanode(start-dfs.sh) and Resourcemanager, Nodemanager(start-yarn.sh)
[root@nn1 hadoop-2.7.5] ./sbin/start-dfs.sh
[root@nn1 hadoop-2.7.5] ./sbin/start-yarn.sh

## 17. jps on nn1
[root@nn1 hadoop-2.7.5] jps
21776 SecondaryNameNode
22240 Jps
21574 NameNode
21974 ResourceManager

## 18. jps on dn1 and dn2
[root@dn1 hadoop-2.7.5] jps
12752 DataNode
16644 Jps
16586 NodeManager


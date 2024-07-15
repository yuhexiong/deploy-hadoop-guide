# Deploy Hadoop Guide


## Overview

- Virtual Machine: Ubuntu v22.04.4
- System: Hadoop v3.3.6

### Architecture

**10.0.0.1 hadoop01.domain.name**: NameNode DataNode  
**10.0.0.2 hadoop02.domain.name**: SecondaryNameNode DataNode  
**10.0.0.3 hadoop03.domain.name**: DataNode  
**10.0.0.4** dns  
mount at **/mnt/hadoop**  

## IP And Host

### Setting IP

```
sudo vim /etc/netplan/00-installer-config.yaml
```
refer to [00-installer-config.yaml](./00-installer-config.yaml)

### Setting Hostname And Hosts

```
sudo vim /etc/hostname
```
modify to hadoop01.domain.name

```
sudo vim /etc/hostname
```
modify as below
```
10.0.0.1 hadoop01.domain.name
10.0.0.2 hadoop02.domain.name
10.0.0.3 hadoop03.domain.name
```

## Hadoop Admin

```
sudo addgroup hadoop_group
sudo adduser --ingroup hadoop_group hadoop_admin
usermod -aG sudo hadoop_admin
```

switch to hadoop_admin
```
su hadoop_admin
```

## SSH Key
```
su hadoop_admin
ssh-keygen -t rsa -P ""
cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys
```

## Install Java And Hadoop

### Java
```
sudo apt-get install openjdk-8-jdk
```

### Install Hadoop
use /usr/local/hadoop as HADOOP_HOME
```
cd ~
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
tar zxvf hadoop-3.3.6.tar.gz
sudo mv hadoop-3.3.6/ /usr/local/hadoop
```

### Environment
```
vim ~/.bashrc
```
add below
```
export HADOOP_HOME=/usr/local/hadoop

export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=$HADOOP_HOME/lib/native"

export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_CLASSPATH=$JAVA_HOME/lib/tools.jar
```

make environment variables effective
```
source ~/.bashrc
```

check variable
```
echo $HADOOP_HOME
```

## Setting HDFS Config

### CORE
```
vim /usr/local/hadoop/etc/hadoop/core-site.xml
```

refer to [core-site.xml](./core-site.xml)

### HDFS

```
vim /usr/local/hadoop/etc/hadoop/hdfs-site.xml
```

refer to [hdfs-site.xml](./hdfs-site.xml)

### Works

```
sudo vim /usr/local/hadoop/etc/hadoop/workers
```
add below
```
hadoop01.domain.name
hadoop02.domain.name
hadoop03.domain.name
```

### Environment

```
sudo vim /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```
add below
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64

export HDFS_NAMENODE_USER="hadoop_admin"
export HDFS_DATANODE_USER="hadoop_admin"
export HDFS_SECONDARYNAMENODE_USER="hadoop_admin"
export YARN_RESOURCEMANAGER_USER="hadoop_admin"
export YARN_NODEMANAGER_USER="hadoop_admin"
```

### Mount Disk
create file and change mode  
```
sudo mkdir /mnt/hadoop
sudo chmod -R 777 /mnt/hadoop
```

## Start
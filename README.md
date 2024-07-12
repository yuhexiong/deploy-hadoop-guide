# Deploy Hadoop Guide


## Overview

```
10.0.0.1 hadoop01.domain.name: NameNode DataNode
10.0.0.2 hadoop02.domain.name: SecondaryNameNode DataNode
10.0.0.3 hadoop03.domain.name: DataNode
10.0.0.4 dns
```
## IP And Host

### setting IP

```
sudo vim /etc/netplan/00-installer-config.yaml
```
refer to [00-installer-config.yaml](./00-installer-config.yaml)

### setting hostname and hosts

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

## hadoop admin

```
sudo addgroup hadoop_group
sudo adduser --ingroup hadoop_group hadoop_admin
usermod -aG sudo hadoop_admin
```

switch to hadoop_admin
```
su hadoop_admin
```

## SSH key
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

## Setting HDFS

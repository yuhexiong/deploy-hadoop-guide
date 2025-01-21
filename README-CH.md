# Deploy Hadoop Guide  
在三台虛擬機上部署 Apache Hadoop 的指南。  

![image](hadoop.png)

## Overview  

- 虛擬機：Ubuntu v22.04.4  
- 平台：JDK 8  
- 系統：Hadoop v3.3.6  

### Architecture  

**10.0.0.1 hadoop01**：NameNode DataNode  
**10.0.0.2 hadoop02**：SecondaryNameNode DataNode  
**10.0.0.3 hadoop03**：DataNode  
**10.0.0.4**：DNS（非必要）  
掛載於 **/mnt/hadoop**  

## IP And Host  

### 設定 IP  

```bash
sudo vim /etc/netplan/00-installer-config.yaml
```  
參考 [00-installer-config.yaml](./00-installer-config.yaml)  

### 設定 Hostname 和 Hosts  

```bash
sudo vim /etc/hostname
```  
修改為 hadoop01  

```bash
sudo vim /etc/hosts
```  
新增以下內容  
```
10.0.0.1 hadoop01
10.0.0.2 hadoop02
10.0.0.3 hadoop03
```  

設定完成後重新啟動虛擬機  
```bash
sudo reboot
```  

## Hadoop Admin  

```bash
sudo addgroup hadoop_group
sudo adduser --ingroup hadoop_group hadoop_admin
sudo usermod -aG sudo hadoop_admin
```  

切換到 hadoop_admin  
```bash
su hadoop_admin
cd ~
```  

## SSH Key  
```bash
ssh-keygen -t rsa -P ""
cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys
```  

## 安裝 Java 和 Hadoop  

```bash
sudo apt-get update
```  

### Java  
```bash
sudo apt-get install openjdk-8-jdk
```  

### 安裝 Hadoop  
使用 `/usr/local/hadoop` 作為 HADOOP_HOME  

```bash
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
tar zxvf hadoop-3.3.6.tar.gz
sudo mv hadoop-3.3.6/ /usr/local/hadoop
```  

### 環境變數  
```bash
vim ~/.bashrc
```  
新增以下內容  
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

使環境變數生效  
```bash
source ~/.bashrc
```  

檢查變數  
```bash
echo $HADOOP_HOME
```  

## 設定 HDFS 配置  

### CORE  
```bash
vim /usr/local/hadoop/etc/hadoop/core-site.xml
```  
參考 [core-site.xml](./core-site.xml)  

設定：  
- fs.defaultFS 指向 hadoop01  
- hadoop.tmp.dir 設為 `/mnt/hadoop`，若出現錯誤，可以刪除此目錄並重啟。  

### HDFS  

```bash
vim /usr/local/hadoop/etc/hadoop/hdfs-site.xml
```  
參考 [hdfs-site.xml](./hdfs-site.xml)  

設定：  
- NameNode 指向 hadoop01  
- NameNode 和 DataNode 的 tmp.dir 設為 `/mnt/hadoop`，若出現錯誤，可以刪除此目錄並重啟。  
- SecondaryNameNode 指向 hadoop02  

### Workers（DataNode）  

```bash
sudo vim /usr/local/hadoop/etc/hadoop/workers
```  
新增以下內容  
```
hadoop01
hadoop02
hadoop03
```  

### Environment  

```bash
sudo vim /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```  
新增以下內容  
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64

export HDFS_NAMENODE_USER="hadoop_admin"
export HDFS_DATANODE_USER="hadoop_admin"
export HDFS_SECONDARYNAMENODE_USER="hadoop_admin"
export YARN_RESOURCEMANAGER_USER="hadoop_admin"
export YARN_NODEMANAGER_USER="hadoop_admin"
```  

### 掛載磁碟  
建立目錄並更改權限  
```bash
sudo mkdir -p /mnt/hadoop
sudo chmod -R 777 /mnt/hadoop
```  

## 啟動  

將虛擬機複製為三份並修改 IP 和 Hostname，**不需要其他設定調整**。  

### 格式化 NameNode（在 hadoop01 執行）  
```bash
cd $HADOOP_HOME
bin/hdfs namenode -format
```  

### 啟動所有服務（在 hadoop01 執行）  
```bash
sbin/start-all.sh
```  

## 狀態檢查  

### JPS（每台虛擬機執行）  
```bash
jps
```  
依我們的架構設定，應預期如下結果  
```
2132 NameNode
2265 DataNode
7546 NodeManager
9295 Jps
```  

### HDFS（在 hadoop01 執行）  
```bash
hdfs dfsadmin -report
```  
預期看到三個 DataNode，如下所示  
```
Configured Capacity: 12983532773376 (11.81 TB)
Present Capacity: 12323766214656 (11.21 TB)
DFS Remaining: 12323766124544 (11.21 TB)
DFS Used: 90112 (88 KB)
DFS Used%: 0.00%
Replicated Blocks:
        Under replicated blocks: 0
        Blocks with corrupt replicas: 0
        Missing blocks: 0
        Missing blocks (with replication factor 1): 0
        Low redundancy blocks with highest priority to recover: 0
        Pending deletion blocks: 0
Erasure Coded Block Groups:
        Low redundancy block groups: 0
        Block groups with corrupt internal blocks: 0
        Missing block groups: 0
        Low redundancy blocks with highest priority to recover: 0
        Pending deletion blocks: 0

-------------------------------------------------
Live datanodes (3):

Name: 10.0.0.1:9866 (hadoop01)
Hostname: hadoop01
Decommission Status : Normal
Configured Capacity: 4327844257792 (3.94 TB)
DFS Used: 339968 (332 KB)
Non DFS Used: 2154496 (2.05 MB)
DFS Remaining: 4107922767872 (3.74 TB)
DFS Used%: 0.00%
DFS Remaining%: 94.92%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 0
Last contact: Mon Jul 15 09:28:38 UTC 2024
Last Block Report: Mon Jul 15 07:58:02 UTC 2024
Num of Blocks: 34

...
```  
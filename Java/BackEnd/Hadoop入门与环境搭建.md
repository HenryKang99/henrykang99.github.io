## Overview

[参考尚硅谷Hadoop教程](https://www.bilibili.com/video/BV1Qp4y1n7EN)

[hadoop docs](https://hadoop.apache.org/docs/)

**大数据：**

> **大数据（big data）**，是指无法在一定时间范围内用常规软件工具进行**采集**、**存储**、**分析处理**的数据集合，是需要新处理模式才能具有更强的决策力、洞察发现力和流程优化能力的**海量、高增长率和多样化**的信息资产。
>
> 大数据的 5V 特点：Volume（大量）、Velocity（高速）、Variety（多样）、Value（低价值密度）、Veracity（真实性）。
>

**大数据部门组织结构：**

<img src="D:/OneDrive/_mine/docsify/_img/image-20210511173318523.png" alt="大数据部门组织架构" style="zoom:80%;" />



## 概述

> Hadoop 是 Apache 开发的用于解决海量数据**存储**与分析**计算**问题的**分布式系统基础架构**。
>
> 特点：高可靠、高扩展、高效、高容错。

### Hadoop 组成

 3.x 在组成上没有变化。2.x 加入了 YARN 将计算和调度分离开来。

<img src="D:/OneDrive/_mine/docsify/_img/image-20210511173900249.png" alt="1.x、2.x 区别" style="zoom: 67%;" />

#### HDFS

> Hadoop Distribute File System，一个 HDFS 集群由一个 NameNode（管理文件系统所有元数据），多个 DataNode（管理数据的存储）组成。

HDFS 中，一个文件被分为许多存在一系列 DataNode 上的块（block）。NameNode 负责管理 block 到 DataNode 的映射，同时负责下达例如打开、关闭、重命名等文件元数据（Metadata）相关的操作指令。DataNode 根据来自 NameNode 或 Client 的指令，负责 block 的创建、销毁、复制等操作。

<img src="D:/OneDrive/_mine/docsify/_img/hdfsarchitecture.png" alt="HDFS Architecture" style="zoom:67%;" />

#### MapReduce

> Hadoop MapReduce 是一个软件框架，用于编写可靠的并行处理数以千计的节点上海量数据的程序。

**一个 MapReduce job 分为两步：**

- map task：将输入的数据集划分为多个独立的 chunk。

- reduce tasks：map 的结果经排序处理，作为 reduce 的输入，进行计算。

这些任务产生的输入和输出都会被存在文件系统中（即**基于磁盘计算**），框架管理着这些任务的执行顺序，并确保执行失败后重新执行。

通常情况下，计算节点和存储节点是对应的，在哪里存就在哪里算，这样框架可以有效地调度任务，节省了集群的带宽。

#### YARN

> Yet Another Resource Negotiator，出现的目的是将资源管理和任务调度分开。

![MapReduce NextGen Architecture](D:/OneDrive/_mine/docsify/_img/yarn_architecture.gif)

**组成：**

- ResourceManager（RM）：大管家，拥有绝对的资源（CPU、memory、disk、network）分配权限，包含 Scheduler 和 ApplicationManager 两个模块。
  - Scheduler：负责根据容量、队列等约束条件，分配资源给正在运行的程序。它根据程序对资源的申请进行调度，只是一个单纯的调度程序，不具备监管和追踪的功能，无法对失败的任务进行重做。调度器采用插件式策略，如可选择CapacityScheduler、 FairScheduler等。
  - ApplicationManager （AM）：负责接收客户端提交的作业，与特定的 ApplicationMaster 协商分配第一个容器，并负责失败重做。
- NodeManager（NM）：管理每个节点 Container 的资源分配，向 RM 汇报资源使用状态。
  - Container：封装执行任务所需资源的抽象容器。
- ApplicationMaster：与 RM 进行协商请求合适的资源，并负责执行、监管任务。

### Hadoop 生态

![HadoopEcosystem](D:/OneDrive/_mine/docsify/_img/image-20210512171150173.png)

## 环境搭建

> 使用 JDK 1.8、Hadoop 3.1.3

### VM 不支持64位系统问题
三种情况：
1. 开启了 hyper-v，在`控制面板-启用或关闭 Windows 功能`中取消勾选，重启。
2. 未开启虚拟化，进入 BIOS 进行设置。
3. 管理员 cmd 输入 `bcdedit` 查看 `hypervisorlaunchtype`，使用命令`bcdedit /set hypervisorlaunchtype off`将其设为 off 并重启。
	
### 最小安装后需要做的事
#### 配置网络
> [这篇文章写得很清楚](https://zhuanlan.zhihu.com/p/130984945)
>
> 配置 NAT 和 虚拟网卡（适配器）在同一网段，并与主机在不同网段。

1. 配置虚拟网卡，虚拟网卡作用是供虚拟机和主机通信。

  <img src="D:\OneDrive\_mine\docsify\_img\image-20210623131913844.png" alt="image-20210623131913844" style="zoom: 67%;" />

2. 在 VM 虚拟网络编辑器中配置 NAT，需要同虚拟网卡在相同网段。配置静态IP的话，可以将`使用本地DNCP服务器xxx`选项取消。

  ![配置NAT](D:/OneDrive/_mine/docsify/_img/setvmnetwork.png)

3. 配置静态 IP：`vi /etc/sysconfig/network-scripts/ifcfg-ens33`，配置完后重启网络服务 `service network restart`。

  ![image-20210623132844903](D:\OneDrive\_mine\docsify\_img\image-20210623132844903.png)

4. ping 一下主机和外网，主机 ping 不通的话需要配置一下 windows 的防火墙，允许 ICMP 通过。

5. 关闭网火墙与自启：

   ```bash
   systemctl stop firewalld.service
   systemctl disable firewalld.service
   ```

6. 修改主机名和 hosts：分别在 `etc/` 下的 `hostname` 和 `hosts` 进行配置。

#### 安装工具

```bash
yum update && yum upgrade
yum install net-tools
yum install rsync
yum install vim
# 安装额外软件仓库epel-release
yum install epel-release
```

#### 添加用户并配置权限

```bash
# 添加用户
useradd user1
passwd user1

# 配置user1具有root权限
# 以后user1用户执行sudu时，就不需要输入密码了
vim /etc/sudoers
# 在 %wheel ALL=(ALL) ALL 下面添加
user1 ALL=(ALL) NOPASSWD:ALL
```

### 安装相关软件

> 如果不是最小安装，需要先卸载自带的 Java 等：
>
> rpm -qa | grep -i java | xargs -n1 rpm -e --nodeps  

#### 安装 JDK、Hadoop

```bash
# 目录结构：
# /opt
#   |--/software  包
#	|--/module	解压放这里
tar -zxvf xxxxxx.tar.gz -C /opt/module/
```

#### 配置环境变量

```bash
# 1.可以添加到最后面
vim /etc/profile
# 观察改文件最后的脚本
for i in /etc/profile.d/*.sh /etc/profile.d/sh.local ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then
            . "$i"
        else
            . "$i" >/dev/null
        fi
    fi
done
# 作用是让 /etc/profile.d/ 下以.sh结尾的文件中配置的环境变量生效
# 2.故也可以在该文件夹下创建 .sh 配置环境变量
vim /etc/profile.d/my_env.sh
# 添加：
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_212
export PATH=$PATH:$JAVA_HOME/bin
#HADOOP_HOME
export HADOOP_HOME=/opt/module/hadoop-3.1.3
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
# 生效一下
source /etc/profile

# 验证
java -version
hadoop version
```

### 克隆虚拟机

关机、克隆出两台虚拟机并配置主机名称、网络。

正确关机姿势：`shutdown -h now`

### 配置密钥登录

目的：方便本地-服务器、服务器-服务器之间建立链接，免去密码输入过程。

方法：要实现 A 能免密登陆 B，需要将 A 的公钥发送给 B，让 B 保存在 Authorized_keys 中。

服务器-服务器：使用`ssh-copy-id`命令。

```bash
# 生成密钥
ssh-keygen -t rsa
# 位于用户家目录下的隐藏文件夹 .ssh/ 下
ls -al 
# 该命令将A的公钥复制到B的Authorized_keys中，需要输入一次密码，后续链接将不需要密码
ssh-copy-id hadoop102
```

本地（win）-服务器：手动 copy 公钥到 Authorized_keys 下。

### 数据传输：scp 与 rsync

**scp：**复制

```bash
# 命令：scp -r 源文件或目录 目标地址
# 参数：-r 表示递归
# 举例：
    # 本机 --> 服务器A
    scp -r path/file  user@hostA:path/
    # 本机 <-- 服务器A
    scp -r user@hostA:path/file  path/
    # 服务器A --> 服务器B
    scp -r user@hostA:path/file  user@hostB:path/
```

**rsync：**同步

```bash
# 如果没有安装
yum install -y rsync
# rsync -av 源文件或目录 目标地址
# 参数：-a 归档拷贝；-v 显示复制过程
# 举例：
	# 同步本地和服务器A的module文件夹
	rsync -av module/ user@hostA:/opt/module/
```

**编写脚本：**便于后期多个服务器同步文件

```bash
#!/bin/bash

#1. 判断参数个数
if [ $# -lt 1 ]
then
  echo Not Enough Arguement!
  exit;
fi

#2. 遍历集群所有机器
for host in hadoop101 hadoop102 hadoop103
do
  # 如果是本机则跳过
  if [ $HOSTNAME == $host ]
  then
    continue
  fi
  echo ==================== $host ====================
  #3. 遍历所有目录，挨个发送
  for file in $@
  do
    #4. 判断文件是否存在
    if [ -e $file ]
    then
      #5. 获取父目录
      pdir=$(cd -P $(dirname $file); pwd)
      #6. 获取当前文件的名称
      fname=$(basename $file)
      ssh $host "mkdir -p $pdir"
      rsync -av $pdir/$fname $host:$pdir
    else
      echo $file does not exists!
    fi
  done
done
```

```bash
# 修改权限
chmod +x xsync
# 将脚本放到 ~/bin 中以方便全局调用
```

## 集群配置

> 规划：NameNode、SecondaryNameNode、RresourceManager 分别部署在 hadoop101-103 上。

### 配置文件说明

**主要目录：**

bin：存放 Hadoop 各组件相关脚本

sbin：存放 Hadoop 启动关闭等脚本

etc：存放自定义配置文件

lib：本地库

share：存放依赖、文档、官方案例等



Hadoop 配置文件分为两类，分别是**默认配置文件**和**自定义配置文件**。

- 默认配置文件位于 `/share` 下的 jar 包内，分别是：
  - hadoop-common-3.1.3.jar/core-default.xml
  - hadoop-hdfs-3.1.3.jar/hdfs-default.xml
  - hadoop-yarn-common-3.1.3.jar/yarn-default.xml
  - hadoop-mapreduce-client-core-3.1.3.jar/mapred-default.xml

- 自定义配置文件放在 `etc/hadoop` 下，分别是：core-site.xml、hdfs-site.xml、yarn-site.xml、mapred-site.xml

### 开始配置

1. core-site.xml

```xml
<configuration>
    <!-- 指定 NameNode 的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop101:8020</value>
    </property>
    <!-- 指定 hadoop 数据的存储目录 -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-3.1.3/data</value>
    </property>
    <!-- 配置 HDFS 网页登录使用的静态用户为 user1 -->
    <property>
        <name>hadoop.http.staticuser.user</name>
        <value>user1</value>
    </property>
</configuration>
```

2. hdfs-site.xml

```xml
<configuration>
    <!-- nn web 端访问地址-->
    <property>
        <name>dfs.namenode.http-address</name>
        <value>hadoop101:9870</value>
    </property>
    <!-- 2nn web 端访问地址-->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop102:9868</value>
    </property>
</configuration>
```

3. yarn-site.xml

```xml
<configuration>
    <!-- 指定 MR 使用 shuffle -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <!-- 指定 ResourceManager 的地址-->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop103</value>
    </property>
    <!-- 环境变量的继承 -->
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

4. mapred-site.xml

```xml
<configuration>
    <!-- 指定 MapReduce 程序运行在 Yarn 上 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

5. 除了配置文件外，还需要配置一个 workers 来群起集群。

```bash
vim etc/hadoop/workers
# 修改为
hadoop101
hadoop102
hadoop103
```

6. 配置好了后，同步到其他服务器。

### 启动集群

第一次启动时，需要手动初始化 NameNode，这时会分配一个集群 id。

```bash
# 在部署nn的hadoop101进行初始化
hdfs namenode -format
```

执行后，hadoop 根目录会多出 `data` 、 `logs` 两个目录。

进入 `data/dfs/name/current/` 目录，查看 `VERSION`，里面记录了集群 id。

**启动：**

```bash
# 1. 启动集群
sbin/start-dfs.sh

# 2. 在 部署 ResourceManager 的 Hadoop103 启动 yarn 
sbin/start-yarn.sh

# jps：Java Virtual Machine Process Status Tool
# jps 命令查看已启动的 java 进程：

#hadoop101
12963 NodeManager
13107 Jps
12540 NameNode
12655 DataNode
#hadoop102
12325 NodeManager
12072 SecondaryNameNode
12474 Jps
12012 DataNode
#hadoop103
12631 NodeManager
11851 DataNode
12076 ResourceManager
12767 Jps
```

**验证：**

访问：http://hadoop101:9870/，看下集群情况、文件信息。

访问：http://hadoop103:8088/，查看任务信息。

### 测试

1. 测试存储

```bash
# 创建一个文件夹
hadoop fs -mkdir /wcinput
# 上传小文件
 hadoop fs -put README.txt /wcinput
 # 上传大文件
hadoop fs -put /opt/software/jdk-8u212-linux-x64.tar.gz /
# http://hadoop101:9870/ 查看一下

# 问题：数据存在了哪里？
pwd
# /opt/module/hadoop-3.1.3/data/dfs/data/current/BP-355040450-192.168.100.101-1624541219298/current/finalized/subdir0/subdir0
cd data/dfs/data/current/
cd BP-xxxx/current/finalized/subdir0/subdir0
cat blk_xxx

# 观察上传的大文件jdk被分成了两块
# 在其他服务器观察，文件存的都一样，因为默认存放三份
```

2. 测试 yarn 

```bash
# 运行官方demo:wordcount
hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount /wcinput/README.txt /wcoutput
```

### 配置历史服务器与日志聚集

1. 在 mapred-site.xml 追加

```xml
<!-- 历史服务器端地址 -->
<property>
    <name>mapreduce.jobhistory.address</name>
    <value>hadoop102:10020</value>
</property>
<!-- 历史服务器 web 端地址 -->
<property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>hadoop102:19888</value>
</property>
```

2. 在 yarn-site.xml 追加

```xml
<!-- 开启日志聚集功能 -->
<property>
    <name>yarn.log-aggregation-enable</name>
    <value>true</value>
</property>
<!-- 设置日志聚集服务器地址 -->
<property>
    <name>yarn.log.server.url</name>
    <value>http://hadoop101:19888/jobhistory/logs</value>
</property>
<!-- 设置日志保留时间为 7 天 -->
<property>
    <name>yarn.log-aggregation.retain-seconds</name>
    <value>604800</value>
</property>
```

**启动：**

```bash
# 配置日志收集后，需要重启 NM、RM、HistoryServer
stop-yarn.sh
mapred --daemon stop historyserver

start-yarn.sh
mapred --daemon start historyserver
```

**验证：**

- 访问 http://hadoop101:19888/jobhistory 查看历史任务
- 点击任务后面的 logs 查看日志

### 编写启动、停止脚本

**常用启停命令：**

```bash
# 整体启停
start-dfs.sh
stop-dfs.sh
start-yarn.sh
stop-yarn.sh

# 单个组件启停
hdfs --daemon start/stop namenode/datanode/secondarynamenode
```

1. myhadoop.sh 集群启停

```bash
#!/bin/bash
if [ $# -lt 1 ]
then
    echo "No Args Input..."
    exit;
fi
case $1 in
"start")
    echo " =================== 启动 hadoop 集群 ==================="
    echo " --------------- 启动 hdfs ---------------"
    ssh hadoop101 "/opt/module/hadoop-3.1.3/sbin/start-dfs.sh"
    echo " --------------- 启动 yarn ---------------"
    ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/start-yarn.sh"
    echo " --------------- 启动 historyserver ---------------"
    ssh hadoop101 "/opt/module/hadoop-3.1.3/bin/mapred --daemon start historyserver"
;;
"stop")
    echo " =================== 关闭 hadoop 集群 ==================="
    echo " --------------- 关闭 historyserver ---------------"
    ssh hadoop101 "/opt/module/hadoop-3.1.3/bin/mapred --daemon stop historyserver"
    echo " --------------- 关闭 yarn ---------------"
    ssh hadoop103 "/opt/module/hadoop-3.1.3/sbin/stop-yarn.sh"
    echo " --------------- 关闭 hdfs ---------------"
    ssh hadoop101 "/opt/module/hadoop-3.1.3/sbin/stop-dfs.sh"
;;
*)
	echo "Input Args Error..."
;;
esac
```

2. jpsall.sh

```bash
#!/bin/bash
for host in hadoop101 hadoop102 hadoop103
do
    echo =============== $host ===============
    ssh $host jps
done
```

注意：赋予执行权限并放置在 `~/bin` 下，同步到其他服务器。
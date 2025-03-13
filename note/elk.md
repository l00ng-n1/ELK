# ELK概述

![image-20250119195308988](pic/image-20250119195308988.png)

-   **Elasticsearch** 是一个实时的全文搜索,存储库和分析引擎。
-   **Logstash** 是数据处理的管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到诸如 Elasticsearch 等存储库中。
-   **Kibana** 则可以让用户在 Elasticsearch 中使用图形和图表对数据进行可视化。

目前 Elastic Stack 中除了Elasticsearch、Logstash 和 Kibana, 还包括一系列丰富的轻量型数据采集代 理，这些代理统称为 **Beats**，可用来向 Elasticsearch 发送数据。

**ELK stack的主要优点：**

-   **功能强大**：Elasticsearch 是实时全文索引，具有强大的搜索功能
-   **配置相对简单**：Elasticsearch 全部其于 JSON，Logstash使用模块化配置，Kibana的配置都比较简单。
-   **检索性能高效**：基于优秀的设计，每次查询可以实时响应，即使百亿级数据的查询也能达到秒级响应。
-   **集群线性扩展**：Elasticsearch 和 Logstash都可以灵活线性扩展
-   **前端操作方便**：Kibana提供了比较美观UI前端，操作也比较简单

**EFK** 由**ElasticSearch**、**Fluentd**和**Kibana**三个开源工具组成。

**Fluentd**是一个实时开源的数据收集器,和logstash功能相似,这三款开源工具的组合为日志数据提供了分布式的实时搜集与分析的监控系统。

## Elasticsearch

### Elasticsearch 介绍

Elasticsearch 是一个分布式的免费开源搜索和分析引擎，适用于包括文本、数字、地理空间、结构化和 非结构化数据等在内的所有类型的数据。

### ELasticsearch原理

原始数据会从多个来源（包括日志、系统指标和网络应用程序）输入到 Elasticsearch 中。数据采集指在 Elasticsearch 中进行索引之前解析、标准化并充实这些原始数据的过程。这些数据在 Elasticsearch 中 索引完成之后，用户便可针对他们的数据运行复杂的查询，并使用聚合来检索自身数据的复杂汇总。

Elasticsearch 索引指**相互关联的文档集合**。Elasticsearch 会以 JSON 文档的形式存储数据。每个文档都会在一组键（字段或属性的名称）和它们对应的值（字符串、数字、布尔值、日期、数组、地理位置或 其他类型的数据）之间建立联系。

Elasticsearch 使用的是一种名为倒排索引的数据结构，这一结构的设计可以允许十分快速地进行全文本 搜索。倒排索引会列出在所有文档中出现的每个特有词汇，并且可以找到包含每个词汇的全部文档。

在索引过程中，Elasticsearch 会存储文档并构建倒排索引，这样用户便可以近乎实时地对文档数据进行 搜索。索引过程是在索引 API 中启动的，通过此 API 您既可向特定索引中添加 JSON 文档，也可更改特 定索引中的 JSON 文档

#### 基本概念

**Cluster 集群**

群集是一个或多个节点（服务器）的集合， 这些节点共同保存整个数据，并在所有节点上提供联合索引 和搜索功能。一个集群由一个唯一集群ID确定，并指定一个集群名（默认为“elasticsearch”）。该集群 名非常重要，因为节点可以通过这个集群名加入群集，一个节点只能是群集的一部分。

确保在不同的环境中不要使用相同的群集名称，否则可能会导致连接错误的群集节点。

**Node 节点**

节点是单个服务器实例，它是集群的一部分，可以存储数据，并参与群集的索引和搜索功能。就像一个 集群，节点的名称默认为一个随机的通用唯一标识符（UUID），确定在启动时分配给该节点。如果不希望默认，可以定义任何节点名。这个名字对管理很重要，目的是要确定网络服务器对应于ElasticSearch 群集节点。

我们可以通过集群名配置节点以连接特定的群集。默认情况下，每个节点设置加入名为“elasticSearch” 的集群。这意味着如果启动多个节点在网络上，假设他们能发现彼此都会自动形成和加入一个名为 “elasticsearch”的集群。

单个群集中，您可以拥有尽可能多的节点。此外，如果“elasticsearch”在同一个网络中，没有其他节 点正在运行，从单个节点的默认情况下会形成一个新的单节点名为"elasticsearch"的集群

**Index 索引**

索引是**具有相似特性的文档集合**。例如，可以为客户数据提供索引，为产品目录建立另一个索引，以及 为订单数据建立另一个索引。索引由名称**（必须全部为小写）**标识，该名称用于在对其中的文档执行索 引、搜索、更新和删除操作时引用索引。在单个群集中，您可以定义尽可能多的索引。

```
注意: 索引名不支持大写字母
```

**Document 文档**

文档是可以被索引的信息的基本单位。例如，您可以为单个客户提供一个文档，单个产品提供另一个文 档，以及单个订单提供另一个文档。本文件的表示形式为JSON（JavaScript Object Notation）格式，这 是一种非常普遍的互联网数据交换格式。

在索引/类型中，您可以存储尽可能多的文档。请注意，尽管文档物理驻留在索引中，**文档实际上必须索引**或分配到索引中的类型。

**Shards & Replicas 分片与副本**

索引可以存储大量的数据，这些数据可能超过单个节点的硬件限制。例如，十亿个文件占用磁盘空间 1TB的单指标可能不适合对单个节点的磁盘, 或者仅从单个节点的搜索请求服务可能太慢

为了解决这一问题，Elasticsearch提供细分指标分成多个块称为分片的能力。当创建一个索引，可以简 单地定义想要的分片数量。每个分片本身是一个全功能的、独立的“指数”，可以托管在集群中的任何节 点。

Shards分片的重要性主要体现在以下两个特征：

-   分片允许您水平拆分或缩放内容的大小
-   分片允许你分配和并行操作的碎片（可能在多个节点上）从而提高性能/吞吐量

在同一个集群网络或云环境上，故障是任何时候都会出现的，拥有一个故障转移机制以防分片和结点因 为某些原因离线或消失是非常有用的，并且被强烈推荐。为此，Elasticsearch允许你创建一个或多个拷 贝，索引分片进入所谓的副本或称作复制品的分片，简称Replicas。

注意：**ES的副本**指不包括主分片的其它副本,**即只包括备份**，这与Kafka是不同的

Replicas的重要性主要体现在以下两个特征：

-   副本为分片或节点失败提供了高可用性。需要注意的是，一个副本的分片不会分配在同一个节点作 为原始的或主分片，副本是从主分片那里复制过来的。
-   副本允许用户扩展你的搜索量或吞吐量，因为搜索可以在所有副本上并行执行。

## Logstash

![image-20250119215819291](pic/image-20250119215819291.png)

Logstash 是 Elastic Stack 的核心产品之一，可用来对数据进行聚合和处理，并将数据发送到 Elasticsearch。

Logstash 是一个基于Java实现的开源的服务器端数据处理管道，允许您在将数据索引到 Elasticsearch 之前同时从多个来源采集数据，并对数据进行过滤和转换。

可以通过插件实现日志收集和转发，支持日志过滤，支持普通log、自定义json格式的日志解析。

## Kibana

Kibana 是一款适用于 Elasticsearch 的**基于Javascript语言**实现的数据可视化和管理工具，可以提供实时的直方图、线形图、饼状图和地图。Kibana 同时还包括诸如 Canvas 和 Elastic Maps 等高级应用程序； Canvas 允许用户基于自身数据创建定制的动态信息图表，而 Elastic Maps 则可用来对地理空间数据进行可视化。

# Elasticsearch 部署和管理

**部署方式**

-   **包安装**
-   **二进制安装**
-   **Docker 部署**
-   **Kubernetes 部署**
-   **Ansible 批量部署**

**ES支持操作系统版本和 Java 版本官方说明**

[支持一览表 | Elastic](https://www.elastic.co/cn/support/matrix)

## ELasticsearch安装前准备

### 安装前环境初始化

```shell
CPU 2C
内存4G或更多
操作系统: Ubuntu22.04,Ubuntu20.04,Ubuntu18.04,Rocky8.X,Centos 7.X
操作系统盘50G
主机名设置规则为nodeX.wang.org
生产环境建议准备单独的数据磁盘
```

主机名

```shell
#各服务器配置自己的主机名
hostnamectl set-hostname es-node1
```

各服务器配置本地域名解析

```shell
vim /etc/hosts
10.0.0.222 es-node1
10.0.0.223 es-node2
10.0.0.224 es-node3
```

内核参数 `vm.max_map_count` 用于限制一个进程可以拥有的VMA(虚拟内存区域)的数量

```shell
echo "vm.max_map_count = 262144" >> /etc/sysctl.conf
# 系统最大打开的文件描述符数 fs.file-max 大于1000000
# echo "fs.file-max = 1000000" >> /etc/sysctl.conf
sysctl -p
```

关于JDK环境说明

```bash
1.x 2.x 5.x 6.x都没有集成JDK的安装包，也就是需要自己安装java环境
7.x 版本的安装包分为带JDK和不带JDK两种包，带JDK的包在安装时不需要再安装java，如果不带JDK的包
仍然需要自己去安装java
8.X 版本内置JDK，不再支持自行安装的JDK
```

## Elasticsearch 安装

### 包安装

```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.17.3-amd64.deb
sudo dpkg -i elasticsearch-8.17.3-amd64.deb
```

重置生成新密码

```shell
# 方法1：生成随机密码
/usr/share/elasticsearch/bin/elasticsearch password -u elastic

# 方法2：交互式生成指定密码
/usr/share/elasticsearch/bin/elasticsearch-reset-password --username elastic -i
```

在实际生产中，Elasticsearch会不可避免的和各种其他服务进行通信，这个过程中该认证会产生很多麻烦，所以在内网安全可到保证的情况下，建议把该安全加固取消

```shell
vim /etc/elasticsearch/elasticsearch.yml
xpack.security.enabled: false

vim /etc/elasticsearch/jvm.options
-Xms1g
-Xmx1g
```



### 二进制安装Elasticsearch

```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.17.3-linux-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.17.3-linux-x86_64.tar.gz.sha512
shasum -a 512 -c elasticsearch-8.17.3-linux-x86_64.tar.gz.sha512 
tar -xf elasticsearch-8.17.3-linux-x86_64.tar.gz
cd elasticsearch-8.17.3/ 
mv elasticsearch-8.17.3 /usr/local/
ln -s /usr/local/elasticsearch-8.17.3 /usr/local/elasticsearch
```

**编辑服务配置文件（集群配置）**

```shell
vim /usr/local/elasticsearch/config/elasticsearch.yml
# 关闭安全功能
xpack.security.enabled: false
# 改数据和日志目录
path.data: /usr/local/elasticsearch/data/es-data
path.logs: /usr/local/elasticsearch/data/es-logs
http.host: 0.0.0.0 
network.host: 0.0.0.0 
```

**修改ELK内存配置**

修改ELK内存配置，推荐使用宿主机物理内存的一半，最大不超过30G

```
vim /usr/local/elasticsearch/config/jvm.options
-Xms1g
-Xmx1g
```

**创建用户**

从ES7.X以后版不允许以root启动服务，需要委创建专用的用户

```shell
useradd -r elasticsearch
```

**目录权限更改**

在所有节点上创建数据和日志目录并修改目录权限为elasticsearch

```shell
chown -R elasticsearch:elasticsearch /usr/local/elasticsearch/
```

**创建service文件**

```shell
vim /lib/systemd/system/elasticsearch.service
[Unit]
Description=Elasticsearch
Documentation=http://www.elastic.co
Wants=network-online.target
After=network-online.target

[Service]
RuntimeDirectory=elasticsearch
PrivateTmp=true
Environment=PID_DIR=/var/run/elasticsearch
WorkingDirectory=/usr/local/elasticsearch
User=elasticsearch
Group=elasticsearch
ExecStart=/usr/local/elasticsearch/bin/elasticsearch -p ${PID_DIR}/elasticsearch.pid --quiet
# StandardOutput is configured to redirect to journalctl since
# some error messages may be logged in standard output before
# elasticsearch logging system is initialized. Elasticsearch
# stores its logs in /var/log/elasticsearch and does not use
# journalctl by default. If you also want to enable journalctl
# logging, you can simply remove the "quiet" option from ExecStart.
# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65535
# Specifies the maximum number of processes
LimitNPROC=4096
# Specifies the maximum size of virtual memory
LimitAS=infinity
# Specifies the maximum file size
LimitFSIZE=infinity
# Disable timeout logic and wait until process is stopped
TimeoutStopSec=0
# SIGTERM signal is used to stop the Java process
KillSignal=SIGTERM
# Send the signal only to the JVM rather than its control group
KillMode=process
# Java process is never killed
SendSIGKILL=no
# When a JVM receives a SIGTERM signal it exits with code 143
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target

systemctl daemon-reload
```

**启动ELasticsearch服务**

```shell
systemctl start elasticsearch.service 
```

## 多节点集群部署

此方式需要3G以上内存，否则会出现OOM报错

修改内核参数

```shell
echo vm.max_map_count=262144 >> /etc/sysctl.conf
sysctl -p
```

9200端口：用于web访问
9300端口：用于集群内nodes之间通信

集群配置

```shell
cluster.name: my-application #指定集群名称，同一个集群内的所有节点相同
node.name: node-1            #修改此行，每个节点不同

# 集群模式必须修改此行，默认是127.0.0.1:9300,否则集群节点无法通过9300端口通信，每个节点相同
network.host: 0.0.0.0     
# 用于发现集群内节点地址
discovery.seed_hosts: ["10.0.0.222", "10.0.0.223", "10.0.0.224"] 
# 指定允许参与主节点选举的nodes
cluster.initial_master_nodes: ["10.0.0.222", "10.0.0.223", "10.0.0.224"]

# 下面这行和上面的相同，将其注释掉防止冲突
# cluster.initial_master_nodes: ["ubuntu2204.wang.org"]     #将此行注释
```

```shell
systemctl restart elasticsearch

curl 10.0.0.222:9200/_cat/nodes
```



## 优化ELK资源配置

### 预留充足的内存空间

**开启 bootstrap.memory_lock: true 可以优化性能，但会导致无法启动的错误解决方法**

开启 `bootstrap.memory_lock: true` 需要足够的内存，建议4G以上，否则内存不足，启动会很失败或很慢

作用：用于在启动前给`Elasticsearch`预留充足的内存空间

```shell
# 开启此功能建议堆内存大小设置是总内存的一半，也就是内存充足的情况下使用
vim /etc/elasticsearch/elasticsearch.yml
#开启此功能导8.X致集群模式无法启动,但单机模式可以启动
bootstrap.memory_lock: true

# 8.X致集群模式需要修改如下配置
[Service]
#加下面一行
LimitMEMLOCK=infinity
```

### 内存优化

**推荐使用宿主机物理内存的一半，ES的heap内存最大不超过30G,26G是比较安全的**

为了保证性能，每个ES节点的JVM内存设置具体要根据 node 要存储的数据量来估算,建议符合下面约定

-   在内存和数据量有一个建议的比例：对于一般日志类文件，1G 内存能存储**48G~96GB**数据
-   JVM 堆内存最大不要超过30GB
-   单个分片控制在30-50GB，太大查询会比较慢，索引恢复和更新时间越长；分片太小，会导致索引 碎片化越严重，性能也会下降

```shell
# 范例
#假设总数据量为1TB，3个node节点，1个副本；那么实际要存储的大小为2TB
每个节点需要存储的数据量为:2TB / 3 = 700GB，每个节点还需要预留20%的空间，所以每个node要存储大约 700*100/80=875GB 的数据；每个节点按照内存与存储数据的比率计算：875GB/48GB=18，即需要JVM内存为18GB,小于30GB
因为要尽量控制分片的大小为30GB；875GB/30GB=30个分片,即最多每个节点有30个分片

#思考：假设总数据量为2TB，3个node节点，1个副本呢？
```

### **修改service文件，做优化配置**

```shell
[root@es-node1 ~]# vim /usr/lib/systemd/system/elasticsearch.service 
LimitNOFILE=1000000       #修改最大打开的文件数，默认值为65535
LimitNPROC=65535          #修改打开最大的进程数，默认值为4096
LimitMEMLOCK=infinity     
```

## Elasticsearch面板

### Head插件

浏览器安装插件

```shell
https://www.mysticalrecluse.com/script/tools/ElasticSearch-Head-0.1.5_0.zip
```

浏览器中导入插件

![image-20250313181204617](pic/image-20250313181204617.png)

### Cerebro插件

[lmenezes/cerebro](https://github.com/lmenezes/cerebro)

cerebro needs Java 11 or newer to run.

```shell
apt install openjdk-11-jdk
wget https://github.com/lmenezes/cerebro/releases/download/v0.9.4/cerebro_0.9.4_all.deb
dpkg -i cerebro_0.9.4_all.deb

vim /etc/cerebro/application.conf
# Path of local database file
data.path: "/var/lib/cerebro/cerebro.db"                                                
#data.path = "./cerebro.db"

systemctl start cerebro.service

# 默认监听9000端口
# 访问浏览器：10.0.0.132:9000,并输入es集群IP连接
```

![image-20250313182949678](pic/image-20250313182949678.png)

![image-20250313182955013](pic/image-20250313182955013.png)

## ELasticsearch API访问
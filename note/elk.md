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

## Elasticsearch 集群扩容和缩容

### 集群扩容

新节点

```shell
vim /etc/elasticsearch/elasticsearch
cluster.name: my-application #和原集群名称相同
node.name: node4  #第二个新节点为node5
network.host: 0.0.0.0
#指定任意集群节点即可
discovery.seed_hosts: ["10.0.0.222","10.0.0.223","10.0.0.224"]
# 以下配置（选配）
# ELasticsearch8.x开始，使用新方式指定节点角色
# 如果是master节点
node.roles: [master]
# 如果是Data节点
node.roles: [data]
# 如果节点同时是 Master 和 Data 节点：
node.roles: [master, data]
# 如果节点是 Coordinating-only 节点（不存储数据也不当选主节点，仅用于查询分发）
#如果改为路由节点，需要先执行/usr/share/elasticsearch/bin/elasticsearch-node repurpose 清理数据
node.roles: []

systemctl restart elasticsearch.service
```

### 集群缩容

从集群中删除两个节点node4和node5，在两个节点按一定的顺序逐个停止服务，即可自动退出集群

注意：停止服务前，要观察索引的情况，按一定顺序关机，防止数据丢失

就是要注意：同一个节点上，不能有相同的分片
停服务的时候，要注意保证先把该节点上独有的分片，复制到其他集群节点上后，在删除该节点

**缺点关停的节点上的主分片在其他节点上有副本。**

节点一个个关停，要等待集群内的分片不缺少，

比如分片1的主分片与副本都在node3与node5上，就不能同时关停3，5.否则数据会缺失

![image-20250121204518014](pic/image-20250121204518014.png)

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

### 修改service文件，做优化配置

```shell
[root@es-node1 ~]# vim /usr/lib/systemd/system/elasticsearch.service 
LimitNOFILE=1000000       #修改最大打开的文件数，默认值为65535
LimitNPROC=65535          #修改打开最大的进程数，默认值为4096
LimitMEMLOCK=infinity     
```

## 冷热数据分离

为了保证Elasticsearch的读写性能，官方建议磁盘使用SSD固态硬盘。然而Elasticsearch要解决的是海 量数据的存储和检索问题，海量的数据就意味需要大量的存储空间，如果都使用SSD固态硬盘成本将成 为一个很大的问题，这也是制约许多企业和个人使用Elasticsearch的因素之一。为了解决这个问题， Elasticsearch冷热分离架构应运而生。

传统的Elasticsearch集群中所有节点均采用相同的配置，然而Elasticsearch并没有对节点的规格一致性 做要求，换而言之就是每个节点可以是任意规格，当然这样做会导致集群各节点性能不一致，影响集群 稳定性。但是如果有规则的将集群的节点分成不同类型，部分是高性能的节点用于存储热点数据，部分 是性能相对差些的大容量节点用于存储冷数据，却可以一方面保证热数据的性能，另一方面保证冷数据 的存储，降低存储成本，这也是Elasticsearch冷热分离架构的基本思想

### 配置方法

3热节点，2冷节点的冷热分离Elasticsearch集群

![image-20250317092123199](pic/image-20250317092123199.png)

### 指定节点的冷热属性

```shell
vim /etc/elasticsearch/elasticsearch.yml
node.attr.{attribute}: {value}
#其中attribute为用户自定义的任意标签名，value为该节点对应的该标签的值
node.attr.temperature: hot //热节点
node.attr.temperature: warm //冷节点
```

### 指定索引冷热属性

用户可以在创建索引，或后续的任意时刻设置这些配置来控制索引在不同标签节点上的分配动作。

```shell
index.routing.allocation.include.{attribute} #表示索引可以分配在包含多个逗号分隔的值中其中一个的节点上。
index.routing.allocation.require.{attribute} #表示索引要分配在包含索引指定多个逗号分隔的值的节点上,并且多个值都要匹配
index.routing.allocation.exclude.{attribute} #表示索引只能分配在不包含指定多个逗号分隔的值的节点上。

#对于热数据，索引设置如下
PUT hot_warm_test_index/_settings
{
  "index.routing.allocation.require.temperature": "hot"
}

#对于冷数据，索引设置
PUT hot_warm_test_index/_settings
{
  "index.routing.allocation.require.temperature": "warm"
}
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

**可以用三种方式和 Elasticsearch进行交互**

-   **curl 命令**和其它浏览器: 基于命令行,操作不方便
-   **插件**: 在node节点上安装head,Cerebro 等插件,实现图形操作,查看数据方便
-   **Kibana**: 需要java环境并配置,图形操作,显示格式丰富

### 查看ES集群状态

```shell
# 查看支持的指令
curl http://127.0.0.1:9200/_cat

#查看es集群状态，?v是详细显示 ?human是可阅读显示
curl http://127.0.0.1:9200/_cat/health

#查看集群健康性
curl http://127.0.0.1:9200/_cluster/health?pretty=true

#查看所有的节点信息
curl 'http://127.0.0.1:9200/_cat/nodes?v'

#列出所有的索引 以及每个索引的相关信息
curl 'http://127.0.0.1:9200/_cat/indices?v'
```

### 创建，查看，删除索引

```shell
#创建索引index1
curl -XPUT '127.0.0.1:9200/index1'
# 默认会自动生成一个副本

# 副本丢失会，再次创建副本
# 颜色含义：
# 绿色：完全健康
# 黄色：副本丢了，数据没丢
# 红色：数据丢失

# 因为少于半数以上可用，即3个节点的集群，关了两个，因此集群不可用，es无法访问

#创建索引index2,格式化输出
curl -XPUT '127.0.0.1:9200/index2?pretty'
#查看所有索引
curl 'http://127.0.0.1:9200/_cat/indices?v'
# 查看索引格式化输出
curl 'http://127.0.0.1:9200/index1?pretty'
```

![image-20250314093328649](pic/image-20250314093328649.png)

![image-20250314094633256](pic/image-20250314094633256.png)

```shell
# 创建3个分片和2个副本的索引
curl -XPUT '127.0.0.1:9200/test' -H 'Content-Type: application/json' -d '
{                     
  "settings": {
    "index": {
      "number_of_shards": 3,  
      "number_of_replicas": 2
   }                         
 }
}'

# 创建3分片，1副本（生产中建议分片数和节点数相同，实现负载均衡）
curl -XPUT '127.0.0.1:9200/index4' -H 'Content-Type: application/json' -d '
{                     
  "settings": {
    "index": {
      "number_of_shards": 3,  
      "number_of_replicas": 1
   }                         
 }
}'
# ES自动将分片数据均匀的放在不同节点上，实现高可用
```

![image-20250314093940483](pic/image-20250314093940483.png)

```shell
curl -XDELETE http://127.0.0.1:9200/index2

# 删除所有索引
vim /etc/elasticsearch/elasticsearch.yml
action.destructive_requires_name: false

curl -X DELETE "http://127.0.0.1:9200/*"

# 也可以不开启action.destructive_requires_name，使用循环进行批量删除
for i in `curl 'http://127.0.0.1:9200/_cat/indices?v'|awk '{print $3}'`;do curl -XDELETE http://127.0.0.1:9200/$i;done
```

### doc 文档操作

```shell
#创建文档时不指定_id，会自动生成
#8.X版本后因为删除了type,所以索引操作：{index}/{type}/需要修改成{index}/_doc/
#8.X版本之后

curl -XPOST http://127.0.0.1:9200/index0/_doc/ -H 'Content-Type: application/json' -d '{"name":"linux", "author": "ll", "version": "1.0"}'

# 指定id，插入文档（通常是系统分配）
curl -XPOST 'http://127.0.0.1:9200/index0/_doc/3?pretty' -H 'Content-Type: application/json' -d '{"name":"golang", "author": "ll", "version": "1.0"}' 
```

查询文档

```shell
# 所有文档
curl 'http://127.0.0.1:9200/index0/_search?pretty'

# 指定ID
curl 'http://127.0.0.1:9200/index0/_doc/3?pretty'
```

更新文档

```shell
curl -XPOST 'http://127.0.0.1:9200/index0/_doc/3' -H 'Content-Type: application/json' -d '{"version": "2.0","name":"golang","author": "ll"}'
```

删除文档

```shell
curl -XDELETE 'http://127.0.0.1:9200/index1/_doc/3'
```

# ELasticsearch集群工作原理

单机节点 ES 存在单点可用性和性能问题,可以实现Elasticsearch多机的集群解决

Elasticsearch 支持集群模式

-   能够提高Elasticsearch可用性，即使部分节点停止服务，整个集群依然可以正常服务
-   能够增大Elasticsearch的性能和容量，如内存、磁盘，使得Elasticsearch集群可以支持PB级的数据

## 节点

### ES 节点分类

*   **Master节点**
    *   ES集群中只有一个 Master 节点，用于**控制**和**管理**整个集群的操作
    *   Master 节点负责**增删索引**,**增删节点**,**分片shard的重新分配**
    *   Master 主要维护**Cluster State**，包括节点名称,节点连接地址,索引名称和配置信息等
    *   Master 接受集群状态的变化并推送给所有其它节点,集群中各节点都有一份完整的集群状态信息， 都由master node负责维护
    *   Master 节点**不需要**涉及到**文档级别**的变更和搜索等操作
    *   协调创建索引请求或查询请求，将请求分发到相关的node上。
    *   当Cluster State有新数据产生后， Master 会将数据同步给其他 Node 节点
    *   Master节点通过超过一半的节点投票选举产生的
    *   可以设置node.master: true 指定为是否参与Master节点选举, 默认true
*   **Data节点**
    *   存储数据的节点即为 data 节点
    *   当创建索引后，索引的数据会存储至某个数据节点
    *   Data 节点消耗内存和磁盘IO的性能比较大
    *   配置node.data: true, 就是Data节点，默认为 true,**即默认所有节点都是 Data 节点类型**
*   **Ingest 节点**
    *   如果集群中有大量数据预处理需求（如日志解析、字段提取等），可以引入专门的 Ingest 节点。 功能类似logstash
    *   将 Ingest 节点与 Data 节点分离，避免数据预处理影响数据存储和查询性能
    *   负责数据预处理（如管道处理、数据转换等）
    *   Ingest 节点的基础原理是：节点接收到数据之后，根据请求参数中指定的管道流 id，找到对应的已 注册管道流，对数据进行处理，然后将处理过后的数据，按照 Elasticsearch 标准的 indexing 流程 继续运行。

*   **Coordinating 节点(协调)**
    *   处理请求的节点即为 coordinating 节点，该节点类型为所有节点的默认角色，不能取消coordinating 节点。主要将请求路由到正确的节点处理。比如创建索引的请求会由 coordinating 路由到 master 节点处理
    *   这可以减轻 Data 节点的负载，提高查询性能。
    *   当配置 node.master:false、node.data:false 则只充当 Coordinating 节点
    *   Coordinating 节点在 Cerebro 等插件中数据页面不会显示出来
*   **Machine Learning 节点**
    *   负责机器学习任务（如异常检测等）。
    *   如果使用 Elasticsearch 的机器学习功能，可以配置专门的 Machine Learning 节点。
    *   这些节点需要较高的 CPU 和内存资源。
    *   需要 enable x-pack

*   **Master-eligible 初始化时有资格选举Master的节点**
    *   集群初始化时有权利参于选举Master角色的节点
    *   只在集群第一次初始化时进行设置有效，后续配置无效
    *   由 cluster.initial_master_nodes 配置节点地址

### 节点规划

单一职责的节点: 一个节点只承担一个角色

![image-20250315194644465](pic/image-20250315194644465.png)

*   Dedicated master nodes：负责集群状态（cluster state）的管理
    *   从高可用 & 避免脑裂的角色出发,一般在生产环境中配置 3 台,一个集群只有 1 台活跃的主节点 使用低配置的 CPU ,RAM 和磁盘
*   Dedicated data nodes: 负责数据存储及处理客户端请求
    *   使用高配置的 CPU,RAM 和磁盘
*   Dedicated ingest nodes: 负责数据处理
    *   使用高配置的 CPU ; 中等配置的 RAM; 低配置的磁盘
*   Dedicate Coordinating Only Node (Client Node)
    *   配置：将 Master ，Data ，Ingest 都配置成 Flase
    *   生产环境中，建议为一些大的集群配置 Coordinating Only Nodes,扮演 Load Balancers。 降低  Master 和 Data Nodes 的负载
    *   负载搜索结果的 Gather / Reduce有时候无法预知客户端会发生怎样的请求大量占用内存的结合操 作，一个深度聚合可能引发 OOM

### 节点架构

当磁盘容量无法满足需求时，可以增加数据节点；磁盘读写压力大时，增加数据节点

![image-20250315195002428](pic/image-20250315195002428.png)

当系统中有大量的复杂查询及聚合时候，增加 Coordinating 节点，增加查询的性能

![image-20250315195014846](pic/image-20250315195014846.png)

读写分离

![image-20250315195024200](pic/image-20250315195024200.png)

## ES集群选举

选举时,优先选举**ClusterStateVersion**最大的Node节点，如果ClusterStateVersion相同，则选举**Node ID**最小的Node

ClusterStateVersion 是集群状态的版本号，每当集群状态发生变更时（例如节点加入、索引创建、分片重新分配等），版本号都会递增。因此，这个版本号越大，说明是越早加入集群的，里面的数据越多

**Node的ID**是在第一次服务启动时随机生成的，直接选用最小ID的Node，主要是为了选举的稳定性，尽量少出现选举不出来的问题。

```shell
curl -XGET "http://10.0.0.222:9200/_cluster/state/version?pretty"
{
  "cluster_name" : "my-application",
  "cluster_uuid" : "o5dy5KnXTLqYi5nxANWaZw",
  "version" : 222, # ClusterStateVersion
  "state_uuid" : "_U7QTJ_PSwezxZyl2NWMdg"
}

curl -XGET "http://10.0.0.222:9200/_cat/nodes?v&h=ip,name,id"
ip         name   id
10.0.0.223 node-2 vaWX
10.0.0.224 node-3 8r4D
10.0.0.222 node-1 wkhV
```



## ES 集群 Shard 和 Replication

### 分片 Shard

ES 中存储的数据可能会很大,有时会达到PB级别，单节点的容量和性能可以无法满足

基于容量和性能等原因,可以将一个索引数据分割成多个小的分片

再将每个分片分布至不同的节点,从而实现数据的分布存储,实现性能和容量的水平扩展

在读取时,可以实现多节点的并行读取,提升性能

除此之外,如果一个分片的主机宕机,也不影响其它节点分片的读取

横向扩展即增加服务器，当有新的Node节点加入到集群中时，集群会动态的重新进行均匀分配和负载

例如原来有两个Node节点，每个节点上有3个分片，即共6个分片,如果再添加一个node节点到集群中，
集群会动态的将此6个分片分配到这三个节点上，最终每个节点上有2个分片。

### 副本 Replication

将一个索引分成多个数据分片,仍然存在数据的单点问题,可以对每一个分片进行复制生成副本,即备份,实现数据的高可用

ES的分片分为主分片（primary shard）和副本分片（复制replica shard），而且通常分布在不同节点

**主分片实现数据读写,副本分片只支持读**

在索引中的每个分片只有一个主分片,而对应的副本分片可以有多个,一个副本本质上就是一个主分片的备份

每个分片的主分片在创建索引时自动指定且后续不能人为更改

**默认分片配置**

默认情况下，elasticsearch将分片相关的配置从配置文件中的属性移除了，可以借助于一个默认的模板接口将索引的分片属性更改成我们想要的分片效果

```shell
curl -XPUT 'http://127.0.0.1:9200/_template/template_http_request_record' -H 'Content-Type: application/json' -d '{"index_patterns": ["*"],"settings": {"number_of_shards": 5,"number_of_replicas": 1}}'

#属性解析：
接口地址：_template/template_http_request_record
索引类型：index_patterns
分片数量：number_of_shards
副本数量：number_of_replicas
```

## 数据同步机制

Elasticsearch主要依赖 **Zen Discovery 协议**来管理集群中节点的加入和离开，以及选举主节点（master node）。

Zen Discovery是Elasticsearch自带的一个协议，不依赖于任何外部服务。

然而，Elasticsearch对于一致性的处理与传统的一致性协议（如Raft或Paxos）有所不同。它采取了一 种“**最终一致性**”（eventual consistency）的模型。

每个索引在Elasticsearch中被分成多个分片（shard），每个分片都有一个主分片和零个或多个副本分片。

主分片负责处理所有的写操作，并将写操作复制到其副本分片。当主分片失败时，一个副本分片会被提升为新的主分片

Elasticsearch为了提高写操作的性能，允许在主分片写入数据后立即确认写操作，而不需要等待数据被所有副本分片确认写入。这就意味着，在某些情况下，主分片可能会确认写操作成功，而实际上副本分片还没有完全写入数据。这就可能导致数据在短时间内在主分片和副本分片之间不一致。然而，一旦所 有副本分片都确认写入了数据，那么系统就会达到“最终一致性”。

为了保证搜索的准确性，Elasticsearch还引入了一个**"refresh"机制**，每隔一定时间（默认为1秒）将最新的数据加载到内存中，使其可以被搜索到。这个过程是在主分片和所有副本分片上独立进行的，所以可能存在在短时间内搜索结果在不同分片之间有些许不一致的情况，但随着时间的推移，所有分片上的数据都会达到一致状态。

## 集群故障转移

故障转移指的是，当集群中有节点发生故障时，ES集群会进行自动修复

![image-20250314152301829](pic/image-20250314152301829.png)

**ES集群的故障转移流程如下**

-   重新选举
    -   假设当前Master节点 node3 节点宕机,同时也导致 node3 的原有的P1和R2分片丢失
    -   node1 和 node2 发现 Master节点 node3 无法响应
    -   过一段时间后会重新发起 master 选举
    -   比如这次选择 node2 为 新 master 节点；此时集群状态变为yellow 状态
    -   其实无论选举出的新Master节点是哪个节点，都不影响后续的分片的重新分布结果

![image-20250121175830942](pic/image-20250121175830942.png)

-   主分片调整
    -   新的Master节点 node2 发现在原来在node3上的主分片 P1 丢失
    -   将 node1 上的 R1 提升为主分片
    -   此时所有的主分片都正常分配，但1和2分片没有副本分片
    -   集群状态变为 Yellow状态
-   副本分片调整
    -   node1 将 P1 和 node2上的P2 主分片重新生成新的副本分片 R1、R2，此时集群状态变为 Green

-   后续修复好node3节点后，Master 不会重新选举，但会自动将各个分片重新均匀分配
    -   保证主分片尽可能分布在每个节点上
    -   副本分片也尽可能分布不同的节点上
    -   重新分配的过程需要一段时间才能完成

![image-20250121180438208](pic/image-20250121180438208.png)

## ES文档路由

ES文档是分布式存储，当在ES集群访问或存储一个文档时，由下面的算法决定此**文档到底存放在哪个主分片中**,再结合**集群状态**找到存放此主分片的节点主机

```shell
shard = hash(routing) % number_of_primary_shards
hash                     #哈希算法可以保证将数据均匀分散在分片中
routing                  #用于指定用于hash计算的一个可变参数，默认是文档id，也可以自定义
number_of_primary_shards #主分片数
# 注意：该算法与主分片数相关，一旦确定后便不能更改主分片，因为主分片数的变化会导致所有分片需要重新分配
# 先根据哈希/主分片数计算出要存放到几号分片中，存放到主分片后，再将数据复制到副本分片中
```

**ES 文档创建删除流程**

![image-20250121181541492](pic/image-20250121181541492.png)

-   客户端向集群中某个节点 Node1 发送**新建索引文档**或者**删除索引文档**请求
-   Node1节点使用文档的 _id 通过上面的算法确定文档属于分片 0
-   因为分片 0 的主分片目前被分配在 Node3 上,请求会被转发到 Node3
-   Node3 在主分片上面执行创建或删除请求
-   Node3 执行如果成功，它将请求并行转发到 Node1 和 Node2 的副本分片上
-   Node3 将向协调节点Node1 报告成功
-   协调节点Node1 客户端报告成功。

**ES 文档读取流程**

![image-20250121181827402](pic/image-20250121181827402.png)

-   客户端向集群中某个节点 Node1 发送读取请求
-   节点使用文档的 _id 来确定文档属于分片 0 。分片 0 的主副本分片存在于所有的三个节点上
-   在处理读取请求时，协调节点在每次请求的时候都会通过轮询所有的主副本分片来达到负载均衡， 此次它将请求转发到 Node2
-   Node2 将文档返回给 Node1 ，然后将文档返回给客户端

## 生产故障

故障说明:负载不高，但是有的索引会查询不出来,查看到下面的日志信息

![image-20250316105812009](pic/image-20250316105812009.png)在Elasticsearch中，search.max_open_scroll_context 参数用于限制集群中可以同时打 开的 scroll 上下文的数量。这个参数的默认值在不同版本的 Elasticsearch 中可能会有所不同，但根 据搜索结果显示，在某些版本的 Elasticsearch 中，默认值可能是 500  或 2000 。这个默认值是为了 防止过多的 scroll 上下文同时打开导致资源耗尽。如果需要处理大量的数据，可能需要根据实际情况调整 这个参数的值

```shell
curl -X PUT localhost:9200/_cluster/settings -H 'Content-Type: application/json' -d'{"persistent" : {"search.max_open_scroll_context": 8000}, "transient": {"search.max_open_scroll_context": 8000}}'

curl -X GET localhost:9200/_cluster/settings
```



# Beats收集数据

Beats 是一个免费且开放的平台，集合了多种单一用途数据采集器。它们从成百上千或成千上万台机器 和系统向 Logstash 或 Elasticsearch 发送数据。 

虽然利用 logstash 就可以收集日志，功能强大，但由于 Logstash 是基于Java实现，需要在采集日志的 主机上安装JAVA环境

ogstash运行时最少也会需要额外的**500M的以上的内存**，会消耗比较多的内存和磁盘空间， 可以采有基于Go开发的 Beat 工具代替 Logstash 收集日志，部署更为方便，而且只占用**10M左右的内 存空间**及更小的磁盘空间。

[Beats: Data Shippers for Elasticsearch | Elastic](https://www.elastic.co/beats)

[elastic/beats: :tropical_fish: Beats - Lightweight shippers for Elasticsearch & Logstash](https://github.com/elastic/beats)

 Beats 是一些工具集,包括以下,其中 filebeat 应用最为广泛

![image-20250315203325197](pic/image-20250315203325197.png)

```shell
filebeat:收集日志文件数据。最常用的工具
packetbeat:用于收集网络数据。一般用zabbix实现此功能
metricbeat:从OS和服务收集指标数据，比如系统运行状态、CPU 内存利用率等。
winlogbeat: 从Windows平台日志收集工具。
heartbeat: 定时探测服务是否可用。支持ICMP、TCP 和 HTTP，也支持TLS、身份验证和代理
auditbeat:收集审计日志
Functionbeat:使用无服务器基础架构提供云数据。面向云端数据的无服务器采集器，处理云数据
```

| Beat                                                         | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Auditbeat](https://github.com/elastic/beats/tree/main/auditbeat) | Collect your Linux audit framework data and monitor the integrity of your files. |
| [Filebeat](https://github.com/elastic/beats/tree/main/filebeat) | Tails and ships log files                                    |
| [Heartbeat](https://github.com/elastic/beats/tree/main/heartbeat) | Ping remote services for availability                        |
| [Metricbeat](https://github.com/elastic/beats/tree/main/metricbeat) | Fetches sets of metrics from the operating system and services |
| [Packetbeat](https://github.com/elastic/beats/tree/main/packetbeat) | Monitors the network and applications by sniffing packets    |
| [Winlogbeat](https://github.com/elastic/beats/tree/main/winlogbeat) | Fetches and ships Windows Event logs                         |
| [Osquerybeat](https://github.com/elastic/beats/tree/main/x-pack/osquerybeat) | Runs Osquery and manages interraction with it.               |

**Beats 版本要和 Elasticsearch 相同的版本，否则可能会出错**

## 利用 Metricbeat 监控性能相关指标

Metricbeat 可以收集指标数据，比如系统运行状态、CPU、内存利用率等。 

生产中一般用 zabbix 等专门的监控系统实现此功能

[Metricbeat quick start: installation and configuration | Metricbeat Reference [8.17\] | Elastic](https://www.elastic.co/guide/en/beats/metricbeat/current/metricbeat-installation-configuration.html)

```shell
curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-8.17.3-amd64.deb
sudo dpkg -i metricbeat-8.17.3-amd64.deb
```

配置

```shell
vim /etc/metricbeat/metricbeat.yml
setup.kibana:
  host: "10.0.0.220:5601" 
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["10.0.0.222:9200","10.0.0.223:9200","10.0.0.224:9200"]

systemctl start metricbeat.service
```

![image-20250316111413808](pic/image-20250316111413808.png)

通过 Kibana 查看收集的性能指标

Observability --- 基础设施- 基础设施库存

## 利用 Heartbeat 监控

[Heartbeat quick start: installation and configuration | Heartbeat Reference [8.17\] | Elastic](https://www.elastic.co/guide/en/beats/heartbeat/current/heartbeat-installation-configuration.html)

```shell
wget https://artifacts.elastic.co/downloads/beats/heartbeat/heartbeat-8.17.3-amd64.deb
sudo dpkg -i heartbeat-8.17.3-amd64.deb
```

官方配置[Configure Heartbeat monitors | Heartbeat Reference [8.17\] | Elastic](https://www.elastic.co/guide/en/beats/heartbeat/current/configuration-heartbeat-options.html)

```shell
# heartbeat.yml
heartbeat.monitors:
- type: icmp
  id: ping-myhost
  name: My Host Ping
  hosts: ["myhost"]
  schedule: '*/5 * * * * * *'
- type: tcp
  id: myhost-tcp-echo
  name: My Host TCP Echo
  hosts: ["myhost:777"]  # default TCP Echo Protocol
  check.send: "Check"
  check.receive: "Check"
  schedule: '@every 5s'
- type: http
  id: service-status
  name: Service Status
  service.name: my-apm-service-name
  hosts: ["http://localhost:80/service/status"]
  check.response.status: [200]
  schedule: '@every 5s'
heartbeat.scheduler:
  limit: 10
```

```shell
Field name     Mandatory?   Allowed values    Allowed special characters
----------     ----------   --------------    --------------------------
Seconds        No           0-59              * / , -
Minutes        Yes          0-59              * / , -
Hours          Yes          0-23              * / , -
Day of month   Yes          1-31              * / , - L W
Month          Yes          1-12 or JAN-DEC   * / , -
Day of week    Yes          0-6 or SUN-SAT    * / , - L #
Year           No           1970–2099         * / , -
```

配置

```shell
vim /etc/heartbeat/heartbeat.yml
setup.kibana:
  host: "10.0.0.220:5601"
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["10.0.0.222:9200","10.0.0.223:9200","10.0.0.224:9200"]
  
# 监控项
heartbeat.monitors:
- type: http
  enabled: true # 启动
  id: my-monitor
  name: My Monitor
  urls: ["http://localhost:9000"]
  schedule: '@every 10s'
  # Total test connection and data exchange timeout
  #timeout: 16s
  # Name of corresponding APM service, if Elastic APM is in use for the monitored service.
  #service.name: my-apm-service-name   
  
setup.template.settings:
  index.number_of_shards: 3         # 修改分片数量，因为集群有3个节点，因此改为3
  index.codec: best_compression
```

Observability---运行时间--监测 Uptime Monitors

![image-20250316115627597](pic/image-20250316115627597.png)

利用 Kibana 将 Heartbeat 的数据进行可视化

## 利用 Filebeat 收集日志

Filebeat 是用于转发和集中日志数据的轻量级传送程序。作为服务器上的代理安装，Filebeat监视您指定 的日志文件或位置，收集日志事件，并将它们转发到Elasticsearch或Logstash进行索引。

filebeat 支持从日志文件,Syslog,Redis,Docker,TCP,UDP,标准输入等读取数据,对数据做简单处理，再输 出至Elasticsearch,logstash,Redis,Kafka等

Filebeat的工作方式如下： 

*   启动Filebeat时，它将启动一个或多个输入源，这些输入将在为日志数据指定的位置中查找。
*    对于Filebeat所找到的每个日志，Filebeat都会启动收集器harvester进程。 
*   每个收集器harvester都读取一个日志以获取新内容，并将新日志数据发送到libbeat 
*   libbeat会汇总事件并将汇总的数据发送到为Filebeat配置的输出。

![image-20250316180223271](pic/image-20250316180223271.png)

注意: Filebeat 支持多个输入,但不支持同时有多个输出，如果多输出，会报错如下

```shell
Exiting: error unpacking config data: more than one namespace configured 
accessing 'output' (source:'/etc/filebeat/stdout_file.yml')
```

### 安装

```shell
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.17.3-amd64.deb
sudo dpkg -i filebeat-8.17.3-amd64.deb
```

### Filebeat 配置

```shell
vim /etc/filebeat/filebeat.yml
setup.kibana:
  host: "10.0.0.220:5601"
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["10.0.0.222:9200","10.0.0.223:9200","10.0.0.224:9200"]

#语法检查
filebeat test config  -c  /etc/filebeat/filebeat.yml
```

### 从STDIN读取再输出STDOUT

```shell
vim ~/stdin.yaml
filebeat.inputs:
- type: stdin
  enabled: true
  tags: ["stdin-tags","myapp"]  #添加新字段名tags，可以用于判断不同类型的输入，实现不同的输出，日志做分类
  fields:
    status_code: "200"      #添加新字段名fields.status_code，可以用于判断不同类型的输入，实现不同的输出，日志做分类
    author: "ll"

output.console:
  pretty: true
  enable: true
  
filebeat test config -c /root/stdin.yaml 

#从指定文件中读取配置
#-e 表示Log to stderr and disable syslog/file output
# 解析文本，不能解析json格式文本
filebeat -e -c /root/stdin.yaml
# 在屏幕输入hello


# 解析Json格式文本
filebeat.inputs:
- type: stdin
  enable: true
  json.keys_under_root: true  # 解析json

output.console:
  pretty: true
  enable: true
  
# 输入{"name" : "mystical", "age" : "18", "phone" : "0123456789"}
```

### 从STDIN读取再输出FILE

```shell
vim ~/stdin2file.yaml
filebeat.inputs:
- type: stdin
  enable: true
  json.keys_under_root: true

output.file:
  path: "/tmp"
  filename: "filebeat.log"
  
filebeat -c /root/stdin2file.yaml
{"name" : "ll", "age" : "18", "phone" : "0123456789"}

cat /tmp/filebeat.log-20250316.ndjson | jq
{
  "@timestamp": "2025-03-16T11:41:06.299Z",
  "@metadata": {
    "beat": "filebeat",
    "type": "_doc",
    "version": "8.17.3"
  },
  "log": {
    "offset": 0,
    "file": {
      "path": ""
    }
  },
  "name": "mystical",
  "input": {
    "type": "stdin"
  },
  "agent": {
    "id": "f2138a29-4b67-4053-bbf5-267bcc60cad4",
    "name": "ubuntu-2204",
    "type": "filebeat",
    "version": "8.17.3",
    "ephemeral_id": "e3e6ca1f-ef15-446e-95c4-527eeaca8845"
  },
  "ecs": {
    "version": "8.0.0"
  },
  "host": {
    "name": "ubuntu-2204"
  },
  "age": "18",
  "phone": "0123456789"
}
```

### 从FILE读取再输出至STDOUT

filebeat 会将每个文件的读取数据的相关信息记录在`log.json`文件中,可以实现日志采集的持续性,而不会重复采集

-   当日志文件大小发生变化时，filebeat会接着上一次记录的位置继续向下读取新的内容
-   当日志文件大小没有变化，但是内容发生变化，filebeat会将文件的全部内容重新读取一遍

```yaml
vim /root/file2stdout.yaml 
filebeat.inputs:
- type: log
  enable: true
  json.keys_under_root: true
  paths:
  #- /var/log/syslog
  - /var/log/test.log

output.console:
  pretty: true
  enable: true

filebeat -c /root/file2stdout.yaml 

echo '{"name" : "ll", "age" : "20", "phone" : "0123456789"}' >> /var/log/test.log
```

不同的文件不同的tag与fields

```shell
filebeat.inputs:
- type: log
  paths:
  - /var/log/app1
  json.keys_under_root: true
  tags: ["app1_tag"]
  fields:
    type: "app1"

- type: log
  paths:
  - /var/log/app2
  json.keys_under_root: true
  tags: ["app2_tag"]
  field:
    type: "app2"
    
output.console:
  pretty: true
  
filebeat -c /root/file2stdout.yaml 

echo '{"name" : "llk", "age" : "20", "phone" : "0123456789"}' >> /var/log/app1
echo '{"name" : "ll", "age" : "20", "phone" : "0123456789"}' >> /var/log/app2
```

### 利用 Filebeat 收集系统日志到 ELasticsearch

```shell
vim /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  json.keys_under_root: true
  enabled: true                   # 开启日志
  paths:
  - /var/log/syslog                    # 指定收集的日志

output.elasticsearch:
  hosts: ["10.0.0.222:9200", "10.0.0.223:9200", "10.0.0.224:9200"] # 指定ELK集群任意节点地址和端口，多个端口容错
```

### 自定义索引名称收集日志到 ELasticsearch

按天存索引，方便清理日志

```shell
filebeat.inputs:
- type: log
  json.keys_under_root: true
  enabled: true                       #开启日志 
  paths:
  - /var/log/syslog                   #指定收集的日志文件
  #include_lines: ['sshd','failed', 'password']  #只过滤指定包含关健字的日志
  #include_lines: ['^ERR', '^WARN']   #只过滤指定包含关健字的日志
  #exclude_lines: ['Debug']           #排除包含关健字的日志
  #exclude_files: ['.gz$']            #排除文件名包含关健字的日志文件
  
output.elasticsearch:
  hosts: ["10.0.0.222:9200"]        #指定ES集群服务器地址和端口
  index: "ll-%{[agent.version]}-%{+yyyy.MM.dd}"
  #注意: 8.X版生成的索引名为 .ds-wang-%{[agent.version]}-日期-日期-00000n
  #注意：如果自定义索引名称，没有添加下面三行的配置会导致filebeat无法启动
  
setup.ilm.enabled: false  #关闭索引生命周期ilm功能，默认开启时索引名称只能为filebeat-*，自定义索引名必须修改为false
setup.template.name: "ll" #定义模板名称,要自定义索引名称,必须指定此项,否则无法启动
setup.template.pattern: "ll-*" #定义模板的匹配索引名称,要自定义索引名称,必须指定此项,否则无法启动

setup.template.settings:
  index.number_of_shards: 3
  index.number_of_replicas: 1
```

### 收集 Nginx

#### 利用 tags 收集 Nginx的 Json 格式访问日志和错误日志 到 Elasticsearch 不同的索引

```shell
apt update && apt -y install nginx

vim /etc/nginx/nginx.conf
http{
			    log_format access_json '{"@timestamp":"$time_iso8601",'
			    '"host":"$server_addr",'
			    '"clientip":"$remote_addr",'
			    '"size":$body_bytes_sent,'
			    '"responsetime":$request_time,'
			    '"upstreamtime":"$upstream_response_time",'
			    '"upstreamhost":"$upstream_addr",'
			    '"http_host":"$host",'
			    '"uri":"$uri",'
			    '"domain":"$host",'
			    '"xff":"$http_x_forwarded_for",'
			    '"referer":"$http_referer",'
			    '"tcp_xff":"$proxy_protocol_addr",'
			    '"http_user_agent":"$http_user_agent",'
			    '"status":"$status"}';
			    
			    access_log  /var/log/nginx/access_json.log access_json ;
    #access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    
    location / {
        #try_files $uri $uri/ =404;       # 注释掉                                                 
    }
 
}
```

Filebeat 配置文件

```shell
vim /etc/filebeat/filebeat.yml
filebeat.inputs:
- type: log
  json.keys_under_root: true
  enabled: true                       
  paths:
  - /var/log/nginx/access_json.log                   
  json.keys_under_root: true
  json.overwrite_keys: true  #设为true,使用json格式日志中自定义的key替代默认的message字段,此项可选
  tags: ["nginx-access"]     #指定tag,用于分类
  
- type: log
  enabled: true
  paths:
  - /var/log/nginx/error.log
  tags: ["nginx-error"]
  
output.elasticsearch:
  hosts: ["10.0.0.222:9200"]        #指定ES集群服务器地址和端口
  indices:
    - index: "nginx-access-%{[agent.version]}-%{+yyy.MM.dd}" 
      when.contains:
        tags: "nginx-access" 
    - index: "nginx-error-%{[agent.version]}-%{+yyy.MM.dd}"
      when.contains:
        tags: "nginx-error"
  
setup.ilm.enabled: false  
setup.template.name: "nginx" 
setup.template.pattern: "nginx-*" 

setup.template.settings:
  index.number_of_shards: 3
  index.number_of_replicas: 1
```

#### 利用 fields 收集 Nginx的同一个访问日志中不同响应码 的行到 Elasticsearch 不同的索引

```shell
filebeat.inputs:
- type: log
  enabled: true
  paths:
  - /var/log/nginx/access.log
  include_lines: ['404']  #文件中包含404的行
  fields:
    status_code: "404"

- type: log
  enabled: true
  paths:
  - /var/log/nginx/access.log
  include_lines: ['200']    #文件中包含404的行
  fields:
    status_code: "200"

- type: log
  enabled: true
  paths:
  - /var/log/nginx/access.log
  include_lines: ['304']
  fields:
    status_code: "304"
    
output.elasticsearch:
  hosts: ["http://10.0.0.201:9200"]
  indices:
    - index: "myapp-error-404-%{+yyyy.MM.dd}"
      when.equals:
        fields.status_code: "404"
    - index: "myapp-ok-200-%{+yyyy.MM.dd}"
      when.equals:
        fields.status_code: "200"
    - index: "myapp-red-304-%{+yyyy.MM.dd}"
      when.equals:
        fields.status_code: "304"
        
setup.ilm.enabled: false
setup.template:
  enabled: true  #默认值，可选
  name: "myapp"
  pattern: "myapp-*"
```

### 收集Tomat

#### 利用 Filebeat 收集 Tomat 的 Json 格式的访问日志和错 误日志到 Elasticsearch

```shell
apt update && apt -y install tomcat9 tomcat9-admin

#修改 Tomcat 的访问日志为Json格式
vim /etc/tomcat9/server.xml

<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" prefix="localhost_access_log" suffix=".txt" pattern="{&quot;clientip&quot;:&quot;%h&quot;,&quot;ClientUser&quot;:&quot;%l&quot;,&quot;authenticated&quot;:&quot;%u&quot;,&quot;AccessTime&quot;:&quot;%t&quot;,&quot;method&quot;:&quot;%r&quot;,&quot;status&quot;:&quot;%s&quot;,&quot;SendBytes&quot;:&quot;%b&quot;,&quot;Query?string&quot;:&quot;%q&quot;,&quot;partner&quot;:&quot;%{Referer}i&quot;,&quot;AgentVersion&quot;:&quot;%{User-Agent}i&quot;}"/>
```

Filebeat 配置文件

```shell
filebeat.inputs:
- type: log
  enabled: true
  paths:
  - /var/log/tomcat9/localhost_access_log.*
  json.keys_under_root: true  
  json.overwrite_keys: false 
  tags: ["tomcat-access"]
  
- type: log
  enabled: true
  paths:
  - /var/log/tomcat9/catalina.*.log       
  tags: ["tomcat-error"]

output.elasticsearch:
  hosts: ["10.0.0.101:9200"]        
  indices:
    - index: "tomcat-access-%{[agent.version]}-%{+yyyy.MM.dd}" 
      when.contains:
        tags: "tomcat-access"
    - index: "tomcat-error-%{[agent.version]}-%{+yyyy.MM.dd}"
      when.contains:
        tags: "tomcat-error"
        
setup.ilm.enabled: false 
setup.template.name: "tomcat" 
setup.template.pattern: "tomcat-*"
setup.template.settings:
  index.number_of_shards: 3
  index.number_of_replicas: 1
```

#### 利用 Filebeat 收集 Tomat 的多行错误日志到 Elasticsearch

Tomcat 是 Java 应用,当只出现一个错误时,会显示很多行的错误日志,如下所示

Java 应用的一个错误导致生成的多行日志其实是同一个事件的日志的内容 

而ES默认是根据每一行来区别不同的日志,就会导致一个错误对应多行错误信息会生成很多行的ES文档记 录 

可以将一个错误对应的多个行合并成一个ES的文档记录来解决此问题

[Manage multiline messages | Filebeat Reference [8.17\] | Elastic](https://www.elastic.co/guide/en/beats/filebeat/current/multiline-examples.html)

```shell
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/tomcat9/catalina.*.log #包安装
  tags: ["tomcat-error"]
  multiline.type: pattern            #此为默认值,可省略
  multiline.pattern: '^[0-3][0-9]-'  #正则表达式匹配以两位,或者为'^\d{2}'
  multiline.negate: true             #negate否定无效
  multiline.match: after
  multiline.max_lines: 5000          #默认只合并500行,指定最大合并5000行
#格式2
filebeat.inputs:
- type: filestream
  parsers:
  - multiline:
      type: pattern
      pattern: '^[0-3][0-9]-'
      negate: true
      match: after
```

### Filebeat输出至 Redis

```shell
output.redis:
  hosts: ["10.0.0.105:6379"]
  password: "123456"
  db: "0"
  key: "filebeat" #所有日志都存放在key名称为filebeat的列表中，llen filebeat可查看长度，即日志记录数
 #keys: #也可以用下面的不同日志存放在不同的key的形式
 #  - key: "nginx_access"
 #    when.contains:
 #    tags: "access"
 #  - key: "nginx_error"
 #    when.contains:
 #    tags: "error"
```

redis

```shell
apt -y install redis
sed -i.bak '/^bind.*/c bind 0.0.0.0'  /etc/redis/redis.conf
systemctl restart  redis

redis-cli 
127.0.0.1:6379> keys *
1) "filebeat"
127.0.0.1:6379> type filebeat
list
127.0.0.1:6379> llen filebeat
(integer) 2097
127.0.0.1:6379> LINDEX filebeat 2096
```

### Filebeat输出至 Kafka

```shell
output.kafka:
  hosts: ["10.0.0.201:9092", "10.0.0.202:9092", "10.0.0.203:9092"]
  topic: filebeat-log       #指定kafka的topic
  partition.round_robin:
    #true表示只发布到可用的分区，不可用分区就不发布，false 时表示所有分区，如果一个节点down，会block
    reachable_only: true    
  #如果为0，不启用确认机制，错误消息可能会丢失，1等待写入主分区（默认），-1等待写入副本分区
  required_acks: 1          
  compression: gzip  
  max_message_bytes: 1000000 #每条消息最大长度，以字节为单位，如果超过将丢弃

output.kafka:
  hosts: ["localhost:9092"]
  topic: "logs-%{[agent.version]}"
  topics:
    - topic: "critical-%{[agent.version]}"
      when.contains:
        message: "CRITICAL"
    - topic: "error-%{[agent.version]}"
      when.contains:
        message: "ERR"
```

### Filebeat输出至 Logstash

[Configure the Logstash output | Filebeat Reference [8.17\] | Elastic](https://www.elastic.co/guide/en/beats/filebeat/current/logstash-output.html)

```shell
output.logstash:
  hosts: ["10.0.0.104:5044","10.0.0.105:5044"]
  index: filebeat  
  loadbalance: true     #默认为false,只随机输出至一个可用的logstash,设为true,则输出至全部logstash
  worker: 1             #线程数量        
  compression_level: 3  #压缩比
```

# Logstash 过滤

![image-20250316185029407](pic/image-20250316185029407.png)

Logstash 是免费且开放的服务器端数据处理管道，能够从多个来源采集数据，转换数据，然后将数据发 送到您最喜欢的一个或多个“存储库”中

Logstash 是整个ELK中拥有最丰富插件的一个组件,而且支持可以水平伸缩 

Logstash 基于 Java 和 Ruby 语言开发

[Logstash: Collect, Parse, Transform Logs | Elastic](https://www.elastic.co/logstash)

*   输入 Input：用于日志收集,常见插件: Stdin、File、Kafka、Redis、Filebeat、Http 
*   过滤 Filter：日志过滤和转换,常用插件: grok、date、geoip、mutate、useragent 
*   输出 Output：将过滤转换过的日志输出, 常见插件: File,Stdout,Elasticsearch,MySQL,Redis,Kafka

 Logstash 和 Filebeat 比

*   Logstash 功能更丰富,可以支持直接将非Josn 格式的日志统一转换为Json格式,且支持多目标输出, 和filebeat相比有更为强大的过滤转换功能 
*   Logstash 资源消耗更多,不适合在每个需要收集日志的主机上安装

## 安装

8.X版本之后的logstash包已经内置了JDK无需安装

```shell
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install logstash
```

或

```shell
wget https://artifacts.elastic.co/downloads/logstash/logstash-8.17.3-amd64.deb
dpkg -i logstash-8.17.3-amd64.deb
```

##  Logstash 配置

```shell
vim /etc/logstash/logstash.yml
node.name: logstash-node01
pipeline.workers: 2
pipeline.batch.size: 1000    #批量从IPNPUT读取的消息个数，可以根据ES的性能做性能优化
pipeline.batch.delay: 5      #处理下一个事件前的最长等待时长，以毫秒ms为单位,可以根据ES的性能做性能优化
path.data: /var/lib/logstash #默认值
path.logs: /var/log/logstash #默认值

#内存优化
vim /etc/logstash/jvm.options
-Xms1g
-Xmx1g

#Logstash默认以logstash用户运行,如果logstash需要收集本机的日志,可能会有权限问题,可以修改为root
vim /lib/systemd/system/logstash.service
[Service]
User=root
Group=root

# 主pipeline
cat /etc/logstash/pipelines.yml
- pipeline.id: main
  path.config: "/etc/logstash/conf.d/*.conf"
  
# /etc/logstash/conf.d/目录下存放子pipeline
```

## Logstash 使用

### Logstash 命令

```shell
https://www.elastic.co/guide/en/logstash/current/first-event.html
#各种插件
https://www.elastic.co/guide/en/logstash/current/input-plugins.html
https://www.elastic.co/guide/en/logstash/current/filter-plugins.html
https://www.elastic.co/guide/en/logstash/current/output-plugins.html
https://www.elastic.co/guide/en/logstash/7.6/input-plugins.html
https://www.elastic.co/guide/en/logstash/7.6/filter-plugins.html
https://www.elastic.co/guide/en/logstash/7.6/output-plugins.html
```

[Logstash Reference [8.17\] | Elastic](https://www.elastic.co/guide/en/logstash/current/index.html)

```shell
/usr/share/logstash/bin/logstash 
#常用选项
-e # 指定配置内容
-f # 指定配置文件，支持绝对路径，如果用相对路径，是相对于/usr/share/logstash/的路径
-t # 语法检查
-r # 修改配置文件后自动加载生效,注意:有时候修改配置还需要重新启动生效

# 列出所有插件
/usr/share/logstash/bin/logstash-plugin list
```

[github Logstash Plugins](https://github.com/logstash-plugins)

### Logstash 输入 Input 插件

#### 标准输入输出

codec 用于输入数据的编解码器，默认值为plain表示单行字符串，若设置为json，表示按照json方式解 析

```shell
#不支持Json格式输入
/usr/share/logstash/bin/logstash  -e 'input { stdin{} } output { stdout{ codec => rubydebug }}'
# 输入
hello ll
# 输出
{
    "@timestamp" => 2025-03-17T07:17:45.629451437Z,
       "message" => "hello ll",
         "event" => {
        "original" => "hello ll"
    },
          "host" => {
        "hostname" => "ubuntu-2204"
    },
      "@version" => "1"
}
```

```shell
# 支持Json格式输入
/usr/share/logstash/bin/logstash -e 'input { stdin{ codec => json } } output { stdout{ }}'
# 输入
{ "name":"ll","age": "18","gender":"male"}
# 输出
{
        "gender" => "male",
    "@timestamp" => 2025-03-17T07:21:34.248315192Z,
         "event" => {
        "original" => "{ \"name\":\"ll\",\"age\": \"18\",\"gender\":\"male\"}\n"
    },
          "name" => "ll",
          "host" => {
        "hostname" => "ubuntu-2204"
    },
      "@version" => "1",
           "age" => "18"
}
hello ll
[WARN ] 2025-03-17 07:22:02.624 [[main]<stdin] jsonlines - JSON parse error, original data now in message field {:message=>"Unrecognized token 'hello': was expecting (JSON String, Number, Array, Object or token 'null', 'true' or 'false')\n at [Source: REDACTED (`StreamReadFeature.INCLUDE_SOURCE_IN_LOCATION` disabled); line: 1, column: 6]", :exception=>LogStash::Json::ParserError, :data=>"hello ll"}
{
    "@timestamp" => 2025-03-17T07:22:02.626771804Z,
         "event" => {
        "original" => "hello ll\n"
    },
          "host" => {
        "hostname" => "ubuntu-2204"
    },
       "message" => "hello ll",
      "@version" => "1",
          "tags" => [
        [0] "_jsonparsefailure"
    ]
}
```

```shell
#添加tag和type
/usr/share/logstash/bin/logstash  -e 'input { stdin{  tags => "stdin_tag"  type => "stdin_type" codec => json }  } output { stdout{  }}'
# 输入
{ "name":"ll","age": "18","gender":"male"}
# 输出
{
         "event" => {
        "original" => "{ \"name\":\"ll\",\"age\": \"18\",\"gender\":\"male\"}\n"
    },
        "gender" => "male",
    "@timestamp" => 2025-03-17T07:24:50.753221355Z,
          "name" => "ll",
      "@version" => "1",
          "type" => "stdin_type",
          "tags" => [
        [0] "stdin_tag"
    ],
          "host" => {
        "hostname" => "ubuntu-2204"
    },
           "age" => "18"
}
```

配置格式

```shell
vim /etc/logstash/conf.d/stdin2stdout.conf
input {
    stdin {
        type => "stdin_type"  #自定义类型
        tags => "stdin_tag"   #自定义tag
        codec => "json"
    }
}

output{
    stdout {
        codec => "rubydebug" #输出格式,此为默认值,可省略
    }
}

# 语法检查
/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/stdin2stdout.conf -t

#执行logstash,选项-r表示动态加载配
/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/stdin2stdout.conf -r
# 输入
{ "name":"ll","age": "18","gender":"male"}
# 输出
{
    "@timestamp" => 2025-03-17T07:40:57.492464783Z,
      "@version" => "1",
          "tags" => [
        [0] "stdin_tag"
    ],
          "host" => {
        "hostname" => "ubuntu-2204"
    },
          "name" => "ll",
        "gender" => "male",
           "age" => "18",
         "event" => {
        "original" => "{ \"name\":\"ll\",\"age\": \"18\",\"gender\":\"male\"}\n"
    },
          "type" => "stdin_type"
}
```

#### 从文件输入

Logstash 会记录每个文件的读取位置,下次自动从此位置继续向后读取

```shell
vim /etc/logstash/conf.d/file2stdout.conf
input {
    file {
        path => "/tmp/wang.*"
        type => "wanglog"                     # 添加自定义的type字段,可以用于条件判断,和filebeat中tag功能相似
        exclude => "*.txt"                    # 排除不采集数据的文件，使用通配符glob匹配语法
        start_position => "beginning"         # 第一次从头开始读取文件,可以取值为:beginning和end,默认为end,即只从最后尾部读取日志
        stat_interval => "3"                  # 定时检查文件是否更新，默认1s
        codec => json                         # 如果文件是Json格式,需要指定此项才能解析,如果不是Json格式而添加此行也不会影响结果
    }
    
    file {
        path => "/var/log/syslog"
        type => "syslog"
        start_position => "beginning"
        stat_interval => "3"
    }

}

output {
    stdout {
        codec => rubydebug
    }
}

# 语法检查
/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/file2stdout.conf -t

#执行logstash,选项-r表示动态加载配
/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/file2stdout.conf -r
echo '{ "name":"11", "age":"18" }' >> /tmp/wang.log
{
           "log" => {
        "file" => {
            "path" => "/tmp/wang.log"
        }
    },
      "@version" => "1",
           "age" => "18",
          "name" => "11",
          "type" => "wanglog",
         "event" => {
        "original" => "{ \"name\":\"11\", \"age\":\"18\" }"
    },
    "@timestamp" => 2025-03-17T07:51:02.739917383Z,
          "host" => {
        "name" => "ubuntu-2204"
    }
}
```

#### 从 Http 请求买取数据

```shell
vim /etc/logstash/conf.d/http2stdout.conf
input {
    http {
        port => 6666
        codec => json
    }
}

output {
    stdout {
        codec => rubydebug
    }
}

/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/http2stdout.conf

curl -X POST -d "This is a test data" 10.0.0.220:6666
# 输出
{
           "url" => {
          "path" => "/",
        "domain" => "10.0.0.220",
          "port" => 6666
    },
          "tags" => [
        [0] "_jsonparsefailure"
    ],
    "@timestamp" => 2025-03-17T07:57:57.903190431Z,
    "user_agent" => {
        "original" => "curl/7.81.0"
    },
          "host" => {
        "ip" => "10.0.0.220"
    },
          "http" => {
        "request" => {
            "mime_type" => "application/x-www-form-urlencoded",
                 "body" => {
                "bytes" => "19"
            }
        },
         "method" => "POST",
        "version" => "HTTP/1.1"
    },
      "@version" => "1",
       "message" => "This is a test data"
}

curl -X POST -d '{"name":"ll", "age":"18"}' 10.0.0.220:6666
# 输出
{
           "url" => {
          "path" => "/",
        "domain" => "10.0.0.220",
          "port" => 6666
    },
          "host" => {
        "ip" => "10.0.0.220"
    },
    "@timestamp" => 2025-03-17T07:58:49.359525508Z,
    "user_agent" => {
        "original" => "curl/7.81.0"
    },
          "name" => "ll",
      "@version" => "1",
         "event" => {
        "original" => "{\"name\":\"ll\", \"age\":\"18\"}"
    },
          "http" => {
        "request" => {
            "mime_type" => "application/x-www-form-urlencoded",
                 "body" => {
                "bytes" => "25"
            }
        },
         "method" => "POST",
        "version" => "HTTP/1.1"
    },
           "age" => "18"
}
```

http请求调试工具：

*   postman
*   insomnia [Download - Insomnia](https://insomnia.rest/download)

#### 从 Filebeat 读取数据

filebeat配置

```shell
vim /etc/filebeat/filebeat.yml 
output.logstash:
  hosts: ["10.0.0.220:5044"]             #指定Logstash服务器的地址和端口
```

logstash配置

```shell
vim /etc/logstash/conf.d/filebeat2stdout.conf
input {
    beats {
       port => 5044
    }
}
 
output {
    stdout {
        codec => rubydebug
    }
}

/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/filebeat2stdout.conf
```

#### 从 Redis 输入数据

redis中数据被取走后，就没数据了

```shell
input {
    redis {
        host => '10.0.0.104'
        port => "6379"
        password => "123456"
        db => "0"
        data_type => 'list'
        key => "filebeat"
    }
}

output {
    stdout {
        codec => rubydebug
    }
}
```

Filebeat -> Redis -> Logstatsh

#### 从 Kafka 中读取数据

```shell
input {
    kafka {
        bootstrap_servers => "10.0.0.201:9092,10.0.0.202:9092,10.0.0.203:9092"
        group_id => "logstash" #多个logstash的group_id如果不同，将实现消息共享（发布者/订阅者模式），如果相同（建议使用），则消息独占（生产者/消费者模式）
        topics => ["nginx-accesslog","nginx-errorlog"]
        codec => "json"
        consumer_threads => 8
    }
}

output {
    stdout {
        codec => rubydebug
    }
}
```

Filebeat -> Kafka -> Logstash

#### 从 syslog 输入数据

```shell
input {
    syslog {
        host => "0.0.0.0"
        port => "514"       # 指定监听的UDP/TCP端口，注意普通用户无法监听特权端口（0-1024）
        type => "haproxy"
    }
}

output {
    stdout {
        codec => rubydebug
    }
}

logger -p local6.info -n  <logstash地址>  "This is a test log"
```

####  从TCP/UDP 输入数据

```
input {
    tcp {
        port => "9527"
        host => "0.0.0.0"
        type => "tcplog"
        codec => json
        mode => "server"
    }
}

output {
    stdout {
        codec => rubydebug
    }
}

nc 10.0.0.106 9527
hello, test
```

### Logstash 过滤 Filter 插件

常见的 Filter 插件:

-   利用 **Grok** 从非结构化数据中转化为结构数据
-   利用 GEOIP 根据 IP 地址找出对应的地理位置坐标
-   利用 useragent 从请求中分析操作系统、设备类型
-   利用 Mutate 从请求中整理字段

#### Grok 插件

将日志行与格式匹配, 生产环境常需要将非结构化的数据解析成 json 结构化数据格式

Grok 非常适合将syslog 日志、apache 和其他 web 服务器日志、MySQL 日志等日志格式转换为JSON格 式。

**转化为grok格式，直接问CHATGPT**

将 Nginx 日志格式化为 json 格式

```shell
# nginx中的一条日志
58.250.250.21 - - [14/Jul/2020:15:07:27 +0800] "GET /wp-content/plugins/akismet/_inc/form.js?ver=4.1.3 HTTP/1.1" 200 330 "http://www.wangxiaochun.com/?p=117" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36" "-"
```

```shell
input {
    tcp {
        port => "9527"
        host => "0.0.0.0"
        type => "tcplog"
        codec => json
        mode => "server"
    }
}


filter {
    grok {
        match => {
            # TIME匹配的值，作为timestamp这个健的值，WORD的值，作为log_level这个健的值...
            "message" => "%{COMBINEDAPACHELOG}"
        }
    }
}

output {
    stdout {
        codec => rubydebug
    }
}
```

自定义匹配

[GROK的模式说明以及常用语法_日志服务(SLS)-阿里云帮助中心](https://help.aliyun.com/zh/sls/user-guide/grok-patterns#section-dwq-6dr-cn9)

```shell
mkdir -p /etc/logstash/patterns
vim /etc/logstash/patterns/ufw
UFW_LOG %{SYSLOGTIMESTAMP:timestamp} %{SYSLOGHOST:host} kernel: \[%{NUMBER:uptime:float}\] \[%{DATA:ufw_action}\] IN=%{WORD:interface} (OUT=%{WORD:out_interface}|OUT=) MAC=%{DATA:src_dst_mac} SRC=%{IPV4:src_ip} DST=%{IPV4:dst_ip} LEN=%{NUMBER:packet_length:int} TOS=%{DATA:tos} PREC=%{DATA:prec} TTL=%{NUMBER:ttl:int} ID=%{NUMBER:id:int} PROTO=%{WORD:protocol} SPT=%{NUMBER:src_port:int} DPT=%{NUMBER:dst_port:int} WINDOW=%{NUMBER:window:int} RES=%{DATA:res} %{WORD:tcp_flags} URGP=%{NUMBER:urgp:int}

UFW_SRC_DPT %{SYSLOGTIMESTAMP:timestamp} %{SYSLOGHOST} kernel: .* SRC=%{IPV4:src_ip} .* DPT=%{NUMBER:dst_port:int} .*  

vim /etc/logstash/conf.d/grok.yml

filter {
  grok {
    patterns_dir => ["/etc/logstash/patterns"]
    match => { "message" => "%{UFW_LOG}" }
  }
}

# 测试数据
Mar 16 00:00:35 loong kernel: [8753844.748750] [UFW BLOCK] IN=eth0 OUT= MAC=00:16:3c:9f:f4:e5:00:17:df:b3:a8:00:08:00 SRC=194.180.49.251 DST=148.135.6.161 LEN=40 TOS=0x08 PREC=0x20 TTL=235 ID=55260 PROTO=TCP SPT=51776 DPT=19192 WINDOW=1024 RES=0x00 SYN URGP=0 

/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/grok.yml
{
           "uptime" => 8753844.74875,
            "event" => {
        "original" => "Mar 16 00:00:35 loong kernel: [8753844.748750] [UFW BLOCK] IN=eth0 OUT= MAC=00:16:3c:9f:f4:e5:00:17:df:b3:a8:00:08:00 SRC=194.180.49.251 DST=148.135.6.161 LEN=40 TOS=0x08 PREC=0x20 TTL=235 ID=55260 PROTO=TCP SPT=51776 DPT=19192 WINDOW=1024 RES=0x00 SYN URGP=0\n"
    },
             "tags" => [
        [0] "_jsonparsefailure"
    ],
           "dst_ip" => "148.135.6.161",
              "tos" => "0x08",
         "dst_port" => 19192,
        "timestamp" => "Mar 16 00:00:35",
       "@timestamp" => 2025-03-18T05:27:15.492115155Z,
         "@version" => "1",
        "tcp_flags" => "SYN",
             "prec" => "0x20",
          "message" => "Mar 16 00:00:35 loong kernel: [8753844.748750] [UFW BLOCK] IN=eth0 OUT= MAC=00:16:3c:9f:f4:e5:00:17:df:b3:a8:00:08:00 SRC=194.180.49.251 DST=148.135.6.161 LEN=40 TOS=0x08 PREC=0x20 TTL=235 ID=55260 PROTO=TCP SPT=51776 DPT=19192 WINDOW=1024 RES=0x00 SYN URGP=0",
              "res" => "0x00",
         "protocol" => "TCP",
        "interface" => "eth0",
    "packet_length" => 40,
           "window" => 1024,
             "urgp" => 0,
       "ufw_action" => "UFW BLOCK",
           "src_ip" => "194.180.49.251",
      "src_dst_mac" => "00:16:3c:9f:f4:e5:00:17:df:b3:a8:00:08:00",
              "ttl" => 235,
             "host" => {
        "hostname" => "ubuntu-2204"
    },
               "id" => 55260,
         "src_port" => 51776
}
```

#### GeoIP 插件

geoip 根据 ip 地址提供的对应地域信息，比如:经纬度,国家,城市名等,以方便进行地理数据分析

```shell
filter {
    grok {
        match => {
            match => { "message" => "%{UFW_SRCIP}" }
        }

    geoip {
        # 新版的database字段必须添加
        database => "/usr/share/logstash/vendor/bundle/jruby/3.1.0/gems/logstash-filter-geoip-7.3.1-java/vendor/GeoLite2-City.mmdb"
        source => "src_ip"
        target => "geoip"
    }
}
```

#### Date 插件

Date插件可以将日志中的指定的日期字符串对应的源字段生成新的目标字段。

然后替换@timestamp 字段(此字段默认为当前写入logstash的时间而非日志本身的时间)或指定的其他 字段

```shell
filter {
    grok {
        match => {
            match => { "message" => "%{UFW_SRCIP}" }
        }
        
    #解析源字段timestamp的date日期格式: 14/Jul/2020:15:07:27 +0800
    date {
        match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ] # 这里的时间格式必须和timestamp的实际格式匹配
        target => "access_time"         #将时间写入新生成的access_time字段，源字段仍保留
        #target => "@timestamp"         #将时间覆盖原有的@timestamp字段
        timezone => "Asia/Shanghai"
       }
       
     # Mar 16 00:02:50
    date {
        match => ["timestamp", "MMM dd HH:mm:ss" ] # 这里的时间格式必须和timestamp的实际格式匹配
        target => "access_time"         #将时间写入新生成的access_time字段，源字段仍保留
        #target => "@timestamp"         #将时间覆盖原有的@timestamp字段
        timezone => "Asia/Shanghai"
       } 
}
```

#### Useragent 插件

useragent 插件可以根据请求中的 user-agent 字段，解析出浏览器设备、操作系统等信息, 以方便后续的分析使用

```shell
filter {
    #将nginx日志格式化为json格式
    grok {
        match => {
            "message" => "%{COMBINEDAPACHELOG}" }
        }
    }
    #解析date日期如: 10/Dec/2020:10:40:10 +0800
    date {
        match => ["timestamp", "dd/MMM/yyyy:HH:mm:ss Z" ]
        target => "@timestamp"
        #target => "access_time"
        timezone => "Asia/Shanghai"
    }
    #提取agent字段，进行解析
    useragent {
        #source => "agent"                    #7,X指定从哪个字段获取数据
        source => "message"                   #8.X指定从哪个字段获取数据
        #source => "[user_agent][original]"   #8.X指定从哪个字段获取数据,直接分析日志文件，curl或insomnia此方式不成功
        #source => "[user_agent][original][1]" #8.X指定从哪个字段获取数据，如果利用insomnia此方式不成功工具发送http请求，需要加[1]
        target => "useragent" #指定生成新的字典类型的字段的名称，包括os，device等内容
    }
}
```

#### Mutate 插件

 Mutate 插件主要是对字段进行、类型转换、删除、替换、更新等操作,可以使用以下函数

```shell
remove_field   #删除字段
split          #字符串切割,相当于awk取列
add_field      #添加字段    
convert        #字符串替换
gsub           #类型转换，支持的数据类型：integer，integer_eu，float，float_eu，string，boolean 
rename         #字符串改名
lowercase      #转换字符串为小写 
```

```shell
filter {
    # 删除指定字段
    mutate {
        remove_field => ["timestamp", "message"]
    }
    
    # 以|为分隔符，分隔字段
    mutate {
        split => {"message" => "|"}
    }
    
    mutate {
        # 添加字段，将message的列表的第0个元素添加字段名user_id...
        add_field => {
            "user_id" => "%{[message][0]}"
            "action" => "%{[message][1]}"
            "time" => "%{[message][2]}"
        }
        # 添加字段做索引名
        # add_field => {"[@metadata][target_index]" => "app-%{+YYY.MM.dd}"}
    }
    
    mutate{
        # 数据类型转换
        convert => {
            "user_id" => "integer"
            "action" => "string"
            "time" => "string"
        }
    }
    
    mutate {
        gsub=>["message","/n", " "] #将message字段中的换行替换为空格
    }
    
    
}
```

### 条件判断

```shell
# 
filter {
    if "access" in [tags][0] {
        mutate {
            add_field => {"target_index" => "access-%{+YYYY.MM.dd}"}
        }
    }
    else if "error" in [tags][0] {
        mutate {
            add_field => {"target_index" => "error-%{+YYYY.MM.dd}"}
        }
    }
    else if "system" in [tags][0] {
        mutate {
            add_field => {"target_index" => "system-%{+YYYY.MM.dd}"}
        }
    }

}

output {
    if [fields][env] == "test" {
        elasticsearch {
            hosts => ["10.0.0.100:9200","10.0.0.101:9200","10.0.0.102:9200"]
            index => "test-nginx-%{+YYYY.MM.dd}"
        }
    }
    if [type] == "mysticallog" {
        stdout {
            codec => rubydebug
        }
    }
}
```

### Logstash 输出 Output插件

#### stdout 插件

```shell
output {
    stdout {
        codec => rubydebug
    }
}
```

#### File 插件

输出到文件，可以将分散在多个文件的数据统一存放到一个文件

```shell
output {
    stdout {
        codec => rubydebug
    }
    file {
        path => "/var/log/test.log"
    }
}
```

#### Elasticsearch 插件

```shell
output {
    elasticsearch {
        hosts => ["10.0.0.100:9200", "10.0.0.101:9200", "10.0.0.102:9200"]  # 一般写ES中data节点地址
        index => "app-%{+YYYY.MM.dd}"               # 指定索引名称，建议加时间，按天建立索引
        # index => "%{[@metadata][target_index]}"   # 使用字段[metadata][target_index]值做为索引名
        template_overwrite => true                  # 覆盖索引模版，此项可选，默认值为false
    }
}
```

#### Redis 插件

```shell
output {
    redis {
        host => 'Redis_IP'
        port => "6379"
        password => "123456"
        db => "0"
        data_type => 'list'
        key => "nginx-accesslog"
    }
}
```

#### Kafka 插件

```shell
output {
    kafka {
       bootstrap_server => '10.0.0.105:9092,10.0.0.107:9092,10.0.0.108:9092'
       topic_id => 'nginx-accesslog'
       codec => 'json'   # 如果是Json格式，需要标识的字段
    }
}
```



# Kibana图形显示

Kibana 是一款开源的数据分析和可视化平台，它是 Elastic Stack 成员之一，设计用于和 Elasticsearch 协作,可以使用 Kibana 对 Elasticsearch 索引中的数据进行搜索、查看、交互操作,您可以很方便的利用 图表、表格及地图对数据进行多元化的分析和呈现。

Kibana 可以使大数据通俗易懂。基于浏览器的界面便于您快速创建和分享动态数据仪表板来追踪 Elasticsearch 的实时数据变化。

## 安装并配置 Kibana

可以通过包或者二进制的方式进行安装,可以安装在独立服务器,或者也可以和elasticsearch的主机安装在 一起

[elastic/kibana: Your window into the Elastic Stack](https://github.com/elastic/kibana)

### 包安装

基于性能原因，建议将Kibana安装到独立节点上，而非和ES节点复用

```shell
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.17.3-amd64.deb
sudo dpkg -i kibana-8.17.3-amd64.deb
useradd -r -s /sbin/nologin kibana
```

### 配置

```shell
vim /etc/kibana/kibana.yml
server.port: 5601  #监听端口,此为默认值
server.host: "0.0.0.0" #修改此行的监听地址,默认为localhost，即：127.0.0.1:5601

#修改此行,指向ES任意服务器地址或多个节点地址实现容错,默认为localhost
elasticsearch.hosts: ["http://10.0.0.222:9200","http://10.0.0.223:9200","http://10.0.0.224:9200"] 

i18n.locale: "zh-CN"   #修改此行,使用"zh-CN"显示中文界面,默认英文

#8.X版本新添加配置,默认被注释,会显示下面提示
server.publicBaseUrl: "http://kibana.loong.com"

systemctl start kibana.service
```

磁盘空间不足，elastic不会接收新的分片。

**检查磁盘水位**

查看 `node-2` 和 `node-3` 的磁盘使用情况：

```shell
curl -X GET "http://localhost:9200/_cat/allocation?v"
```

如果 `node` 的 `disk.percent` 接近 85%，Elasticsearch 可能不会分配副本给 `node-2`。

 **解决方法**： 增加磁盘水位阈值：

```shell
curl -X PUT "http://localhost:9200/_cluster/settings" -H "Content-Type: application/json" -d'
{
  "persistent": {
    "cluster.routing.allocation.disk.watermark.low": "95%",
    "cluster.routing.allocation.disk.watermark.high": "97%",
    "cluster.routing.allocation.disk.watermark.flood_stage": "98%",
    "cluster.info.update.interval": "1m"
  }
}'
```

然后再尝试分片分配：

```shell
curl -XPOST "http://localhost:9200/_cluster/reroute?pretty"
```

## 使用 Kibana

### 连接ES

#### Kibana 8.X 开启xpack.security功能连接ES

```shell
#在ES节点上生成token
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token --scope kibana
123456

#查看 verification code
systemctl status kibana.service
Your verification code is:  201 867
```

浏览器访问,填写上面的token

![image-20250315195437267](pic/image-20250315195437267.png)

填写code

![image-20250315195442505](pic/image-20250315195442505.png)

#### Kibana 8.X 禁用xpack.security功能连接ES

![image-20250315195650413](pic/image-20250315195650413.png)

![image-20250315195735457](pic/image-20250315195735457.png)

![image-20250315195750136](pic/image-20250315195750136.png)

![image-20250315195830052](pic/image-20250315195830052.png)

### 管理索引

![image-20250315200133491](pic/image-20250315200133491.png)

```shell
GET _search

GET _search 
{ 
    "query":{
        "match_all":{}
    }
}
```

![image-20250315200334073](pic/image-20250315200334073.png)

创建索引并创建doc

```shell
POST /index_wang/_doc/1
{
    "username": "wang",
    "age": 18,
    "title": "cto"
}

# 查看
GET /index_wang/_doc/1
```

![image-20250315200707082](pic/image-20250315200707082.png)

## Kibana 可视化画图功能

Kibana支持多种图形展示功能，需要日志是json格式的支持

### lens 可视化

垂直条形图

水平条形图

![image-20250318163755156](pic/image-20250318163755156.png)

表

markdown

![image-20250318163918765](pic/image-20250318163918765.png)

指标

热力图

面积图

饼状图

线图 Line Chart

标签云图

地图

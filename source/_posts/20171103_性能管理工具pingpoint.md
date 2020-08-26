---
title: 性能管理工具pingpoint
date: 2017-11-03 22:37:50
tags:
    tools
---


    最近没什么事，研究了一下pingpoint，供以后参考。
    这是一个能够管理大型分布式应用的性能管理工具，通过追踪分布式的事务会话，提供分析整体架构中容器之间的交互的解决方案。
    优点是侵入少，通过几行代码就能安装监控引擎，对应用最低只有3%的影响。
    pingpoint地址：https://github.com/naver/pinpoint

## 整体架构
{% img /img/20171103_性能管理工具pingpoint/01.png %}
## 准备 (一定要注意版本搭配)
    centOS虚拟机3台,2台2G,1台4G(关掉防火墙的)
    jdk 1.8
    tomcat 8
    HBase 1.2.6
    Pingpoint Collector 1.6.2
    Pingpoint Web   1.6.2
    Pingpoint Agent 1.6.2
    zookeeper 3.4.9
### 1.启动zookeeper
    三台虚机地址
    192.168.247.128
    192.168.247.129
    192.168.247.130
    每台都布置了zk
### 2.配置hbase
    hbase存储收集的信息，一般也要分布式的集群，我这里就启动一台
#### 修改hbase-env.sh
    配置自己的jdk的地址
    export JAVA_HOME=/usr/local/jdk1.8.0_101
    用自个的,不用hbase管理zk
    export HBASE_MANAGES_ZK=false
#### 修改hbase-site.xml
    一般hbase要用hdfs做数据落地
    配置zookeeper地址
    配置hbase为分布式的
    配置zookeeper数据存放路径
    面是我自己的配置
{% codeblock %}
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://data1:9000/hbase</value>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>192.168.247.128,192.168.247.129,192.168.247.130</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/usr/local/zookeeper-3.4.9/data/</value>
    </property>
{% endcodeblock %}
#### 在hbase目录执行
    ./bin/start-hbase.sh
    可能会要输入password验明正身，在zookeeper的bin目录下用zkCli.sh去看路径多了一个hbase的znode节点。
    [root@server3 hbase-1.2.6]# jps
    13555 HMaster       
    10794 QuorumPeerMain
    13659 HRegionServer
    18203 Jps
    然后需要初始化pingpoint的数据格式文件hbase-create.hbase，这个文件在下载网页的source code.zip解压完的hbase->scripts文件下面
    然后执行
    ./hbase-1.2.6/bin/hbase shell hbase-create.hbase
    hbaes提供了界面化的工具，hbase部署的ip:端口 192.168.247.130:16010
### 3.配置web,collector,agent
    tomcat的tar包解压，记得修改conf.server.xml的端口号。
    把pinpoint-web-1.6.2.war解压放在tomcat_web/webapps/ROOT,然后启动,地址192.168.247.130:38080
    把pinpoint-collector-1.6.2.war解压在tomcat_collector/webapps/ROOT,然后启动
    把pinpoint-agent-1.6.2.tar.gz解压到/usr/local/pinpoint-agent-1.6.2
    需要监控的程序，比如tomcat
    修改bin/catalina.sh
    在头上加
    CATALINA_OPTS="$CATALINA_OPTS -javaagent:/usr/local/pinpoint-agent-1.6.2/pinpoint-bootstrap-1.6.2.jar"
    CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.agentId=ppserverOne"
    CATALINA_OPTS="$CATALINA_OPTS -Dpinpoint.applicationName=server"
    访问一下测试程序
### 4.监控
    打开tomcat_web的地址192.168.247.130:38080
    主页面
{% img /img/20171103_性能管理工具pingpoint/02.PNG %}
    server状态
{% img /img/20171103_性能管理工具pingpoint/03.PNG %}




 













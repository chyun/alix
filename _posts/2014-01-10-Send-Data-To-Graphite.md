---
title: 给Graphite喂数据
description: >
  自写脚本收集数据，再喂给Graphite
layout: post
categories: 运维
---

最近需要给一台服务器配置一个性能监控方案，由于服务数量较少，只有两台，一台数据库服务器，另一台应用服务器，担心性能监控软件会拖累服务器性能，就没在服务器上装graphite和collectd，在自己的PC上安装了graphite。然后，自己写了几个脚本收集数据，并喂给graphite，以此监控服务器性能。特此记录，以备查询。


服务器端数据收集脚本
================

cpu负载数据采集
---

collectCPU.sh，每分钟收集一次CPU负载信息

```
#!/bin/bash
#collect the loads info.

delay=60
while [ True ];do

	loads=`uptime`
	#echo $loads
	#get the substring of lods, after the last substring of "average:"
	loads=${loads##*average:}
	#echo $loads
	load_1=`echo $loads|awk -F, '{print $1}'`
	#echo $loads
	load_5=`echo $loads|awk -F, '{print $2}'`
	load_15=`echo $loads|awk -F, '{print $3}'`
	#echo $load_1

    #get time stamp
	time=`date '+%s'`
    
    #serverID.cpu.loadavg_1min，这个名字要取得规范一些，方便自己知道这些数据是什么意义
	echo "serverID.cpu.loadavg_1min $load_1 $time"         
	echo "serverID.cpu.loadavg_5min $load_5 $time"
	echo "serverID.cpu.loadavg_15min $load_15 $time"

	sleep $delay
done >>cpuinfo_$(date +%Y%m%d).log       #append to the file
```

输出示例

```
serverID.cpu.loadavg_5min  0.38 1389359414
serverID.cpu.loadavg_15min  0.26 1389359414
serverID.cpu.loadavg_1min 0.32 1389359474
serverID.cpu.loadavg_5min  0.36 1389359474
serverID.cpu.loadavg_15min  0.26 1389359474
```

剩余内存采集
---

collectMem.sh，每分钟收集一次剩余内存

```
#!/bin/bash

while [ TRUE ]; do
	remain_mem=`free -m |sed -n '2p'|awk '{print $4}'`
	echo "serverID.memory.remain $remain_mem $(date +%s)"
	sleep 60
done >>memory_$(date +%Y%m%d).log

```

输出示例：

```
serverID.memory.remain 1917 1389359739
serverID.memory.remain 1921 1389359799
serverID.memory.remain 1921 1389359859
serverID.memory.remain 1919 1389359919
```

剩余磁盘容量采集
---

collectDisk.sh，每分钟收集一次剩余磁盘容量

```
#!/bin/bash

while [ TRUE ]; do
     #focus: I first use "df -h|sed -n '2p'|awk '{print $4}'" to get
    #the remain memory, but the result ends up with a 'G', the graphite
    #can't recognise it, so I cut off the 'G'
	remain_disk=`df -h|sed -n '2p'|awk '{print $4}'|grep -oP '\d+'`
	echo "serverID.remain $remain_disk $(date +%s)"
	sleep 60
done >>disk_$(date +%Y%m%d).log
```
输出示例：

```
serverID.disk.remain 21 1389359877
serverID.disk.remain 21 1389359937
serverID.disk.remain 21 1389359997
serverID.disk.remain 21 1389360057
```

每秒网络流量采集
---

collectNetwork.sh，每分钟收集一次网络流量

```
#!/bin/bash

delay=60

while [ TRUE ]; do

	R1=`cat /sys/class/net/eth0/statistics/rx_bytes`
	T1=`cat /sys/class/net/eth0/statistics/tx_bytes`
	sleep 1
	R2=`cat /sys/class/net/eth0/statistics/rx_bytes`
	T2=`cat /sys/class/net/eth0/statistics/tx_bytes`

	TBPS=`expr $T2 - $T1`
	RBPS=`expr $R2 - $R1`

	TKBPS=`expr $TBPS / 1024`
	RKBPS=`expr $RBPS / 1024`

	echo "serverID.network.upload $TKBPS $(date +%s)"

	echo "serverID.network.download $RKBPS $(date +%s)"

	sleep $delay
done >>network_$(date +%Y%m%d).log
```

连接数采集
---

collectConnectionNum.sh，每分钟收集一次与8080端口的连接数目

```
#!/bin/bash

delay=60
while [ TRUE ]; do
	conn_num=`netstat -anp|grep tcp|awk '{print $4}'|grep 8080|wc -l`
	echo "serverID.connection.number $conn_num $(date +%s)"
	sleep $delay
done >>connectionNum_$(date +%Y%m%d).log
```

给graphite发送数据
===

有了数据就可以喂给praphite，让他画图了。graphite依靠后台carbon进程来接收数据，carbon的监听端口时2003。因此，我们只需将收集到的数据发送到2003端口就OK了。

首先，打开一个tcp连接

```
exec 3<>/dev/tcp/127.0.0.1/2003
```

接着发送数据

```
cat cpu_20140110.log >&3
cat memory_20140110.log >&3
cat disk_20140110.log >&3
```

记得发送玩之后，要关闭tcp连接

```
exec 3<&-
exec 3>&-
```

carbon接收数据后，graphite就会画图了，如下所示。

![enter image description here][1]

这里有[一篇给graphite发送数据的参考文章][2]。

其中提到了在发送数据之前，要在/opt/graphite/conf/storage-schemas.conf文件中创建一个schema，用于数据保存。我什么都没配就OK了，可能我的版本更新一些。


  [1]: https://github.com/chyun/Blog/blob/gh-pages/images/20140110_cpuinfo.png?raw=true
  [2]: http://graphite.wikidot.com/getting-your-data-into-graphite

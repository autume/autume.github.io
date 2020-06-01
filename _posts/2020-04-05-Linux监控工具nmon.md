---
layout: post
title: Linux监控工具nmon
categories: Linux
keywords: nmon,Linux
---

nmon是一种在linux操作系统上广泛使用的监控与分析工具，能在系统运行过程中实时捕捉系统资源的使用情况，并输出结果到文件中，然后通过nmon_analyzer工具产生数据文件与图形化结果。

## 下载与安装
apt install nmon
## nmon用法

```
nmon -f -F demo.nmon -s 1 -c 10 -t
```

- -f：输出文件，文件名为默认名称
- -F <filename>: 自定义输出文件名称
- -s: 采集数据频率，也就是保存数据的频率
- -c：采集次数
- -t: 输出最消耗资源的进程数据


## nmon_anylyzer
下载：https://www.ibm.com/developerworks/community/wikis/home?lang=en#!/wiki/Power+Systems/page/nmon_analyser
重点sheet：
- SYS_SUMM:系统汇总页，包含cup占有率变化情况，磁盘IO变化情况等
- AAA: 关于操作系统以及nmon本身的一些信息
- CPUnn：显示执行时间内CPU占用情况
- CPU_ALL:所有CPU概述，显示所有CPU平均占用情况
- CPU_SUMM:每一个CPU在执行时间内的占用情况
- DGWRITE:每个磁盘的平均写情况
- DGXFER：每个磁盘组的I/O每秒操作
- MEM:内存相关的主要信息，使用、空闲内存大小等
- NET: 显示系统中每个网络适配器的数据传输速率（千字节、秒）


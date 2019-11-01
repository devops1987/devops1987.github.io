---
title: Kafka常用操作记录
author: qfxl
catalog: true
tags:
    - Kafka
---

> 记录kafka日常操作，持续更新。

#启动kafka
```shell
# /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties &
或者（以守护进程的方式启动）：
# /opt/kafka/bin/kafka-server-start.sh -daemon /opt/kafka/config/server.properties
```


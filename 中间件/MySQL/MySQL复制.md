MySQL 复制是高可用的基础

目标：搭建高可用的的 MySQL 架构，防止因物理硬件故障导致业务不可用。

方案：架设 MySQL 集群，开启主从复制

## 高可用

高可用是指系统**无中断**地执行其功能的能力，表示系统的可用性程度。高可用的指标：可用性 A

$$
A=平均故障间隔时间/（平均故障间隔时间+平均恢复时间）
$$

平均故障间隔时间 MTBF 越高说明故障次数越少，平均恢复时间 MTRR 越短说明系统受影响时长越短。

对于 MySQL 来说，降低平均恢复时间的方案

- 不使用单节点数据库，一旦单节点挂掉系统就全部瘫痪

- 使用数据库自动化运维管理平台，自动化恢复集群，加速故障恢复

使用统一管理方案实现对整个数据库集群的实时监控、实时评估集群的健康情况、发现故障节点，实时告警并及时降低服务等级或下线故障服务节点。

MySQL 集群节点自然要考虑主从数据一致性的问题。MySQL 中提供了复制技术实现主从库的数据同步

## MySQL 复制架构

MySQL 复制的本质就是数据同步。基于二进制日志(binary log)增量同步,二进制日志记录所有对 MySQL 数据库的修改操作。

### 优势

- 业务量庞大，单机无法满足，多库存储可降低磁盘I/O访问频率，提高单机的性能

- 读写分离，主库负责写，从库负责读，可以有效提高性能，且如果主库出现锁表也不会影响正常业务

- 数据热备份，主从复制，一个节点坏掉不影响线上服务功能

### 复制基本过程

- Master 库将数据写入到 binary log 文件中，然后通过 Dump 线程发送给从库

- 从库中 I/O 线程接收binary log 文件，并保存为中继日志

- 执行线程负责并行执行中继日志，即在从库服务器上重新执行主库产生的日志

## 开启主从复制

1. 创建从库的账号和权限，

2. 从主库备份一份数据

3. 搭建复制关系，使用命令：change master to

修改主库的配置文件my.cnf，开启二进制日志，设置 service 

```bash
[mysqld]
    log-bin=mysql-bin # 启动二进制日志
    server-id=1       # 服务器唯一ID，默认为1，一般以IP值最后一段
```

修改从库的配置文件 my.cnf, 每个从库的 server-id 都是唯一的

```bash
server-id =2
# 保证主从数据的一致性
relay_log_recovery = ON
master_info_repository = TABLE 
relay_log_info_repository = TABLE
```

设置完后都需要重启使配置生效

```bash
systemctl restart mysqld
```

配置主从库通信

```bash
mysql> change master to master_host='',master_port='',master_user='',master_password='',master_log_file='',
```

start slave启动从服务器复制线程

```bash
mysql> start slave; 
mysql> show slave status \G; #检查从库状态
```

参考资料：

- https://learn.lianglianglee.com/%E4%B8%93%E6%A0%8F/MySQL%E5%AE%9E%E6%88%98%E5%AE%9D%E5%85%B8/15%20%20MySQL%20%E5%A4%8D%E5%88%B6%EF%BC%9A%E6%9C%80%E7%AE%80%E5%8D%95%E4%B9%9F%E6%9C%80%E5%AE%B9%E6%98%93%E9%85%8D%E7%BD%AE%E5%87%BA%E9%94%99.md)

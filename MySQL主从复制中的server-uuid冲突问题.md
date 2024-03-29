### 配置MySQL的主从复制，从机的server-uuid与主机相同(冲突)
辛苦在VMware中新建了一台虚拟机，在其中安装了常用软件(MySQL)并作了各种配置，
为了省事，用VMware的clone功能快速克隆出另外一台新虚拟机，用以测试MySQL的主从复制功能。

### 可能出现如下问题：
```
mysql> show slave status\G;
Slave_IO_Running: No
。。。。。。。
Last_IO_Error: Fatal error: The slave I/O thread stops because master and slave have equal MySQL server UUIDs; these UUIDs must be different for replication to work.
。。。。。。。
```
![pic](https://github.com/tyouken/MySQL-series/raw/main/graph/MySQL%20server-uuid%20conflict.jpg)

```
cd /var/lib/mysql
```
- 修改auto.cnf文件中的server-uuid值，或者删除该文件(删除后，MySQL启动时会重建)

- 重启MySQL

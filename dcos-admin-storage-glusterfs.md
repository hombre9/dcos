## GlusterFS存储

### 存储规划

三个节点（node-24，node-25和node-26），每个节点一个独立的SSD硬盘\(\/srv\)，作为GlusterFS存储磁盘。

操作系统：CentOS 7.2.1511，内核：4.8.6。

**注意，在实际进行资源实施前，推荐充分阅读并理解\*\***[GlusterFS的架构](http://gluster.readthedocs.io/en/latest/Quick-Start-Guide/Architecture/)**\*\*及相关文档，并结合业务需求及发展做出具体规划。**

### 服务安装

#### 安装软件并启动服务

首先确保添加了EPEL仓库，然后安装：

```
yum -y install centos-release-gluster
```

该RPM提供了在CentOS上安装GlusterFS所需的软件库。

安装GlusterFS Server：

```
yum -y install glusterfs-server
```

启动GlusterFS服务：

```
systemctl start glusterd.service && systemctl enable glusterd.service
```

```
[root@node-24 ~]# systemctl status glusterd.service
● glusterd.service - GlusterFS, a clustered file-system server Loaded: 
    loaded (/usr/lib/systemd/system/glusterd.service; enabled; vendor preset: disabled) 
    Active: active (running) since 三 2016-11-23 11:25:45 CST; 6s ago 
Main PID: 143134 (glusterd) 
    Memory: 14.0M 
    CGroup: /system.slice/glusterd.service 
            └─143134 /usr/sbin/glusterd -p /var/run/glusterd.pid --log-level INFO

11月 23 11:25:45 node-24 systemd[1]: Starting GlusterFS, a clustered file-system server...
11月 23 11:25:45 node-24 systemd[1]: Started GlusterFS, a clustered file-system server.
```

#### 配置hosts

在\/etc\/hosts中添加节点记录：

```
192.168.1.24 node-24
192.168.1.25 node-25
192.168.1.26 node-26
```

#### 配置网络环境

可以通过 iptables 规则过滤 gluster，根据缺省值，glusterd 将会在 tcp\/24007 上监听，但只在 gluster 节点上打开该端口并不足够。每加入一台新的机器，它将会打开一个新的端口（可以通过 gluster volume status 来查阅） 。根据实际环境需求，利用专用的网卡来传输 gluster／存储器的数据是较好的选择。

在测试或安全可控环境，可以在gluster节点间禁用防火墙。

#### 建立互信节点组

```
[root@node-24 ~]# gluster peer probe node-25
peer probe: success.
[root@node-24 ~]# gluster peer probe node-26
peer probe: success.
[root@node-24 ~]# gluster peer statusNumber of Peers: 2

Hostname: node-25
Uuid: cc6ca3b8-8734-4e79-993d-c4dfa6502593
State: Peer in Cluster (Connected)

Hostname: node-26
Uuid: 1d08d373-155d-4116-8582-c12a3902b4fb
State: Peer in Cluster (Connected)
```

#### 创建数据存储目录

在三个节点上，分别执行：

```
[root@node-24 ~]# mkdir -p /srv/data
[root@node-25 ~]# mkdir -p /srv/data
[root@node-26 ~]# mkdir -p /srv/data
```

#### 创建GlusterFS磁盘

注意，在实际创建GlusterFS磁盘前，请再次阅读[GlusterFS的架构](http://gluster.readthedocs.io/en/latest/Quick-Start-Guide/Architecture/)并根据业务需求做出具体的磁盘构建决策。

此处采用复制模式。创建volume 时带 replica x 数量，将文件复制到 replica x 个节点中。

```
[root@node-24 srv]# gluster volume create glusterfs replica 3 node-24:/srv/data node-25:/srv/data node-26:/srv/data forcevolume create: glusterfs: success: please start the volume to access data
```

#### 查看Volume状态

```
[root@node-24 srv]# gluster volume info

Volume Name: glusterfs
Type: Replicate
Volume ID: 013b9479-c76e-4b1e-8615-f09e40aa0bb7
Status: Created
Snapshot Count: 0
Number of Bricks: 1 x 3 = 3
Transport-type: tcp
Bricks:
Brick1: node-24:/srv/data
Brick2: node-25:/srv/data
Brick3: node-26:/srv/data
Options Reconfigured:
transport.address-family: inet
performance.readdir-ahead: on
nfs.disable: on
```

#### 启动glusterfs卷

```
[root@node-24 srv]# gluster volume start glusterfs
volume start: glusterfs: success
```

### 使用创建的glusterfs卷

#### 设置GlusterFS客户端

客户端为node-22。配置hosts文件，使其能够识别node-24，node-25和node-26。

安装GlusterFS客户端:

```
[root@node-22 ~]# yum install -y glusterfs-client
```

创建挂载目录：

```
[root@node-22 mnt]# mkdir /mnt/glusterfs
```

挂载glusterfs卷（注意，此处可以是`node-25`或`node-26`）

```
[root@node-22 mnt]# mount.glusterfs node-24:/glusterfs /mnt/glusterfs
[root@node-22 mnt]# mount
......
node-24:/glusterfs on /mnt/glusterfs type fuse.glusterfs (rw,relatime,user_id=0,group_id=0,default_permissions,allow_other,max_read=131072)
```

运行`df -h`命令：

```
[root@node-22 mnt]# df -h
文件系统 容量 已用 可用 已用% 挂载点
devtmpfs 16G 0 16G 0% /dev
tmpfs 16G 0 16G 0% /dev/shm
tmpfs 16G 1.4G 14G 9% /run
tmpfs 16G 0 16G 0% /sys/fs/cgroup
tmpfs 3.1G 0 3.1G 0% /run/user/0
node-24:/glusterfs 112G 33M 112G 1% /mnt/glusterfs
```

添加挂载指令到`/etc/rc.local`

```
/usr/sbin/mount.glusterfs node-24:/glusterfs /mnt/glusterfs
```

现在就可以在使用`/mnt/glusterfs`了。

#### 测试

在node-22的`/mnt/glusterfs`下创建一个文件：

```
[root@node-22 glusterfs]# vim test
```

在node-26上`/srv/data`下查看：

```
[root@node-26 ~]# cd /srv/data/
[root@node-26 data]# ll
总用量 4
-rw-r--r-- 2 root root 4 11月 23 14:58 test
```

### GlusterFS性能调优

开启指定Volume的配额：\(`glusterfs`为Volume的名称\)

```
gluster volume quota glusterfs enable
```

限制`glusterfs`中“`/`”\(既根目录\)的最大使用 80GB 空间

```
gluster volume quota glusterfs limit-usage / 80GB
```

设置`cache`缓存为4GB

```
gluster volume set glusterfs performance.cache-size 4GB
```

开启异步，后台操作

```
gluster volume set glusterfs performance.flush-behind on
```

设置IO线程数为32

```
gluster volume set glusterfs performance.io-thread-count 32
```

设置回写（写数据时间，先写入缓存内，再写入硬盘）

```
gluster volume set glusterfs performance.write-behind on
```


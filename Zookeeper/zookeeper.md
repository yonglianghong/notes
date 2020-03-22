#### zookeeper的由来

>雅虎内部很多项目的名字都是使用动物来命名的，所以演变成一个”动物园“，刚好这也是一个用来协调分布式环境的，于是zookeeper应运而生。



#### zookeeper cli

启动cli

```shell
bin/zkCli.sh -server hzabj-ad-data21.server.163.org:2181
```

显示help

```shell
[zk: hzabj-ad-data21.server.163.org:2181(CONNECTED) 0] help
ZooKeeper -server host:port cmd args
	stat path [watch]
	set path data [version]
	ls path [watch]
	delquota [-n|-b] path
	ls2 path [watch]
	setAcl path acl
	setquota -n|-b val path
	history
	redo cmdno
	printwatches on|off
	delete path [version]
	sync path
	listquota path
	rmr path
	get path [watch]
	create [-s] [-e] path data acl
	addauth scheme auth
	quit
	getAcl path
	close
	connect host:port
```

查看znode

```shell
[zk: hzabj-ad-data21.server.163.org:2181(CONNECTED) 1] ls /
[ad_pangu, consumers, brokers, ad, zookeeper, kafka-manager]
```

创建znode，并赋值

```shell
[zk: hzabj-ad-data21.server.163.org:2181(CONNECTED) 2] create /zk_test my_data
```

获取znode

```shell
[zk: hzabj-ad-data21.server.163.org:2181(CONNECTED) 4] get /zk_test
my_data
cZxid = 0x3004b40fb
ctime = Tue Jul 02 11:09:47 CST 2019
mZxid = 0x3004b40fb
mtime = Tue Jul 02 11:09:47 CST 2019
pZxid = 0x3004b40fb
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 0
```

修改znode

```shell
[zk: hzabj-ad-data21.server.163.org:2181(CONNECTED) 6] set /zk_test junk
cZxid = 0x3004b40fb
ctime = Tue Jul 02 11:09:47 CST 2019
mZxid = 0x3004b418a
mtime = Tue Jul 02 11:10:36 CST 2019
pZxid = 0x3004b40fb
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0
```

删除znode

```shell
[zk: hzabj-ad-data21.server.163.org:2181(CONNECTED) 7] delete /zk_test
```


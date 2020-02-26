## 集群部署

### 硬件

#### Master server

- AWS m5.2xlarge
  - 8个vCPU
  - 31 GB内存

#### Data server

- 两个AWS i3.4xlarge
  - 16个vCPU
  - 122 GB内存
  - 2 * 1.9TB SSD储存空间

#### Query server

- AWS m5.2xlarge
  - 8 vCPUs
  - 31 GB RAM

### 下载Druid

```shell
wget http://apache.01link.hk/incubator/druid/0.16.0-incubating/apache-druid-0.16.0-incubating-bin.tar.gz
tar -xzf apache-druid-0.16.0-incubating-bin.tar.gz
cd apache-druid-0.16.0-incubating
```

apache-druid-0.16.0-incubating里有以下文件与目录：

- `DISCLAIMER`, `LICENSE`和`NOTICE`文件
- `bin/*` 与[单机部署](https://druid.apache.org/docs/latest/tutorials/index.html)相关的脚本
- `conf/druid/cluster/*` 集群部署的模板配置
- `extensions/*`  druid的核心扩展
- `hadoop-dependencies/*`  Druid Hadoop依赖项
- `lib/*` Druid的核心库和依赖项
- `quickstart/*` 与[单机部署](https://druid.apache.org/docs/latest/tutorials/index.html)相关的文件

下面的配置都在文件夹`conf/druid/cluster/`内完成，并且应该先在一台机器上完成配置然后再拷贝到其他机器上。

### 配置

#### metadata storage

在`conf/druid/cluster/_common/common.runtime.properties`里, 将以下两项替换为将用作元数据存储的计算机的地址：
`druid.metadata.storage.connector.connectURI`
`druid.metadata.storage.connector.host`

#### deep storage

- 在`conf/druid/cluster/_common/common.runtime.properties`
  - 将`druid-s3-extensions`添加到`druid.extensions.loadList`
  - 注释掉关于storage和indexer的本地配置
  - 针对S3进行配置，参阅[S3配置](https://druid.apache.org/docs/latest/development/extensions-core/s3.html)

```properties
# 例子
druid.extensions.loadList=["druid-s3-extensions"]

#druid.storage.type=local
#druid.storage.storageDirectory=var/druid/segments

druid.storage.type=s3
druid.storage.bucket=your-bucket
druid.storage.baseKey=druid/segments
druid.s3.accessKey=...
druid.s3.secretKey=...

#druid.indexer.logs.type=file
#druid.indexer.logs.directory=var/druid/indexing-logs

druid.indexer.logs.type=s3
druid.indexer.logs.s3Bucket=your-bucket
druid.indexer.logs.s3Prefix=druid/indexing-logs
```

#### Zookeeper连接

Zookeeper集群与Druid服务器应该分开部署

在`conf/druid/cluster/_common/common.runtime.properties`，将`druid.zk.service.host`设置为ZooKeeper服务器de `host:port`，多个用逗号分隔（例如`127.0.0.1:4545`或` 127.0.0.1:3000,127.0.0.1:3001`）

#### 配置调整

如果使用的是开头的示例集群，则无需调整，否则根据[这里](https://druid.apache.org/docs/latest/operations/basic-cluster-tuning.html)进行调整

#### 开放端口

- master server
  - 1527(Derby metadata store;如果使用mysql或者postgresql则不需要)
  - 2181(ZooKeeper; 如果使用单独的zk集群则不需要)
  - 8081(Coordinator)
  - 8090(Overlord)

- data server
  - 8083(Historical)
  - 8091, 8100-8199(Druid Middle Manager; 如果`druid.worker.capacity`的值很大，端口号可以大于8199)
- query server
  - 8082(Broker)
  - 8088(Router, if used)

- 其他
  - 8200 (Tranquility Server, if used)

### 启动master server

将配置好的druid拷贝至master server，在druid根目录执行：

```shell
bin/start-cluster-master-no-zk-server
```

### 启动data server

将配置好的druid拷贝至data server，在druid根目录执行：

```
bin/start-cluster-data-server
```

### Tranquility

### 启动query server

将配置好的druid拷贝至query server，在druid根目录执行：

```
bin/start-cluster-query-server
```


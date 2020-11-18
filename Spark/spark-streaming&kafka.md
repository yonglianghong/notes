#### 1. 知识点

a) kafka分区自适应

Spark Streaming 自适应上游 kafka topic partition 数目变化



b) kafka.common.OffsetOutOfRangeException，意味着消费的offset已经过期（kafka定期清理策略）

- 如果设置的smallest（earliest），则只需要比较每个partition消费的offset与kafka最小的offset，如果小于，则替换成kafka最小的offset 
- 如果设置的largest（latest），同理，替换成kafka最大的就行 



c) spark-streaming优雅停止

1. 自带特效

   ```scala
   conf.set("spark.streaming.stopGracefullyOnShutdown","true")
   ```

2. 通过touch hdfs path标记，如果检测到标记，就停止程序

   ```scala
       val shutdownMarker = "/tmp/spark-test/stop-spark"
       var stopFlag: Boolean = false
       //判断文件系统是否存在文件
       @transient val hadoopconf = spark.sparkContext.hadoopConfiguration
       @transient val fs = org.apache.hadoop.fs.FileSystem.get(hadoopconf)
       //检查间隔毫秒
       val checkIntervalMillis = 1000*3600
       var isStopped = false
       while (!isStopped){
         isStopped = ssc.awaitTerminationOrTimeout(checkIntervalMillis)
         if (isStopped) {
           println("confirmed! The streaming context is stopped. Exiting application...")
         } else {
           println("Streaming App is still running. Timeout...")
         }
         if(getPathOrFileLen(fs, shutdownMarker) != 0 && !stopFlag){
           stopFlag = true
         }
         if(!isStopped && stopFlag){
           println("stopping ssc right now")
           //第一个true：停止相关的SparkContext。无论这个流媒体上下文是否已经启动，底层的SparkContext都将被停止。
           //第二个true：则通过等待所有接收到的数据的处理完成，从而优雅地停止。
           ssc.stop(true, true)
           println("ssc is stopped!!!!!!!")
   
         }
   
       }
   ```

   

#### 2. kafka offset 管理

> KafkaOffsetsHbase.scala

```scala
import kafka.common.TopicAndPartition
import kafka.utils.ZkUtils
import org.I0Itec.zkclient.ZkClient
import org.apache.hadoop.hbase.client.{ConnectionFactory, Put, Scan}
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.hbase.{HBaseConfiguration, TableName}
import org.apache.spark.SparkException
import org.apache.spark.streaming.kafka.KafkaCluster.LeaderOffset
import org.apache.spark.streaming.kafka.{KafkaCluster, OffsetRange}

import scala.collection.immutable.Map

/**
  * Suitable for kafka 0.8
  */
object KafkaOffsetsHbase {

  /*
   Returns last committed offsets for all the partitions of a given topic from HBase in following cases.
   - CASE 1: SparkStreaming job is started for the first time. This function gets the number of topic partitions from
     Zookeeper and for each partition returns the last committed offset as 0
   - CASE 2: SparkStreaming is restarted and there are no changes to the number of partitions in a topic. Last
     committed offsets for each topic-partition is returned as is from HBase.
   - CASE 3: SparkStreaming is restarted and the number of partitions in a topic increased. For old partitions, last
     committed offsets for each topic-partition is returned as is from HBase as is. For newly added partitions,
     function returns last committed offsets as 0
   - CASE 4: If the value stored by hbase expires in kafka, it is replaced with the offset in kafka, depending on "smallest" or "largest".
  */
  def getLastCommittedOffsets(TOPIC_NAME: String, GROUP_ID: String, hbaseTableName: String, zkQuorum: String,
                                 zkRootDir: String, sessionTimeout: Int, connectionTimeOut: Int, offsetReset: String, kc: KafkaCluster): Map[TopicAndPartition, Long] = {
    val hbaseConf = HBaseConfiguration.create()
    hbaseConf.addResource("/hbase-site.xml")
    val zkUrl = zkQuorum + "/" + zkRootDir
    val zkClient = new ZkClient(zkUrl, sessionTimeout, connectionTimeOut)
    val zKNumberOfPartitionsForTopic = ZkUtils.getPartitionsForTopics(zkClient, Seq(TOPIC_NAME)).get(TOPIC_NAME).toList.head.size

    //Connect to HBase to retrieve last committed offsets
    val conn = ConnectionFactory.createConnection(hbaseConf)
    val table = conn.getTable(TableName.valueOf(hbaseTableName))
    val startRow = TOPIC_NAME + ":" + GROUP_ID + ":" + String.valueOf(System.currentTimeMillis())
    val stopRow = TOPIC_NAME + ":" + GROUP_ID + ":" + 0
    val scan = new Scan()
    val scanner = table.getScanner(scan.setStartRow(startRow.getBytes).setStopRow(stopRow.getBytes).setReversed(true))
    val result = scanner.next()

    var hbaseNumberOfPartitionsForTopic = 0 //Set the number of partitions discovered for a topic in HBase to 0
    if (result != null) {
      //If the result from hbase scanner is not null, set number of partitions from hbase to the number of cells
      hbaseNumberOfPartitionsForTopic = result.listCells().size()
    }
    var fromOffsets = collection.mutable.Map[TopicAndPartition, Long]()
    // get kc partition offset
    val kcFromOffsets = collection.mutable.Map[TopicAndPartition, Long]()
    if (offsetReset != null && kc != null) {
      val reset: Option[String] = Some(offsetReset.toLowerCase)
      var leaderOffsets: Map[TopicAndPartition, LeaderOffset] = null
      val partitionsE = kc.getPartitions(Set(TOPIC_NAME))
      if (partitionsE.isLeft) throw new SparkException("get kafka partition failed:")
      val partitions = partitionsE.right.get
      if (reset.contains("smallest")) {
        leaderOffsets = kc.getEarliestLeaderOffsets(partitions).right.get
      } else { // largest
        leaderOffsets = kc.getLatestLeaderOffsets(partitions).right.get
      }
      val offsets = leaderOffsets.map {
        case (tp, offset) => (tp, offset.offset)
      }
      for (o <- offsets.toList) {
        kcFromOffsets += o
      }
    }
    // initial fromOffsets
    if (hbaseNumberOfPartitionsForTopic == 0) {
      fromOffsets = kcFromOffsets
    } else {
      if (zKNumberOfPartitionsForTopic > hbaseNumberOfPartitionsForTopic) {
        // handle scenario where new partitions have been added to existing kafka topic
        for (partition <- 0 until hbaseNumberOfPartitionsForTopic) {
          val fromOffset = Bytes.toString(result.getValue(Bytes.toBytes("offsets"), Bytes.toBytes(partition.toString)))
          fromOffsets += (new TopicAndPartition(TOPIC_NAME, partition) -> fromOffset.toLong)
        }
        for (partition <- hbaseNumberOfPartitionsForTopic until zKNumberOfPartitionsForTopic) {
          fromOffsets += (new TopicAndPartition(TOPIC_NAME, partition) -> 0)
        }
      } else {
        //initialize fromOffsets from last run
        for (partition <- 0 until hbaseNumberOfPartitionsForTopic) {
          val fromOffset = Bytes.toString(result.getValue(Bytes.toBytes("offsets"), Bytes.toBytes(partition.toString)))
          fromOffsets += (new TopicAndPartition(TOPIC_NAME, partition) -> fromOffset.toLong)
        }
      }
      fromOffsets.map {
        case (tp, o) => {
          val n = kcFromOffsets.getOrElse(tp, 0L)
          if (o < n)
            fromOffsets.put(tp, n) // override old value
        }
      }
    }
    scanner.close()
    conn.close()
    fromOffsets.toMap
  }


  /*
   Save Offsets into HBase
   */
  def saveOffsets(TOPIC_NAME: String, GROUP_ID: String, offsetRanges: Array[OffsetRange], hbaseTableName: String,
                     batchTime: org.apache.spark.streaming.Time) = {
    val hbaseConf = HBaseConfiguration.create()
    hbaseConf.addResource(this.getClass.getResourceAsStream("/hbase-site.xml"))
    val conn = ConnectionFactory.createConnection(hbaseConf)
    val table = conn.getTable(TableName.valueOf(hbaseTableName))
    val rowKey = TOPIC_NAME + ":" + GROUP_ID + ":" + String.valueOf(batchTime.milliseconds)
    val put = new Put(rowKey.getBytes)
    for (offset <- offsetRanges) {
      put.addColumn(Bytes.toBytes("offsets"), Bytes.toBytes(offset.partition.toString),
        Bytes.toBytes(offset.untilOffset.toString))
    }
    table.put(put)
    conn.close()
  }

}
```



> Kafka010OffsetsHbase.scala

```scala
import kafka.common.TopicAndPartition
import kafka.utils.ZkUtils
import org.apache.hadoop.hbase.client.{ConnectionFactory, Put, Scan}
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.hbase.{HBaseConfiguration, TableName}
import org.apache.spark.SparkException
import org.apache.spark.streaming.kafka.KafkaCluster
import org.apache.spark.streaming.kafka.KafkaCluster.LeaderOffset
import org.apache.spark.streaming.kafka010.OffsetRange

import scala.collection.immutable.Map

/**
  * Suitable for kafka 1.0+
  */
object Kafka010OffsetsHbase {

  /*
   Returns last committed offsets for all the partitions of a given topic from HBase in following cases.
    - CASE 1: SparkStreaming job is started for the first time. This function gets the number of topic partitions from
      Zookeeper and for each partition returns the last committed offset as 0
    - CASE 2: SparkStreaming is restarted and there are no changes to the number of partitions in a topic. Last
      committed offsets for each topic-partition is returned as is from HBase.
    - CASE 3: SparkStreaming is restarted and the number of partitions in a topic increased. For old partitions, last
      committed offsets for each topic-partition is returned as is from HBase as is. For newly added partitions,
      function returns last committed offsets as 0
    - CASE 4: If the value stored by hbase expires in kafka, it is replaced with the offset in kafka, depending on "earliest" or "latest".
   */
  def getLastCommittedOffsets(TOPIC_NAME: String, GROUP_ID: String, hbaseTableName: String, zkQuorum: String,
                              zkRootDir: String, sessionTimeout: Int, connectionTimeOut: Int, offsetReset: String, kc: KafkaCluster): Map[TopicAndPartition, Long] = {
    val hbaseConf = HBaseConfiguration.create()
    hbaseConf.addResource("/hbase-site.xml")
    val zkUrl = zkQuorum + "/" + zkRootDir
    val zkClientAndConnection = ZkUtils.createZkClientAndConnection(zkUrl, sessionTimeout, connectionTimeOut)
    val zkUtils = new ZkUtils(zkClientAndConnection._1, zkClientAndConnection._2, false)
    val zKNumberOfPartitionsForTopic = zkUtils.getPartitionsForTopics(Seq(TOPIC_NAME)).get(TOPIC_NAME).toList.head.size

    //Connect to HBase to retrieve last committed offsets
    val conn = ConnectionFactory.createConnection(hbaseConf)
    val table = conn.getTable(TableName.valueOf(hbaseTableName))
    val startRow = TOPIC_NAME + ":" + GROUP_ID + ":" + String.valueOf(System.currentTimeMillis())
    val stopRow = TOPIC_NAME + ":" + GROUP_ID + ":" + 0
    val scan = new Scan()
    val scanner = table.getScanner(scan.setStartRow(startRow.getBytes).setStopRow(stopRow.getBytes).setReversed(true))
    val result = scanner.next()

    var hbaseNumberOfPartitionsForTopic = 0 //Set the number of partitions discovered for a topic in HBase to 0
    if (result != null) {
      //If the result from hbase scanner is not null, set number of partitions from hbase to the number of cells
      hbaseNumberOfPartitionsForTopic = result.listCells().size()
    }

    var fromOffsets = collection.mutable.Map[TopicAndPartition, Long]()
    // get kc partition offset
    val kcFromOffsets = collection.mutable.Map[TopicAndPartition, Long]()
    if (offsetReset != null && kc != null) {
      val reset: Option[String] = Some(offsetReset.toLowerCase)
      var leaderOffsets: Map[TopicAndPartition, LeaderOffset] = null
      val partitionsE = kc.getPartitions(Set(TOPIC_NAME))
      if (partitionsE.isLeft) throw new SparkException("get kafka partition failed:")
      val partitions = partitionsE.right.get
      if (reset.contains("earliest")) {
        leaderOffsets = kc.getEarliestLeaderOffsets(partitions).right.get
      } else {
        // latest
        leaderOffsets = kc.getLatestLeaderOffsets(partitions).right.get
      }
      val offsets = leaderOffsets.map {
        case (tp, offset) => (tp, offset.offset)
      }
      for (o <- offsets.toList) {
        kcFromOffsets += o
      }
    }
    // initial fromOffsets
    if (hbaseNumberOfPartitionsForTopic == 0) {
      fromOffsets = kcFromOffsets
    } else {
      if (zKNumberOfPartitionsForTopic > hbaseNumberOfPartitionsForTopic) {
        // handle scenario where new partitions have been added to existing kafka topic
        for (partition <- 0 until hbaseNumberOfPartitionsForTopic) {
          val fromOffset = Bytes.toString(result.getValue(Bytes.toBytes("offsets"), Bytes.toBytes(partition.toString)))
          fromOffsets += (new TopicAndPartition(TOPIC_NAME, partition) -> fromOffset.toLong)
        }
        for (partition <- hbaseNumberOfPartitionsForTopic until zKNumberOfPartitionsForTopic) {
          fromOffsets += (new TopicAndPartition(TOPIC_NAME, partition) -> 0)
        }
      } else {
        //initialize fromOffsets from last run
        for (partition <- 0 until hbaseNumberOfPartitionsForTopic) {
          val fromOffset = Bytes.toString(result.getValue(Bytes.toBytes("offsets"), Bytes.toBytes(partition.toString)))
          fromOffsets += (new TopicAndPartition(TOPIC_NAME, partition) -> fromOffset.toLong)
        }
        fromOffsets.map {
          case (tp, o) => {
            val n = kcFromOffsets.getOrElse(tp, 0L)
            if (o < n)
              fromOffsets.put(tp, n) // override old value
          }
        }
      }
    }
    scanner.close()
    conn.close()
    fromOffsets.toMap
  }

  /*
   Save Offsets into HBase
   */
  def saveOffsets(TOPIC_NAME: String, GROUP_ID: String, offsetRanges: Array[OffsetRange], hbaseTableName: String,
                  batchTime: org.apache.spark.streaming.Time) = {
    val hbaseConf = HBaseConfiguration.create()
    hbaseConf.addResource(this.getClass.getResourceAsStream("/hbase-site.xml"))
    val conn = ConnectionFactory.createConnection(hbaseConf)
    val table = conn.getTable(TableName.valueOf(hbaseTableName))
    val rowKey = TOPIC_NAME + ":" + GROUP_ID + ":" + String.valueOf(batchTime.milliseconds)
    val put = new Put(rowKey.getBytes)
    for (offset <- offsetRanges) {
      put.addColumn(Bytes.toBytes("offsets"), Bytes.toBytes(offset.partition.toString),
        Bytes.toBytes(offset.untilOffset.toString))
    }
    table.put(put)
    conn.close()
  }
  
}
```



> KafkaOffsetsZk.scala

```scala
import kafka.common.TopicAndPartition
import kafka.message.MessageAndMetadata
import kafka.serializer.Decoder
import org.apache.spark.SparkException
import org.apache.spark.rdd.RDD
import org.apache.spark.streaming.StreamingContext
import org.apache.spark.streaming.dstream.InputDStream
import org.apache.spark.streaming.kafka.KafkaCluster.LeaderOffset
import org.apache.spark.streaming.kafka.{HasOffsetRanges, KafkaCluster, KafkaUtils, OffsetRange}

import scala.reflect.ClassTag

/**
  * 适用于kafka0.8版本
  * 参考: https://blog.csdn.net/kwu_ganymede/article/details/50930962
  * @param kafkaParams
  */
class KafkaOffsetsZk(val kafkaParams: Map[String, String]) extends Serializable {

  private val kc = new KafkaCluster(kafkaParams)
  private val flag = 1150 * 10000l //可按需定义

  /**
    * 创建数据流
    * createDirectStream[String, String, StringDecoder, StringDecoder, (String, String)](ssc, kafkaParams, topic)
    */
  def createDirectStream[K: ClassTag, V: ClassTag, KD <: Decoder[K] : ClassTag, VD <: Decoder[V] : ClassTag](ssc: StreamingContext, kafkaParams: Map[String, String], topics: Set[String]): InputDStream[(K, V)] = {
    val groupId = kafkaParams.get("group.id").get
    // 在zookeeper上读取offsets前先根据实际情况更新offsets
    setOrUpdateOffsets(topics, groupId)

    //从zookeeper上读取offset开始消费message
    val messages = {
      val partitionsE = kc.getPartitions(topics)
      if (partitionsE.isLeft)
        throw new SparkException(s"get kafka partition failed: ${partitionsE.left.get}")
      val partitions = partitionsE.right.get
      val consumerOffsetsE = kc.getConsumerOffsets(groupId, partitions)
      if (consumerOffsetsE.isLeft)
        throw new SparkException(s"get kafka consumer offsets failed: ${consumerOffsetsE.left.get}")
      val consumerOffsets = consumerOffsetsE.right.get
      KafkaUtils.createDirectStream[K, V, KD, VD, (K, V)](
        ssc, kafkaParams, consumerOffsets, (mmd: MessageAndMetadata[K, V]) => (mmd.key, mmd.message))
    }
    messages
  }


  /**
    * 创建数据流前，根据实际消费情况更新消费offsets
    * @param topics
    * @param groupId
    */
  private def setOrUpdateOffsets(topics: Set[String], groupId: String): Unit = {
    topics.foreach(topic => {
      var hasConsumed = true
      val partitionsE = kc.getPartitions(Set(topic))
      if (partitionsE.isLeft)
        throw new SparkException(s"get kafka partition failed: ${partitionsE.left.get}")
      val partitions = partitionsE.right.get
      val consumerOffsetsE = kc.getConsumerOffsets(groupId, partitions)
      if (consumerOffsetsE.isLeft) hasConsumed = false
      if (hasConsumed) {
        // 消费过
        // 如果streaming程序执行的时候出现kafka.common.OffsetOutOfRangeException，
        // 说明zk上保存的offsets已经过期了，即kafka的定时清理策略已经将包含该offsets的文件删除。
        // 针对这种情况，只要判断一下zk上的consumerOffsets和earliestLeaderOffsets的大小，
        // 如果consumerOffsets比earliestLeaderOffsets还小的话，说明consumerOffsets已过时,
        // 这时把consumerOffsets更新为earliestLeaderOffsets
        val earliestLeaderOffsetsE = kc.getEarliestLeaderOffsets(partitions)
        if (earliestLeaderOffsetsE.isLeft)
          throw new SparkException(s"get earliest leader offsets failed: ${earliestLeaderOffsetsE.left.get}")
        val earliestLeaderOffsets = earliestLeaderOffsetsE.right.get
        val consumerOffsets = consumerOffsetsE.right.get

        // 可能只是存在部分分区consumerOffsets过时，所以只更新过时分区的consumerOffsets为earliestLeaderOffsets
        var offsets: Map[TopicAndPartition, Long] = Map()
        consumerOffsets.foreach({
          case (tp, n) =>
            val earliestLeaderOffset = earliestLeaderOffsets(tp).offset
            if (n < earliestLeaderOffset) {
              println("consumer group:" + groupId + ",topic:" + tp.topic + ",partition:" + tp.partition +
                " offsets已经过时，更新为" + earliestLeaderOffset)
              offsets += (tp -> earliestLeaderOffset)
            }
        })
        if (offsets.nonEmpty) {
          kc.setConsumerOffsets(groupId, offsets)
        }
      }
      else {
        // 没有消费过
        val reset = kafkaParams.get("auto.offset.reset").map(_.toLowerCase)
        var leaderOffsets: Map[TopicAndPartition, LeaderOffset] = null
        if (reset.contains("smallest")) {
          val leaderOffsetsE = kc.getEarliestLeaderOffsets(partitions)
          if (leaderOffsetsE.isLeft)
            throw new SparkException(s"get earliest leader offsets failed: ${leaderOffsetsE.left.get}")
          leaderOffsets = leaderOffsetsE.right.get
        } else { // largest
          val leaderOffsetsE = kc.getLatestLeaderOffsets(partitions)
          if (leaderOffsetsE.isLeft)
            throw new SparkException(s"get latest leader offsets failed: ${leaderOffsetsE.left.get}")
          leaderOffsets = leaderOffsetsE.right.get
        }
        val offsets = leaderOffsets.map {
          case (tp, offset) => (tp, offset.offset)
        }
        kc.setConsumerOffsets(groupId, offsets)
      }
    })
  }


  /**
    * 更新zookeeper上的消费offsets
    * 把当前的消费记录，写入zk
    * @param rdd
    */
  def updateZKOffsets(rdd: RDD[(String, String)]): Unit = {
    val groupId = kafkaParams.get("group.id").get
    val offsetsList = rdd.asInstanceOf[HasOffsetRanges].offsetRanges

    for (offsets <- offsetsList) {
      val topicAndPartition = TopicAndPartition(offsets.topic, offsets.partition)
      val o = kc.setConsumerOffsets(groupId, Map((topicAndPartition, offsets.untilOffset)))
      if (o.isLeft) {
        println(s"Error updating the offset to Kafka cluster: ${o.left.get}")
      }
    }
  }


  /**
    * 更新zookeeper上的消费offsets
    * 把当前的消费记录的offset往前推
    * 并写入zk
    * @param offsetRanges
    * @param day
    */
  def updateZKOffsetsFromoffsetRanges(offsetRanges: Array[OffsetRange], day: Double): Unit = {
    val groupId = kafkaParams.get("group.id").get

    for (offsets <- offsetRanges) {
      val topicAndPartition = TopicAndPartition(offsets.topic, offsets.partition)

      var offsetStreaming = 0l
      println("offsets.untilOffset " + offsets.untilOffset)

      // 如果streaming挂掉，则从偏移量的前flag开始计算
      // 由于在streaming里的window函数中进行了去重处理
      // 因此不用担心数据重复的问题
      if (offsets.untilOffset >= flag) {
        offsetStreaming = offsets.untilOffset - (flag * day).toLong
      } else {
        offsetStreaming = 0
      }
      println("offsetStreaming " + offsetStreaming)

      val o = kc.setConsumerOffsets(groupId, Map((topicAndPartition, offsetStreaming)))
      if (o.isLeft) {
        println(s"Error updating the offset to Kafka cluster: ${o.left.get}")
      }
    }
  }
  
}
```



> 依赖jar

```wiki
    <!--测试版本-->
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-streaming-kafka-0-10_2.11</artifactId>
        <version>${spark.version}</version>
    </dependency>

    <!--稳定版本-->
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-streaming-kafka-0-8_2.11</artifactId>
        <version>${spark.version}</version>
    </dependency>

    <!--kafka 1.1.0-->
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka_2.11</artifactId>
        <version>1.1.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>1.1.0</version>
    </dependency>

    <!--kafka 0.8.2.1-->
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka_2.11</artifactId>
        <version>0.8.2.1</version>
    </dependency>
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
        <version>0.8.2.1</version>
    </dependency>

    <!--zookeeper-->
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
        <version>3.4.6</version>
    </dependency>
    <dependency>
        <groupId>com.101tec</groupId>
        <artifactId>zkclient</artifactId>
        <version>0.8</version>
    </dependency>
    
    <!--scala-logging-->
    <dependency>
       <groupId>com.typesafe.scala-logging</groupId>
       <artifactId>scala-logging_2.11</artifactId>
       <version>3.7.2</version>
    </dependency>
```


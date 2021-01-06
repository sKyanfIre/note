# kafka-producer

#### 1.Producter实体

* 事务

* 发送消息

* 度量指标

* 获取topic分区信息

#### 2.Product实例

##### KafkaProducer

  ###### `关键组件`

  *  [分区器 Partitioner](#partitioner)

  * [序列化器 Serializer](#serializer)

  * 度量指标 Metrics

  * 读取配置  <font color=pink> ProducerConfig </font>

  * 事务管理 TransactionManager

  * 生产者线程 Sender

  * 压缩方式 CompressionType(<font color="grey">NONE,GZIP,SNAPPY,LZ4,ZSTD</font>)

  * [消息缓存 RecordAccumulator](#recordAccumulator)

    

#####   3.发送方式

* 消息实体 <font color=yellowgreen>ProducerRecord</font>

```java
public class ProducerRecord<K, V> {
	
    private final String topic;
    private final Integer partition;
    private final Headers headers;
    private final K key;
    private final V value;
    private final Long timestamp;
}
```



* 返回元数据 <font color=yellowgreen>RecordMetadata</font>

```java
public final class RecordMetadata {
     /**
     * Partition value for record without partition assigned
     */
    public static final int UNKNOWN_PARTITION = -1;
    private final long offset;
    private final long timestamp;
    private final int serializedKeySize;
    private final int serializedValueSize;
    private final TopicPartition topicPartition;
    private volatile Long checksum;
}     
```



1. 同步方式

```java
// demo
Future<RecordMetadata> sendFuture =
				producer.send(producerRecord, buildCallback(producerRecord, producer, future, sample));
	sendFuture.get();

```

2. 异步方式

```java
// interface 
public interface Producer<K, V> extends Closeable {
     Future<RecordMetadata> send(ProducerRecord<K, V> record);
}
```



3. 异步回调方式

```java
// interface 
public interface Producer<K, V> extends Closeable {
    Future<RecordMetadata> send(ProducerRecord<K, V> record, Callback callback);
}
// demo
Future<RecordMetadata> sendFuture =
				producer.send(producerRecord, buildCallback(producerRecord, producer, future, sample));
```

###### <a id='partitioner'>分区 Partitioner</a>

* <font color="yellow">默认分区器 DefaultPartitioner</font>

  1.key为null时，使用缓存和随机方式分区

```java
public int partition(String topic, Cluster cluster) {
        Integer part = indexCache.get(topic);
        if (part == null) {
            return nextPartition(topic, cluster, -1);
        }
        return part;
    }
// nextPartition() 部分代码
if (availablePartitions.size() < 1) {
                Integer random = Utils.toPositive(ThreadLocalRandom.current().nextInt());
                newPart = random % partitions.size();
            } else if (availablePartitions.size() == 1) {
                newPart = availablePartitions.get(0).partition();
            } else {
                while (newPart == null || newPart.equals(oldPart)) {
                    Integer random = Utils.toPositive(ThreadLocalRandom.current().nextInt());
                    newPart = availablePartitions.get(random % availablePartitions.size()).partition();
                }
            }
```

2. key不为空时，hash取模运算

```java
 public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        if (keyBytes == null) {
            return stickyPartitionCache.partition(topic, cluster);
        } 
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        // hash the keyBytes to choose a partition
        return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
    }
```



* <font color="yellow">轮询分区 RoundRobinPartitioner</font>

```java
  public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        int nextValue = nextValue(topic);
        List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
        if (!availablePartitions.isEmpty()) {
            int part = Utils.toPositive(nextValue) % availablePartitions.size();
            return availablePartitions.get(part).partition();
        } else {
            // no partitions are available, give a non-available partition
            return Utils.toPositive(nextValue) % numPartitions;
        }
    }
```

##### 4.发送过程

​	<a id="recordAccumulator">消息缓存 RecordAccumulator</a>

```java
  public final class RecordAccumulator {
    private final AtomicInteger flushesInProgress;
    private final AtomicInteger appendsInProgress;
    private final int batchSize;
    private final CompressionType compression;
    private final int lingerMs;
    private final long retryBackoffMs;
    private final int deliveryTimeoutMs;
    private final BufferPool free;
    // copy on write map
    private final ConcurrentMap<TopicPartition, Deque<ProducerBatch>> batches;
   
  }
```

​		消息批次`ProducerBatch`

​		内存写入消息 `MemoryRecordsBuilder`

​		内存消息缓存池 `BufferPool`

​		消息缓存单位 `ByteBuffer`

1. 消息按``topic``和``partition``分组发送

2. 分组之后的消息按批发送

   
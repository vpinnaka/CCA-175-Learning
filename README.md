# CCA-175-Learning

### labs.itversity details
gw01.itversity.com
vinaydatta


pyspark --master yarn --conf spark.ui.port=12808


## Table of Contents
- [Flume](https://github.com/vpinnaka/CCA-175-Learning#flume)
- [Kafka](https://github.com/vpinnaka/CCA-175-Learning#kafka)
- [Spark Streaming](https://github.com/vpinnaka/CCA-175-Learning#spark-streaming)

## FLUME

Flume config file to get the streaming data to store in HDFS format, other parameters such as file suffix, format, size and type are also set in the configuration
- Source: exec
- Sink: hdfs
- Channel: memory
```
# example.conf: A single-node Flume configuration

# Name the components on this agent
wh.sources = ws
wh.sinks = hd
wh.channels = mem

# Describe/configure the source
wh.sources.ws.type = exec
wh.sources.ws.command = tail -F /opt/gen_logs/logs/access.log

# Describe the sink
wh.sinks.hd.type = hdfs
wh.sinks.hd.hdfs.path = hdfs://nn01.itversity.com:8020/user/vinaydatta/flume_example

wh.sinks.hd.hdfs.filePrefix = flume_example
wh.sinks.hd.hdfs.fileSuffix = .txt
wh.sinks.hd.hdfs.rollInterval = 120
wh.sinks.hd.hdfs.rollSize = 10485760
wh.sinks.hd.hdfs.rollCount = 100
wh.sinks.hd.hdfs.fileType = DataStream

# Use a channel which buffers events in memory
wh.channels.mem.type = memory
wh.channels.mem.capacity = 1000
wh.channels.mem.transactionCapacity = 100

# Bind the source and sink to the channel
wh.sources.ws.channels = mem
wh.sinks.hd.channel = mem

```

Command to execute flume, to get near-real time streaming data and store in HDFS
```
flume-ng agent -n wh -f /home/vinaydatta/FlumeExample/webLogs/wshdfs.conf 
```
[Documentation Link](https://archive.cloudera.com/cdh5/cdh/5/flume-ng/FlumeUserGuide.html#hdfs-sink)


## Kafka
Create a topic named 'kafka_demo' in kafka
```
bin/kafka-topics.sh --create --zookeeper nn01.itversity.com:2181, nn02.itversity.com:2181, rm01.itversity.com:2181 --replication-factor 1 --partitions 1 --topic kafka_demo
```
See thr topic with
```
bin/kafka-topics.sh --list --zookeeper nn01.itversity.com:2181, nn02.itversity.com:2181, rm01.itversity.com:2181 --topic kafka_demo
```
Send some messages
```
bin/kafka-console-producer.sh --broker-list nn01.itversity.com:6667, nn02.itversity.com:6667, rm01.itversity.com:6667 --topic kafka_demo
```
Create a consumer
```
bin/kafka-console-consumer.sh  --zookeeper nn01.itversity.com:2181, nn02.itversity.com:2181, rm01.itversity.com:2181 --topic kafka_demo --from-beginning
```
[Documentation Link](https://kafka.apache.org/quickstart)

## Spark Streaming
Word count program using spark streaming
```
from pyspark import SparkContext, SparkConf
from pyspark.streaming import StreamingContext

conf = SparkConf().setAppName("Spark Streaming Word Count").setMaster("yarn-client")
sc = SparkContext(conf=conf)
ssc = StreamingContext(sc, 15)

lines = ssc.socketTextStream("gw01.itversity.com", 19999)
words = lines.flatMap(lambda lines: line.split(" "))
wordPairs = words.map(lambda word: (word, 1))
wordCounts = wordPairs.reduceByKey(lambda x, y : x + y)

wordCounts.pprint()

ssc.start()
ssc.awaitTermination()

```
spark submit command
```
spark-submit --master yarn --conf spark.ui.port=12890 src/main/python/StreamingWordCountVinay.py
```
- *Real-time application* - Getting Departments count in retail db
```
from pyspark import SparkContext, SparkConf
from pyspark.streaming import StreamingContext
from operator import add
import sys

hostname = sys.argv[1]
port = int(sys.argv[2])

conf = SparkConf().setAppName("Streaming department Count").setMaster("yarn-client")
sc = SparkContext(conf=conf)

# Create a local StreamingContext with two working thread and batch interval of 30 second
ssc = StreamingContext(sc, 30)

messages = ssc.socketTextStream(hostname, port)
departmentMessages = messages.filter(lambda msg: msg.split(' ')[6].split('/')[1] == 'department') \
                             .map(lambda msg: (msg.split(' ')[6].split('/')[2], 1)) 
departmentCount = departmentMessages.reduceByKey(add)

outputPrefix = sys.argv[3]
departmentCount.saveAsTextFiles(outputPrefix)

ssc.start()             # Start the computation
ssc.awaitTermination()  # Wait for the computation to terminate
```
[Documentation](http://spark.apache.org/docs/1.6.0/streaming-programming-guide.html#a-quick-example)











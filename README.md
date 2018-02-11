# CCA-175-Learning
----------------------------

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

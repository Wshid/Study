## Flume 설정
- flume에 대한 기본 설정에 대해 기록

### 구동 pipeline
- ![image](https://user-images.githubusercontent.com/10006290/52528707-30ab8680-2d28-11e9-8fc2-5a1a29e2aa88.png)
- 실제 데이터 유입시, `source -> channel -> sink`의 과정을 거친다.

### 특징
- 장애시에도 수집된 로그 유실 방지가 가능
- scale-up/scale-out 방식의 확장 모두 지원
- source와 sink는 별도의 thread로 동작

### 설정파일 예시(`kafka` -> `flume` -> `HDFS`)
```
# $FLUME_HOME/conf/flume.conf
## tier1은 한 노드의 이름이다.
 tier1.sources  = source1
 tier1.channels = channel1
 tier1.sinks = sink1
 
 tier1.sources.source1.type = org.apache.flume.source.kafka.KafkaSource
 tier1.sources.source1.kafka.bootstrap.servers = kafka-broker01.example.com:9092
 tier1.sources.source1.kafka.topics = weblogs
 tier1.sources.source1.kafka.consumer.group.id = flume
 tier1.sources.source1.channels = channel1
 tier1.sources.source1.interceptors = i1
 tier1.sources.source1.interceptors.i1.type = timestamp
 tier1.sources.source1.kafka.consumer.timeout.ms = 100
 
 tier1.channels.channel1.type = memory #type을 메모리로 설정
 tier1.channels.channel1.capacity = 10000
 tier1.channels.channel1.transactionCapacity = 1000
 
 tier1.sinks.sink1.type = hdfs
 tier1.sinks.sink1.hdfs.path = /tmp/kafka/%{topic}/%y-%m-%d
 tier1.sinks.sink1.hdfs.rollInterval = 5 # File을 rolling하기 위해 대기할 시간(s)
 tier1.sinks.sink1.hdfs.rollSize = 0 # File Roll Trigger, 0 is NEVEL ROLL
 tier1.sinks.sink1.hdfs.rollCount = 0 # roll 하기 전 이벤트 발생 횟수(0 is NEVER ROLL)
 tier1.sinks.sink1.hdfs.fileType = DataStream
    # SequenceFile, DataStream, CompressedStream
 tier1.sinks.sink1.channel = channel1
```
- `kafka.consumer.group.id`를 동일하게 설정시,
    - multi partition일때, 성능을 향상시킬 수 있음
    - 다른 소스에서 데이터를 읽어오기 때문

referenced by. 
- [taewan kim](http://taewan.kim/post/flume_images/)
- [FlumeUserGuide kafka source](https://flume.apache.org/FlumeUserGuide.html#kafka-source)
- [Cloudera](https://www.cloudera.com/documentation/kafka/latest/topics/kafka_flume.html)
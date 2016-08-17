# Storm and Kafka
pom.xml excerpt
```xml
...
    <dependencies>
        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>storm-core</artifactId>
            <version>${storm-core.version}</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>storm-kafka</artifactId>
            <version>${storm-kafka.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.10</artifactId>
            <version>${kafka.version}</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
...
```

run.sh
```bash
storm jar \
  uber-storm-kafka-solr-example-1.0-SNAPSHOT.jar \
  StormKafkaExample \
  "broker1.fqdn.com:6667" \
  topic
```

StormKafkaExample
```java
import backtype.storm.Config;
import backtype.storm.StormSubmitter;
import backtype.storm.topology.IRichSpout;
import backtype.storm.topology.TopologyBuilder;
import storm.kafka.bolt.KafkaBolt;
import storm.kafka.bolt.mapper.FieldNameBasedTupleToKafkaMapper;
import storm.kafka.bolt.selector.DefaultTopicSelector;

import java.util.Properties;

public class StormKafkaExample {

  public static void main(String[] args) throws Exception {
    String brokerList = args[0];
    String topic = args[1];

    TopologyBuilder builder = new TopologyBuilder();

    IRichSpout spout = new MySpout();
    builder.setSpout("spout", spout);

    KafkaBolt<String, String> bolt = new KafkaBolt<String, String>()
        .withTopicSelector(new DefaultTopicSelector(topic))
        .withTupleToKafkaMapper(new FieldNameBasedTupleToKafkaMapper<>());
    builder.setBolt("forwardToKafka", bolt).shuffleGrouping("spout");

    Config conf = new Config();
    Properties props = new Properties();
    props.put("metadata.broker.list", brokerList);
    props.put("request.required.acks", "1");
    props.put("serializer.class", "kafka.serializer.StringEncoder");
    props.put("security.protocol", "PLAINTEXTSASL");
    conf.put(KafkaBolt.KAFKA_BROKER_PROPERTIES, props);

    StormSubmitter.submitTopology("kafkaBoltTest", conf, builder.createTopology());
  }
}
```
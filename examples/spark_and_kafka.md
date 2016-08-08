# Spark and Kafka
## Spark & Kafka - Kerberos
spark-kafka.jaas
```
KafkaClient {
  com.sun.security.auth.module.Krb5LoginModule required
  useTicketCache=false
  useKeyTab=true
  principal="principal@EXAMPLE.COM"
  keyTab="v.keytab"
  renewTicket=true
  storeKey=true
  serviceName="kafka";
};
Client {
  com.sun.security.auth.module.Krb5LoginModule required
  useTicketCache=false
  useKeyTab=true
  principal="principal@EXAMPLE.COM"
  keyTab="v.keytab"
  renewTicket=true
  storeKey=true
  serviceName="zookeeper";
};
```

### Read
run.sh
```bash
spark-submit \
  --verbose \
  --master yarn \
  --deploy-mode client \
  --files spark-kafka.jaas#spark-kafka.jaas,v.keytab#v.keytab \
  --driver-java-options "-Djava.security.auth.login.config=./spark-kafka.jaas" \
  --conf "spark.executor.extraJavaOptions=-Djava.security.auth.login.config=./spark-kafka.jaas" \
  --class "SparkKafkaKeberosRead" \
  uber-spark-kafka-kerberos-1.0-SNAPSHOT.jar \
  TOPIC
```

SparkKafkaKerberosRead
```java
import java.util.HashMap;
import java.util.Map;

import org.apache.spark.SparkConf;
import org.apache.spark.storage.StorageLevel;
import org.apache.spark.streaming.Seconds;
import org.apache.spark.streaming.api.java.JavaPairReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka.KafkaUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import kafka.serializer.StringDecoder;

public class SparkKafkaKeberosRead {

  // Spark information
  private static final String appName = SparkKafkaKeberosRead.class.getSimpleName();
  private static final Logger logger = LoggerFactory.getLogger(SparkKafkaKeberosRead.class);

  // Kafka information
  private static final String zkQuorum  = "zk1.fqdn.com,zk2.fqdn.com,zk3.fqdn.com";
  private static final String group = "test_spark_kerberos";

  public static void main(String[] args) {

    // Check that all required args were passed in.
    if (args.length != 1) {
      System.err.println("Usage: SparkKafkaKeberosRead <topic> \n");
      System.exit(1);
    }

    // Configure the application
    SparkConf conf = configureSpark();

    // Create the context
    JavaStreamingContext context = createContext(conf, args[0]);

    // Stop the application
    context.start();
    context.awaitTermination();
  }

  /**
   * This is the kernel of the spark application
   */
  private static JavaStreamingContext createContext(SparkConf conf, String topic) {

    logger.info("-------------------------------------------------------");
    logger.info("|            Starting: {}             |", appName);
    logger.info("-------------------------------------------------------");

    // Create the spark streaming context
    JavaStreamingContext context = new JavaStreamingContext(conf, Seconds.apply(5));

    Map<String, String> kafkaParams = new HashMap<>();
    kafkaParams.put("group.id", group);
    kafkaParams.put("auto.offset.reset", "smallest");
    kafkaParams.put("security.protocol", "PLAINTEXTSASL");
    kafkaParams.put("zookeeper.connect", zkQuorum);

    Map<String, Integer> topics = new HashMap<>();
    topics.put(topic, 1);

    // Read from a Kerberized Kafka
    JavaPairReceiverInputDStream<String, String> kafkaStream =
        KafkaUtils.createStream(context, String.class, String.class, StringDecoder.class, StringDecoder.class,
            kafkaParams, topics, StorageLevel.MEMORY_AND_DISK_SER());

    kafkaStream.print();

    logger.info("-------------------------------------------------------");
    logger.info("|            Finished: {}             |", appName);
    logger.info("-------------------------------------------------------");

    return context;
  }

  /**
   * Create a SparkConf and configure it.
   */
  private static SparkConf configureSpark() {
    logger.info(">- Initializing '%s'.", appName);
    SparkConf conf = new SparkConf().setAppName(appName);

    logger.info(">- Configuration done with the follow properties:");
    logger.info(conf.toDebugString());

    return conf;
  }
}
```

### Read & Write
run.sh
```bash
spark-submit \
  --verbose \
  --master yarn \
  --deploy-mode client \
  --files spark-kafka.jaas#spark-kafka.jaas,v.keytab#v.keytab \
  --driver-java-options "-Djava.security.auth.login.config=./spark-kafka.jaas" \
  --conf "spark.executor.extraJavaOptions=-Djava.security.auth.login.config=./spark-kafka.jaas" \
  --class "SparkKafkaKeberosReadWrite" \
  uber-spark-kafka-kerberos-1.0-SNAPSHOT.jar \
  READ_TOPIC WRITE_TOPIC
```

SparkKafkaKerberosReadWrite
```java
import kafka.javaapi.producer.Producer;
import kafka.producer.KeyedMessage;
import kafka.producer.ProducerConfig;
import kafka.serializer.StringDecoder;
import org.apache.spark.SparkConf;
import org.apache.spark.storage.StorageLevel;
import org.apache.spark.streaming.Seconds;
import org.apache.spark.streaming.api.java.JavaPairReceiverInputDStream;
import org.apache.spark.streaming.api.java.JavaStreamingContext;
import org.apache.spark.streaming.kafka.KafkaUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import scala.Tuple2;

import java.util.HashMap;
import java.util.Map;
import java.util.Properties;

public class SparkKafkaKeberosReadWrite {

  // Spark information
  private static final String appName = SparkKafkaKeberosReadWrite.class.getSimpleName();
  private static final Logger logger = LoggerFactory.getLogger(SparkKafkaKeberosReadWrite.class);

  // Kafka information
  private static final String zkQuorum  = "zk1.fqdn.com,zk2.fqdn.com,zk3.fqdn.com";
  private static final String group = "test_spark_kerberos";

  public static void main(String[] args) {

    // Check that all required args were passed in.
    if (args.length != 2) {
      System.err.println("Usage: SparkKafkaKeberosReadWrite <readTopic> <writeTopic> \n");
      System.exit(1);
    }

    // Configure the application
    SparkConf conf = configureSpark();

    // Create the context
    JavaStreamingContext context = createContext(conf, args[0], args[1]);

    // Stop the application
    context.start();
    context.awaitTermination();
  }

  /**
   * This is the kernel of the spark application
   */
  private static JavaStreamingContext createContext(SparkConf conf, String readTopic, String writeTopic) {

    logger.info("-------------------------------------------------------");
    logger.info("|            Starting: {}             |", appName);
    logger.info("-------------------------------------------------------");

    // Create the spark streaming context
    JavaStreamingContext context = new JavaStreamingContext(conf, Seconds.apply(5));

    Map<String, String> kafkaParams = new HashMap<>();
    kafkaParams.put("group.id", group);
    kafkaParams.put("auto.offset.reset", "smallest");
    kafkaParams.put("security.protocol", "PLAINTEXTSASL");
    kafkaParams.put("zookeeper.connect", zkQuorum);

    Properties props = new Properties();
    props.put("metadata.broker.list", "broker1.fqdn.com:6667");
    props.put("serializer.class", "kafka.serializer.StringEncoder");
    props.put("request.required.acks", "1");
    props.put("security.protocol", "PLAINTEXTSASL");

    Map<String, Integer> topics = new HashMap<>();
    topics.put(readTopic, 1);

    // Read from a Kerberized Kafka
    JavaPairReceiverInputDStream<String, String> kafkaStream =
        KafkaUtils.createStream(context, String.class, String.class, StringDecoder.class, StringDecoder.class,
            kafkaParams, topics, StorageLevel.MEMORY_AND_DISK_SER());

    kafkaStream.print();
    kafkaStream.foreachRDD(rdd -> {
      rdd.foreachPartition(partition -> {
        ProducerConfig config = new ProducerConfig(props);
        Producer<String, String> producer = new Producer<>(config);
        while (partition.hasNext()) {
          Tuple2<String, String> tuple2 = partition.next();
          KeyedMessage<String, String> data = new KeyedMessage<>(writeTopic, tuple2._1(), tuple2._2());
          producer.send(data);
        }
      });
      return null;
    });

    logger.info("-------------------------------------------------------");
    logger.info("|            Finished: {}             |", appName);
    logger.info("-------------------------------------------------------");

    return context;
  }

  /**
   * Create a SparkConf and configure it.
   */
  private static SparkConf configureSpark() {
    logger.info(">- Initializing '%s'.", appName);
    SparkConf conf = new SparkConf().setAppName(appName);

    logger.info(">- Configuration done with the follow properties:");
    logger.info(conf.toDebugString());

    return conf;
  }
}
```
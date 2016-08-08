# Spark and HBase
## Spark & HBase - Kerberos
run.sh
```bash
#!/usr/bin/env bash

CP="/etc/hbase/conf:/usr/hdp/current/hadoop-client/hadoop-common.jar:/usr/hdp/current/hadoop-client/lib/guava-11.0.2.jar:/usr/hdp/current/hbase-client/lib/hbase-client.jar:/usr/hdp/current/hbase-client/lib/hbase-common.jar:/usr/hdp/current/hbase-client/lib/hbase-protocol.jar:/usr/hdp/current/hbase-client/lib/hbase-server.jar:/usr/hdp/current/hbase-client/lib/hbase-hadoop2-compat.jar:/usr/hdp/current/hbase-client/lib/htrace-core-3.1.0-incubating.jar"

# Needed if deploy-mode is cluster. Comment out if deploy-mode is client.
export SPARK_CLASSPATH=$SPARK_CLASSPATH:$CP

spark-submit \
  --verbose \
  --master yarn \
  --deploy-mode cluster \
  --driver-class-path "$CP" \
  --conf "spark.executor.extraClassPath=$CP" \
  --class "SparkHBaseKerberos" \
  uber-hbase-kerberos-1.0-SNAPSHOT.jar
```

SparkHBaseKerberos
```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.apache.hadoop.hbase.client.Result;
import org.apache.hadoop.hbase.io.ImmutableBytesWritable;
import org.apache.hadoop.hbase.mapreduce.TableInputFormat;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaSparkContext;

public class SparkHBaseKerberos {
  public static <K, V> void main(String[] args) throws Exception {
    System.out.println("Starting");

    SparkConf sparkConf = new SparkConf().setAppName(SparkHBaseKerberos.class.getCanonicalName());
    JavaSparkContext jsc = new JavaSparkContext(sparkConf);

    Configuration config = HBaseConfiguration.create();

    try (Connection connection = ConnectionFactory.createConnection(config);
         Admin admin = connection.getAdmin()) {

      for(TableName tableName : admin.listTableNames()) {
        System.out.println(tableName.getNameAsString());

        config.set(TableInputFormat.INPUT_TABLE, tableName.getNameAsString());
        JavaPairRDD<ImmutableBytesWritable, Result> rdd = jsc.newAPIHadoopRDD(config, TableInputFormat.class, ImmutableBytesWritable.class, Result.class);
        System.out.println("Number of Records found: " + rdd.count());
      }
    }

    System.out.println("Done");

    jsc.stop();
    jsc.close();
  }
}
```
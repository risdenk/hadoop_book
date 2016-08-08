# Spark and HDFS

## Spark & HDFS - Kerberos
run.sh
```bash
#!/usr/bin/env bash

CP="$(hadoop classpath)"

# Needed if deploy-mode is cluster. Comment out if deploy-mode is client.
export SPARK_CLASSPATH=$SPARK_CLASSPATH:$CP

spark-submit \
  --verbose \
  --master yarn \
  --deploy-mode cluster \
  --driver-class-path "$CP" \
  --conf "spark.executor.extraClassPath=$CP" \
  --class "SparkHDFSKerberos" \
  uber-hdfs-kerberos-1.0-SNAPSHOT.jar
```

SparkHDFSKerberos
```java
import org.apache.hadoop.conf.Configuration;
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaSparkContext;

public class SparkHDFSKerberos {
  public static <K, V> void main(String[] args) throws Exception {
    System.out.println("Starting");

    SparkConf sparkConf = new SparkConf().setAppName(SparkHDFSKerberos.class.getCanonicalName());
    JavaSparkContext jsc = new JavaSparkContext(sparkConf);

    // TODO

    System.out.println("Done");

    jsc.stop();
    jsc.close();
  }
}
```
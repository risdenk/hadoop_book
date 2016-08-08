# Spark and Solr
* https://github.com/lucidworks/spark-solr/

Need: https://github.com/lucidworks/spark-solr/issues/79

run.sh
```bash
export SPARK_OPTS="-Djavax.net.ssl.trustStore=./${TRUSTSTORE}.jks -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD} -Djava.security.auth.login.config=./JAAS.conf"

spark-submit \
  --verbose \
  --master yarn \
  --deploy-mode client \
  --files ${TRUSTSTORE}.jks#${TRUSTSTORE}.jks,JAAS.conf#JAAS.conf,v.keytab#v.keytab \
  --driver-java-options "-Djavax.net.ssl.trustStore=./${TRUSTSTORE}.jks -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD} -Djava.security.auth.login.config=./JAAS.conf" \
  --conf "spark.executor.extraJavaOptions=-Djavax.net.ssl.trustStore=./${TRUSTSTORE}.jks -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD} -Djava.security.auth.login.config=./JAAS.conf" \
  --class "SparkSolrKerberos" \
  uber-solr-ssl-kerberos-1.0-SNAPSHOT.jar \
  "${HDFS_PATH}" "zk1.fqdn.com,zk2.fqdn.com,zk3.fqdn.com/solr" "${COLLECTION}"
```

JAAS.conf
```
SolrJClient {
  com.sun.security.auth.module.Krb5LoginModule required
  useTicketCache=false
  useKeyTab=true
  principal="principal@EXAMPLE.COM"
  keyTab="v.keytab"
  renewTicket=true
  storeKey=true;
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

pom.xml
* Need to shade org.apache.http
  * httpclient versions conflict between Spark and Solr
* commons-codec to get base64 encoding for SSL

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.apache.solr</groupId>
    <artifactId>solr-ssl-kerberos</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

        <spark.version>1.6.1</spark.version>
        <spark-solr.version>2.0.4</spark-solr.version>
        <solr.version>5.5.2</solr.version>
        <commons-codec.version>1.10</commons-codec.version>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.scala-tools</groupId>
                <artifactId>maven-scala-plugin</artifactId>
                <version>2.15.2</version>
                <executions>
                    <execution>
                        <id>compile</id>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                        <phase>compile</phase>
                    </execution>

                    <execution>
                        <id>test-compile</id>
                        <goals>
                            <goal>testCompile</goal>
                        </goals>
                        <phase>test-compile</phase>
                    </execution>

                    <execution>
                        <phase>process-resources</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
                    <relocations>
                        <relocation>
                            <pattern>org.apache.http</pattern>
                            <shadedPattern>shaded.org.apache.http</shadedPattern>
                        </relocation>
                    </relocations>
                    <finalName>uber-${project.artifactId}-${project.version}</finalName>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.10</artifactId>
            <version>${spark.version}</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>com.lucidworks.spark</groupId>
            <artifactId>spark-solr</artifactId>
            <version>${spark-solr.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.solr</groupId>
            <artifactId>solr-solrj</artifactId>
            <version>${solr.version}</version>
        </dependency>

        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>${commons-codec.version}</version>
        </dependency>
    </dependencies>
</project>
```

SolrSparkKerberos
```java
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.sql.DataFrame;
import org.apache.spark.sql.SQLContext;
import org.apache.spark.sql.SaveMode;

import java.util.HashMap;
import java.util.Map;

public class SparkSolrKerberos {
  // Spark information
  private static final String appName = SparkSolrKerberos.class.getSimpleName();

  public static void main(String[] args) {
    SparkConf sparkConf = new SparkConf().setAppName(appName);
    JavaSparkContext jsc = new JavaSparkContext(sparkConf);
    SQLContext sqlContext = new SQLContext(jsc);

    String file = args[0];
    String zkHost  = args[1];
    String collection = args[2];

    DataFrame df = sqlContext.read().format("json").load(file);

    Map<String, String> options = new HashMap<>();
    options.put("zkhost", zkHost);
    options.put("collection", collection);
    //options.put("gen_uniq_key", "true"); // Generate unique key if the 'id' field does not exist

    // Write to Solr
    df.write().format("solr").options(options).mode(SaveMode.Overwrite).save();

    jsc.stop();
    jsc.close();
  }
}
```
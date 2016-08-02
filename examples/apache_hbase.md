# Apache HBase
## CLI
```bash
kinit

hbase shell
```

## Java
run.sh
```bash
kinit

java -cp $(hbase classpath):uber-hbase-example-1.0-SNAPSHOT.jar HBaseTest
```

HBaseTest
```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.Admin;
import org.apache.hadoop.hbase.client.Connection;
import org.apache.hadoop.hbase.client.ConnectionFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HBaseTest {
  private final static Logger logger = LoggerFactory.getLogger(HBaseTest.class);

  public static void main(String[] args) throws Exception {
    Configuration config = HBaseConfiguration.create();

    try (Connection connection = ConnectionFactory.createConnection(config);
         Admin admin = connection.getAdmin()) {

      for(TableName tableName : admin.listTableNames()) {
        logger.info(tableName.getNameAsString());
      }
    }
  }
}
```
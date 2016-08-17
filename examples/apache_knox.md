# Apache Knox
## Knox DSL
### HBase
```
import org.apache.hadoop.gateway.shell.Hadoop;
import org.apache.hadoop.gateway.shell.hbase.HBase;

public class KnoxDSL {
  public static void main(String[] args) throws Exception {

    // Gateway must include the /hbase
    // ie: https://localhost:8442/gateway/default/hbase"
    String gateway = args[0];
    String username = args[1];
    String password = args[2];
    String table = args[3];
    String rowkey = args[4];

    Hadoop session = Hadoop.login(gateway, username, password);

    System.out.println("System version : " + HBase.session(session).systemVersion().now().getString());
    System.out.println("Cluster version : " + HBase.session(session).clusterVersion().now().getString());
    System.out.println("Status : " + HBase.session(session).status().now().getString());

    System.out.println(HBase.session(session).table(table).row(rowkey).query().now().getString());

    session.shutdown();
  }
}
```
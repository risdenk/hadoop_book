# Apache Solr
## Java - SSL & Kerberos
run.sh
```bash
kinit

java \
  -cp uber-solr-ssl-kerberos-1.0-SNAPSHOT.jar \
  -Djavax.net.ssl.trustStore=./solr.jks -Djavax.net.ssl.trustStorePassword=changeit \
  -Djava.security.auth.login.config=./solr-client-jaas.conf \
  SolrSSLKerberosExample \
  "zk1.fqdn.com,zk2.fqdn.com,zk3.fqdn.com/solr" COLLECTION
```

solr-client-jaas.conf
```
SolrJClient {
  com.sun.security.auth.module.Krb5LoginModule required
  useTicketCache=true;
};
Client {
  com.sun.security.auth.module.Krb5LoginModule required
  useTicketCache=true
  serviceName="zookeeper";
};
```

SolrSSLKerberosExample
```java
import org.apache.solr.client.solrj.impl.CloudSolrClient;
import org.apache.solr.client.solrj.impl.HttpClientUtil;
import org.apache.solr.client.solrj.impl.Krb5HttpClientConfigurer;
import org.apache.solr.common.SolrInputDocument;

public class SolrSSLKerberosExample {
  public static void main(String[] args) throws Exception {
    String zkHost = args[0];
    String collection = args[1];

    // Setup Krb5HttpClientConfigurator must be done before first SolrClient is instantiated
    if (System.getProperty(Krb5HttpClientConfigurer.LOGIN_CONFIG_PROP) != null) {
      HttpClientUtil.setConfigurer(new Krb5HttpClientConfigurer());
    }

    // Connect to Solr
    try(CloudSolrClient client = new CloudSolrClient(zkHost)) {
      SolrInputDocument doc = new SolrInputDocument();
      doc.addField("id", "1234");
      doc.addField("name", "A lovely summer holiday");
      client.add(collection, doc);
      client.commit(collection);
    }
  }
}
```


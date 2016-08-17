# Storm and Solr
run.sh
```bash
storm jar \
  uber-storm-kafka-solr-example-1.0-SNAPSHOT.jar \
  StormSolrExample \
  "zk1.fqdn.com,zk2.fqdn.com,zk3.fqdn.com/solr" \
  ${COLLECTION} \
  ${TRUSTSTORE} \
  ${TRUSTSTORE_PASSWORD}
```

StormSolrExample
```java
import backtype.storm.Config;
import backtype.storm.StormSubmitter;
import backtype.storm.topology.IRichSpout;
import backtype.storm.topology.TopologyBuilder;

import java.util.Properties;

public class StormSolrExample {
  public static void main(String[] args) throws Exception {
    String zkHost = args[0];
    String collection = args[1];
    String truststore = args[2];
    String truststorePassword = args[3];

    TopologyBuilder builder = new TopologyBuilder();

    IRichSpout spout = new MySpout();
    builder.setSpout("spout", spout);

    SolrBolt bolt = new SolrBolt();
    builder.setBolt("forwardToSolr", bolt).shuffleGrouping("spout");

    Properties props = new Properties();
    props.put(SolrBolt.SOLR_ZKHOST, zkHost);
    props.put(SolrBolt.SOLR_COLLECTION, collection);
    props.put(SolrBolt.SOLR_TRUSTSTORE, truststore);
    props.put(SolrBolt.SOLR_TRUSTSTORE_PASSWORD, truststorePassword);

    Config conf = new Config();
    conf.put(SolrBolt.SOLR_PROPERTIES, props);

    StormSubmitter.submitTopology("solrBoltTest", conf, builder.createTopology());
  }
}
```

SolrBolt
```java
import backtype.storm.task.OutputCollector;
import backtype.storm.task.TopologyContext;
import backtype.storm.topology.OutputFieldsDeclarer;
import backtype.storm.topology.base.BaseRichBolt;
import backtype.storm.tuple.Tuple;
import backtype.storm.utils.TupleUtils;
import org.apache.solr.client.solrj.impl.CloudSolrClient;
import org.apache.solr.client.solrj.impl.HttpClientUtil;
import org.apache.solr.client.solrj.impl.Krb5HttpClientConfigurer;
import org.apache.solr.common.SolrInputDocument;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;
import java.util.Map;

public class SolrBolt extends BaseRichBolt {

  private static final Logger LOG = LoggerFactory.getLogger(SolrBolt.class);

  public static final String SOLR_PROPERTIES = "solr.properties";
  public static final String SOLR_TRUSTSTORE = "solr.truststore";
  public static final String SOLR_TRUSTSTORE_PASSWORD = "solr.truststore.password";
  public static final String SOLR_COLLECTION = "solr.collection";
  public static final String SOLR_ZKHOST = "solr.zkhost";

  private OutputCollector collector;
  private String collection;
  private CloudSolrClient cloudSolrClient;

  @Override
  public void prepare(Map stormConf, TopologyContext context, OutputCollector collector) {
    Map configMap = (Map) stormConf.get(SOLR_PROPERTIES);

    // Setup SSL trustStore on the Storm worker
    System.setProperty("javax.net.ssl.trustStore", String.valueOf(configMap.get(SOLR_TRUSTSTORE)));
    System.setProperty("javax.net.ssl.trustStorePassword", String.valueOf(configMap.get(SOLR_TRUSTSTORE_PASSWORD)));

    this.collection = String.valueOf(configMap.get(SOLR_COLLECTION));
    this.collector = collector;

    // Setup Krb5HttpClientConfigurator must be done before first SolrClient is instantiated
    if (System.getProperty(Krb5HttpClientConfigurer.LOGIN_CONFIG_PROP) != null) {
      HttpClientUtil.setConfigurer(new Krb5HttpClientConfigurer());
    }

    this.cloudSolrClient = new CloudSolrClient(String.valueOf(configMap.get(SOLR_ZKHOST)));
  }

  @Override
  public void execute(Tuple input) {
    if (TupleUtils.isTick(input)) {
      collector.ack(input);
      return; // Do not try to send ticks to Solr
    }

    try {
      SolrInputDocument doc = new SolrInputDocument();
      for(String field : input.getFields().toList()) {
        doc.addField(field, input.getValueByField(field));
      }

      this.cloudSolrClient.add(this.collection, doc);

      collector.ack(input);
    } catch (Exception ex) {
      collector.reportError(ex);
      collector.fail(input);
    }
  }

  @Override
  public void declareOutputFields(OutputFieldsDeclarer declarer) {

  }

  @Override
  public void cleanup() {
    super.cleanup();

    if(this.cloudSolrClient != null) {
      try {
        this.cloudSolrClient.close();
      } catch (IOException e) {
        LOG.error("Error closing CloudSolrClient", e);
      }
    }
  }
}
```
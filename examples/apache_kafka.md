# Apache Kafka
## Kafka Console Consumer
```bash
/usr/hdp/current/kafka-broker/bin/kafka-console-consumer.sh --zookeeper zk1.fqdn.com,zk2.fqdn.com,zk3.fqdn.com --topic $TOPIC --security-protocol PLAINTEXTSASL ...
```

## Kafka Console Producer
```bash
/usr/hdp/current/kafka-broker/bin/kafka-console-producer.sh --broker-list kafka1.fqdn.com,kafka2.fqdn.com,kafka3.fqdn.com --topic $TOPIC --security-protocol PLAINTEXTSASL ...
```
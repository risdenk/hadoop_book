# Apache Hadoop - MapReduce

## Specifying Truststore
```bash
hadoop jar \
  -Djavax.net.ssl.trustStore=${TRUSTSTORE_PATH}.jks \
  -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD} \
  -files ${TRUSTSTORE_PATH}.jks \
  -Dmapreduce.map.java.opts="-Xmx3575m -Djavax.net.ssl.trustStore=${TRUSTSTORE_PATH}.jks -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD}" \
  -Dmapreduce.reduce.java.opts="-Xmx3575m -Djavax.net.ssl.trustStore=${TRUSTSTORE_PATH}.jks -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD}" \
  ...
```
# Apache Hadoop - MapReduce

## Specifying Truststore
```bash
hadoop jar \
  -Djavax.net.ssl.trustStore=${TRUSTSTORE_PATH}.jks \
  -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD} \
  -files ${TRUSTSTORE_PATH}.jks \
  -Dmapreduce.map.java.opts="$EXISTING_MAP_JAVA_OPTS -Djavax.net.ssl.trustStore=${TRUSTSTORE_PATH}.jks -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD}" \
  -Dmapreduce.reduce.java.opts="$EXISTING_REDUCE_JAVA_OPTS -Djavax.net.ssl.trustStore=${TRUSTSTORE_PATH}.jks -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD}" \
  ...
```
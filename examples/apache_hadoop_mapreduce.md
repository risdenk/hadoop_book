# Apache Hadoop - MapReduce

## Specifying Truststore
```bash
hadoop jar \
  -files ${TRUSTSTORE_PATH}.jks \
  -Djavax.net.ssl.trustStore=${TRUSTSTORE_PATH}.jks \
  -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD}
```
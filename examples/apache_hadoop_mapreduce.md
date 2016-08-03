# Apache Hadoop - MapReduce

## Specifying Truststore
There are 3 JVMs for which the truststore must be specified:
* client
* map
* reduce

For the client JVM, this is accomplished with standard Java properties (`-D`). Map and reduce JVMs run on different machines in the cluster. The `-files` argument ensures that the JKS is copied out to the proper cluster nodes. The `mapreduce.[map|reduce].java.opts` argument specifies the truststore properties on the map and reduce JVMs.

Example:
```bash
hadoop jar \
  -Djavax.net.ssl.trustStore=${TRUSTSTORE_PATH}.jks \
  -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD} \
  -files ${TRUSTSTORE_PATH}.jks \
  -Dmapreduce.map.java.opts="$EXISTING_MAP_JAVA_OPTS -Djavax.net.ssl.trustStore=${TRUSTSTORE_PATH}.jks -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD}" \
  -Dmapreduce.reduce.java.opts="$EXISTING_REDUCE_JAVA_OPTS -Djavax.net.ssl.trustStore=${TRUSTSTORE_PATH}.jks -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD}" \
  ...
```
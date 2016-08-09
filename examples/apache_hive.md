# Apache Hive
## Beeline CLI
### Transport Mode = http
If transport mode is http then need to add the following to the connection string:

```
;transportMode=http;httpPath=cliservice;
```

### No Kerberos or SSL
```bash
beeline -u 'jdbc:hive2://${BEELINE_HOST}:${BEELINE_PORT}/'
```

### Kerberos Only
```bash
kinit

beeline -u 'jdbc:hive2://${BEELINE_HOST}:${BEELINE_PORT}/;principal=hive/_HOST@EXAMPLE.COM'
```

### SSL Only
```bash
beeline -u 'jdbc:hive2://${BEELINE_HOST}:${BEELINE_PORT}/;ssl=true;sslTrustStore=${TRUSTSTORE}.jks;trustStorePassword=${TRUSTSTORE_PASSWORD}'
```

### Kerberos and SSL
```bash
kinit

beeline -u 'jdbc:hive2://${BEELINE_HOST}:${BEELINE_PORT}/;principal=hive/_HOST@EXAMPLE.COM;ssl=true;sslTrustStore=${TRUSTSTORE}.jks;trustStorePassword=${TRUSTSTORE_PASSWORD}'
```
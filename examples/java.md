# Java
## SSL
Unsigned Certificate

Generate Truststore
```bash
openssl s_client -showcerts -connect ${SSLHOST}:${SSLPORT} </dev/null 2>/dev/null > ${SSLHOST}.crt
keytool -import -keystore ${TRUSTSTORE_PATH}.jks -file ${SSLHOST}.crt -storepass changeit -alias $SSLHOST -noprompt
rm ${SSLHOST}.crt
```

Use Truststore
```bash
java \
  ... \
  -Djavax.net.ssl.trustStore=${TRUSTSTORE_PATH}.jks \
  -Djavax.net.ssl.trustStorePassword=${TRUSTSTORE_PASSWORD} \
  ... \
  MainClass \
  Args...
```

## Kerberos
```bash
java \
  ... \
  -Djava.security.auth.login.config=PATH_TO_JAAS.conf  \
  ... \
  MainClass \
  Args...
```
### JAAS
* https://docs.oracle.com/javase/8/docs/technotes/guides/security/jaas/JAASRefGuide.html
* https://steveloughran.gitbooks.io/kerberos_and_hadoop/content/sections/jaas.html
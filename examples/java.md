# Java
## SSL
Unsigned Certificate
```bash
java \
  ... \
  -Djavax.net.ssl.trustStore=PATH_TO_TRUSTSTORE.jks \
  -Djavax.net.ssl.trustStorePassword=TRUSTSTORE_PASSWORD \
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
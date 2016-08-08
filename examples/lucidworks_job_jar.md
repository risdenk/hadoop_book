# Lucidworks Job Jar

## SSL and Kerberos
```
hadoop jar
-files ./TRUSTSTORE.jks,./JAAS.conf,./EXAMPLE.keytab
-Dlww.truststore=./TRUSTSTORE.jks
-Dlww.truststore.password=TRUSTSTORE_PASSWORD
-Dlww.jaas.file=./JAAS.conf
-Dlww.jaas.appname=SolrJClient
...
```

JAAS.conf
```
SolrJClient {
  com.sun.security.auth.module.Krb5LoginModule required
  useKeyTab=true
  keyTab="./EXAMPLE.keytab"
  storeKey=true
  useTicketCache=false
  principal="principal@EXAMPLE.COM";
};
Client {
  com.sun.security.auth.module.Krb5LoginModule required
  useKeyTab=true
  keyTab="./EXAMPLE.keytab"
  storeKey=true
  useTicketCache=false
  principal="principal@EXAMPLE.COM";
  serviceName="zookeeper";
};
```
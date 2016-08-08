# Kerberos Debugging

```bash
KRB5_TRACE=/dev/stdout kinit -V
HADOOP_OPTS="-Dsun.security.krb5.debug=true" #-Dsun.security.spnego.debug=true"
HADOOP_ROOT_LOGGER=DEBUG,console hdfs ...
hdfs groups USERNAME
hadoop org.apache.hadoop.security.HadoopKerberosName USER_PRINCIPAL
hadoop org.apache.hadoop.security.UserGroupInformation
```
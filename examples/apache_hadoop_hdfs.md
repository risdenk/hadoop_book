# Apache Hadoop - HDFS
## CLI
```bash
kinit

hdfs dfs -ls
```
## Java
run.sh
```bash
kinit

java -cp $(hdfs classpath):uber-hdfs-example-1.0-SNAPSHOT.jar HDFSTest /user/$(whoami)
```

HDFSTest.java
```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.*;

public class HDFSTest {
  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    FileSystem fs = FileSystem.get(conf);

    RemoteIterator<LocatedFileStatus> fileStatusListIterator = fs.listFiles(new Path(args[0]), true);
    while(fileStatusListIterator.hasNext()){
      LocatedFileStatus fileStatus = fileStatusListIterator.next();
      System.out.println(fileStatus.getPath());
    }
  }
```
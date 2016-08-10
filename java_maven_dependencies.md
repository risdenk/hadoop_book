# Java - Maven Dependencies

## Hortonworks
* https://community.hortonworks.com/articles/43727/hortonworks-data-platform-development-guide.html
* http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.1.15/bk_user-guide/content/user-guide-setup-maven-repo.html
* https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.4.2/bk_installing_manually_book/content/ch01s13.html
* https://community.hortonworks.com/questions/626/how-do-i-resolve-maven-dependencies-for-building-a.html

### Browsing the repository
* http://repo.hortonworks.com/index.html#view-repositories

### pom.xml excerpt
```xml
    <repositories>
        <repository>
            <releases>
                <enabled>true</enabled>
                <updatePolicy>always</updatePolicy>
                <checksumPolicy>warn</checksumPolicy>
            </releases>
            <snapshots>
                <enabled>false</enabled>
                <updatePolicy>never</updatePolicy>
                <checksumPolicy>fail</checksumPolicy>
            </snapshots>
            <id>HDPReleases</id>
            <name>HDP Releases</name>
            <url>http://repo.hortonworks.com/content/repositories/releases/</url>
            <layout>default</layout>
        </repository>
    </repositories>
```

## Cloudera
* https://www.cloudera.com/documentation/enterprise/release-notes/topics/cdh_vd_cdh5_maven_repo.html

### Browsing the repository
* https://repository.cloudera.com/artifactory/cloudera-repos/

### pom.xml excerpt
```xml
  <repositories>
    <repository>
      <id>cloudera</id>
      <url>https://repository.cloudera.com/artifactory/cloudera-repos/</url>
    </repository>
  </repositories>
```
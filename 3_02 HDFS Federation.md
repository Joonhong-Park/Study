# HDFS Federation

다른 클러스터에 있는 HDFS에 연결해보자

여기에선 CDP HDFS에서 HDP HDFS시스템으로 접근함



#### Hosts 파일 수정

경로 : `C:\Windows\System32\drivers\etc`

추가할 NameNode의 IP, 호스트명을 추가

#### core-site.xml 수정

`dfs.nameservices`에 네임노드를 추가하고 (여기에선 `ernn`) 여기에 해당되는 Property들을 추가해줌

> NameNode 명에는 `_`를 사용할 수 없음

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://nn</value>
    </property>
    <property>
        <name>dfs.nameservices</name>
        <value>nn, ernn</value>
    </property>
    <!-- my nn -->
    <property>
        <name>dfs.ha.namenodes.nn</name>
        <value>nn1,nn2</value>
    </property>
    <property>
        <name>dfs.namenode.rpc-address.nn.nn1</name>
        <value>hdfs://hdm1.cdp.jh.io:8020</value>
    </property>
    <property>
        <name>dfs.namenode.rpc-address.nn.nn2</name>
        <value>hdfs://hdm2.cdp.jh.io:8020</value>
    </property>
    <property>
        <name>dfs.client.failover.proxy.provider.nn</name>
        <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>
    
    <!----------------------- er nn 추가 ----------------------->
    <property>
        <name>dfs.ha.namenodes.ernn</name>
        <value>nn1,nn2</value>
    </property>
    <property>
        <name>dfs.namenode.http-address.ernn.nn1</name>
        <value>hdfs://tt05nn001.hdp.local:50070</value>
    </property>
    <property>
        <name>dfs.namenode.http-address.ernn.nn2</name>
        <value>hdfs://tt05nn002.hdp.local:50070</value>
    </property>
    <property>
        <name>dfs.namenode.rpc-address.ernn.nn1</name>
        <value>hdfs://tt05nn001.hdp.local:8020</value>
    </property>
    <property>
        <name>dfs.namenode.rpc-address.ernn.nn2</name>
        <value>hdfs://tt05nn002.hdp.local:8020</value>
    </property>
    <property>
        <name>dfs.client.failover.proxy.provider.ernn</name>
        <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>
</configuration>
```



#### Filesystem 객체가 NameNode명을 가져오도록 작성

```java
/** Configuration setting & get FileSystem
     * @return
     * @throws IOException
     * @throws InterruptedException
     */
    public static FileSystem getFileSystem(String nn) throws IOException, InterruptedException {

        Configuration conf = new Configuration();
        
        // core-site.xml 의 "fs.defaultFS" 값을 연결하고자하는 네임노드값으로 변경
        conf.set("fs.defaultFS", "hdfs://" + nn);
        FileSystem fs = FileSystem.get(conf);

        return fs;
    }
```



#### NameNode명을 명시하여 사용

```java
public static void main(String[] args) throws Exception{
        UserGroupInformation ugi = UserGroupInformation.createRemoteUser("impala");
        ugi.doAs((PrivilegedAction<Integer>) () -> {
            try {
                FileSystem filesystem = getFileSystem("ernn");

                ls(filesystem, "/");

            } catch (IOException | InterruptedException e) {
                e.printStackTrace();
            }
            return 0;
        });
    }
```


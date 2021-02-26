# HDFS JAVA API

HDFS dfs 의 명령어를 구현하는 java api 작성



개발툴(Intellij 또는 Eclipse) > Maven Proejct 생성 > pom.xml 수정



#### pom.xml

##### `hadoop-hdfs`, `hadoop-common`, `hadoop-client` Dependncy 추가

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>hdfs_practice</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>
    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-hdfs -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>3.1.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-common -->
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>3.1.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>3.1.1</version>
        </dependency>
    </dependencies>
</project>
```



#### HDFS 파일시스템 연결

- ##### Configuration 객체 생성하여 연결하는 방법

  ```java
  public static FileSystem getFileSystem() throws IOException {
  	Configuration conf = new Configuration(false);
  	String uri = "hdfs://hdm1.cdp.jh.io";
      FileSystem fs = FileSystem.get(URI.create(uri), conf, "username");
      
      return fs;
      }
  ```

- ##### HDFS HA 설정되어있는 경우

  ```java
  public static FileSystem getFileSystem() throws IOException {
  	Configuration conf = new Configuration(false);
  	String nameservices = "nn";
      String[] namenodesAddr = {"hdm1.cdp.jh.io:8020", "hdm2.cdp.jh.io:8020"};
  	String[] namenodes = {"nn1", "nn2"};
      
  	conf.set("fs.defaultFS", "hdfs://" + nameservices);
  	conf.set("fs.default.name", conf.get("fs.defaultFS"));
  	conf.set("dfs.nameservices", nameservices);
  	conf.set("dfs.ha.namenodes." + nameservices, namenodes[0]+","+namenodes[1]);
  	conf.set("dfs.namenode.rpc-address." + nameservices + "." + namenodes[0], namenodesAddr[0]);
  	conf.set("dfs.namenode.rpc-address." + nameservices + "." + namenodes[1], namenodesAddr[1]);
  	conf.set("dfs.client.failover.proxy.provider." + nameservices,"org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider");
      
  	FileSystem fs = FileSystem.get(conf);
      
      return fs;
      }
  ```

- ##### core-site.xml 작성하여 연결

  ###### resources > core-site.xml 생성

  ```xml
  <configuration>
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://nn</value>
      </property>
      <property>
          <name>dfs.nameservices</name>
          <value>nn</value>
      </property>
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
  </configuration>
  ```

  ```java
  public static FileSystem getFileSystem() throws IOException {
  	Configuration conf = new Configuration();
  	FileSystem fs = FileSystem.get(conf);
  
          return fs;
      }
  ```

  파일명은 반드시 `core-site.xml`이여야 한다.

  다른 파일명을 사용하고 싶은 경우 Configuration 객체에 addResource로 해당 파일명을 추가해주면 됨

  ```
  conf.addResource("hdfs-site.xml");
  ```

  

- ##### 위에서 방법 사용시 username을 명시해주지 않아 windows username으로 접속을 시도하게 되어 권한이 없는 폴더에 접근하거나 write 시도시 Permission dined가 되는 문제가 발생

  이럴 경우 `UserGroupInformation`을 사용하여 유저명을 직접 명시해주면 됨

  하지만 바람직한 사용방법은 아니므로 개인이 사용할 경우에만 활용할 것

  ```java
  public static void main(String[] args) throws Exception{
          UserGroupInformation ugi = UserGroupInformation.createRemoteUser("impala");
          ugi.doAs((PrivilegedAction<Integer>) () -> {
              try {
                  FileSystem filesystem = getFileSystem("ernn");
  //                ls_pattern(filesystem, new Path("/????"));
                    ls(filesystem, "/");
  //                makedir(filesystem, "/user/impala/test2");
  //				  put(filesystem, "C:/Users/joonhong/Documents/ipaddr.txt", "/tmp/");
              } catch (IOException | InterruptedException e) {
                  e.printStackTrace();
              }
              return 0;
          });
      }
  ```

  HDFS 파일 권한을 777로 변경해주면 누구든지 쓰고 읽기가 가능해지나 보안에 매우 취약해지므로 절대 쓰지 말것

- 가장 바람직한 방법은 java 실행파일로 만들어 리눅스에 넣은 뒤, 해당 계정으로 로그인하여 사용하는 것이라 함



#### 해당 경로의 파일 목록 확인 (`hdfs dfs -ls /`)

```java
 	/**
     * 해당 경로 파일 목록 확인
     * @param fs getFileSystem으로 가져온 filesystem객체
     * @param input_path 확인할 디렉토리 경로
     * @throws IOException
     */
    public static void ls(FileSystem fs, String input_path) throws IOException{

        Path path = new Path(input_path);
        FileStatus[] files = fs.listStatus(path);
        for(FileStatus file : files){
            System.out.print(file.getPermission()+" ");
            // 심볼릭 링크 수를 출력해줄 코드 필요
            System.out.print("-"+file.getOwner()+" ");
            System.out.print(file.getGroup()+" ");
            System.out.print(file.getBlockSize()+" ");
            System.out.print(file.getModificationTime()+" "); // 시간 출력 방법 변경 필요
            System.out.println("/"+file.getPath().getName()+" ");
        }
    }
```



#### 정규식 패턴을 읽어 해당 경로의 파일 목록 확인 (`hdfs dfs -ls /*`)

```java
	/**
     * 패턴을 사용한 해당 경로 파일 목록 확인
     * @param fs getFileSystem으로 가져온 filesystem객체
     * @param path 패턴이 적용된 경로
     * @throws IOException
     */
    public static void ls_pattern(FileSystem fs, Path path) throws IOException{
        FileStatus[] fileStatus = fs.globStatus(path);
        for (FileStatus file : fileStatus){
            System.out.println(file);
        }
    }
```



#### 해당 경로 디렉토리 생성 (`hdfs dfs -mkdir <dirsrc>`)

```java
	/**
     * 해당 경로 디렉토리 생성
     * @param fs getFileSystem으로 가져온 filesystem객체
     * @param path 생성할 디렉토리 경로
     * @throws IOException
     */
    public static void makedir(FileSystem fs, String path) throws IOException{
        fs.mkdirs(new Path(path));
    }
```



#### 로컬 파일을 HDFS 디렉토리로 복사 (`hdfs dfs -put <src> <dirsrc>`)

```java
	/**
     * 파일 복사
     * @param fs getFileSystem으로 가져온 filesystem객체
     * @param file_path 복사할 파일 경로
     * @param dir_path 붙여넣을 디렉토리 경로
     * @throws IOException
     */
    public static void put(FileSystem fs, String file_path, String dir_path) throws IOException{
        fs.copyFromLocalFile(new Path(file_path), new Path(dir_path));
    }
```



#### HDFS 파일을 로컬 디렉토리로 복사(`dhfs dfs -cp <src> <dirsrc>`)

```java
	/**
     * 
     * @param fs getFileSystem으로 가져온 filesystem객체
     * @param file_path 복사할 파일 경로
     * @param dir_path 붙여넣을 로컬 디렉토리 경로
     * @throws IOException
     */
    public static void cp(FileSystem fs, String file_path, String dir_path) throws IOException{
        fs.copyToLocalFile(new Path(file_path), new Path(dir_path));

    }
```



#### HDFS에서 파일을 읽어 내용 출력 (`hdfs dfs -cat <filepath>`)

```java
	/**
     * hdfs 파일을 읽어 내용 출력
     * @param fs getFileSystem으로 가져온 filesystem객체
     * @param file_path 읽을 파일 경로
     * @throws IOException
     */
    public static void cat(FileSystem fs, String file_path) throws IOException{
        FSDataInputStream is = fs.open(new Path(file_path));
        IOUtils.copyBytes(is, System.out, 4096, false);
        IOUtils.closeStream(is);

    }
```




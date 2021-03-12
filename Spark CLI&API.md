# Spark CLI/API

## Spark로 csv 파일 읽기



#### Local에 저장 후 hdfs에 저장

```
hdsf dfs -mkdir /user/root/flightdata

cd ~/filghtdata

hdfs dfs -put * /user/root/flightdata

hdfs dfs -ls /user/root/flightdata
```



### 1. Spark-shell 에서 CSV 읽기



#### spark-shell 실행

```
spark-shell
```



#### CSV File 읽기

> Scala에서는 따로 옵션을 주지않으면 Default로 DataFrame 형태로 저장함

```
val flightdata = spark.read.option("inferSchema", true).option("header",true).csv("/user/root/flightdata/*")
```



- ##### inferSchema 옵션을 사용하지않고 직접 Schema를 명시해주면 Read 속도가 향상됨

  ```
  import org.apache.spark.sql.types._
   
  val customSchema = StructType(Array(StructField("DEST_COUNTRY_NAME",StringType,true),
     StructField("ORIGIN_COUNTRY_NAME",StringType,true),
     StructField("count",IntegerType,true)))
   
  val flightdata = spark.read.format("csv").option("header", true).schema(customSchema).load("/user/root/flightdata/*")
  ```

  

#### 결과 출력

```
flightdata.show(10)
```

![](./image/sparkshell.PNG)



- ##### Delimiter가 Comma가 아닌 파일 읽기

  delimiter 옵션만 추가해주면 됨

  ```
  val flightdata = spark.read
  	.option("inferSchema",true)
  	.option("header",true)
  	.option("delimiter","\t")
  	.csv("hdfs:///user/root/flightdata/*")
  ```

  

### 2. jar 파일로 실행



#### Intellij > Maven Project 생성



#### pom.xml 수정

> Hadoop API는 Spark API 에 포함되어 다운되는 것으로 보임
>
> 혹시나 안된다면 Hadoop API도 추가해 볼 것

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>SparkByJava</artifactId>
    <version>1.0-SNAPSHOT</version>
  
    <dependencies>

        <!-- ============= -->
        <!--  Spark API  -->
        <!-- ============= -->

        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.11</artifactId>
            <version>2.4.0.7.1.4.0-203</version>
        </dependency>

        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.11</artifactId>
            <version>2.4.0.7.1.4.0-203</version>
        </dependency>
        
        <!-- ============= -->
        <!--  Hadoop API  -->
        <!-- ============= -->

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>3.1.1.7.1.4.0-203</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-hdfs</artifactId>
            <version>3.1.1.7.1.4.0-203</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-mapreduce-client-jobclient</artifactId>
            <version>3.1.1.7.1.4.0-203</version>
        </dependency>

    </dependencies>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>

    <repositories>
        <repository>
            <id>cloudera-public</id>
            <url>https://repository.cloudera.com/artifactory/public/</url>
        </repository>
    </repositories>

</project>
```



- ##### jar파일을 빌드하면서 옮기는 방법

아래의 코드를 자신의 환경에 맞게 수정하여 pom.xml에 추가

```
	<build>
        <plugins>
            <plugin>
                <artifactId>maven-antrun-plugin</artifactId>
                <executions>
                    <execution>
                        <id>scp</id>
                        <phase>install</phase>
                        <goals>
                            <goal>run</goal>
                        </goals>
                        <configuration>
                            <tasks>
                                <scp todir="root:비밀번호@Spark실행할 네임노드IP:~/jar/spark" trust="true" failonerror="false">
                                    <fileset dir="${basedir}/target">
                                        <include name="${project.build.finalName}*.jar"/>
                                    </fileset>
                                </scp>
                                <sshexec host="Spark실행할 네임노드IP" username="root" trust="true" failonerror="false" password="비밀번호"
                                         command="mv ~/jar/spark/SparkByJava-1.0-SNAPSHOT.jar ~/jar/spark/sbj.jar">
                                </sshexec>
                            </tasks>
                        </configuration>
                    </execution>
                </executions>
                <dependencies>
                    <dependency>
                        <groupId>ant</groupId>
                        <artifactId>ant-jsch</artifactId>
                        <version>1.6.5</version>
                    </dependency>
                    <dependency>
                        <groupId>com.jcraft</groupId>
                        <artifactId>jsch</artifactId>
                        <version>0.1.42</version>
                    </dependency>
                </dependencies>
            </plugin>
        </plugins>
    </build>
```



#### resources에 core-site.xml 추가

HDFS HA가 설정되어 있으므로 아래와 같이 core-site.xml을 추가해주었음

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://nn</value>
    </property>
    <property>
        <name>dfs.nameservices</name>
        <value>nn</value>
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
</configuration>
```



#### CSV File을 DataFrame 형태로 읽은 뒤 출력 및 저장

저장 시 따로 format을 주지않으면 Default인 Parquet파일로 저장됨

```
import org.apache.spark.sql.DataFrameWriter;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

public class ReadCSV {

    public static void main(String[] args) throws Exception {

        if (args.length < 1) {
            System.err.println("Usage : ReadCSV <input file path>");
            System.exit(1);
        }

        SparkSession spark = SparkSession.builder()
                .appName("Java Spark Read CSV")
                .master("yarn")
                .getOrCreate();

        CSVtoDataFrame(spark, args);
    }

    private static void CSVtoDataFrame(SparkSession spark, String[] args) throws Exception {

        Dataset<Row> df = spark.read()
                .option("header", "true")
                .option("inferSchema", "true")
                .csv(args[0]);

        df.show();
        df.printSchema();
        DataFrameWriter<Row> write = df.write();
        write.save("spark_output");
    }
}
```

SparkSession 객체에는 `.config("key","value")`를 이용하여 설정들을 추가할 수 있음



자세한 Spark Configuration Option은 아래의 공식문서를 참조

https://spark.apache.org/docs/latest/configuration.html



#### Jar파일 생성 및 실행

위에서 maven-antrun-plugin을 설정하였으면 Maven > install 하면 설정한 경로에 Jar파일이 생성됨

설정하지 않았다면 Maven > pakage 후 target 디렉토리에 생성된 jar파일을 Spark가 실행될 서버에 저장



jar파일이 있는 디렉토리로 이동 후 spark-submit으로 jar파일 실행

```
cd ~/jar/spark
spark-submit --class ReadCSV sbj.jar /user/root/flightdata/*
```



show(), printschema() 결과

![](./image/show.PNG)



write().save() 결과

![](./image/spark_output.PNG)



------

![](./image/spark_his.png)



## RDD

RDD (Resilient Distributed Dataset) 

Spark에서 사용되는 가장 기본적인 데이터 객체



#### RDD를 제어하는 연산 종류

1. Transformation : 외부 데이터 또는 RDD에서 연산 후 새로운 RDD를 생성하는 작업
2. Action : 궁극적으로 원하는 결과물(연산 결과가 RDD가 아닌 다른 타입인 경우)이 출력되는 작업



#### Lazy-Evalution (지연처리)

Transformation 연산을 수행해도 실제로 데이터는 처리되지 않고 계산을 위한 수행 계획(RDD)이 생성됨

실제 계산은 Action 연산을 호출하는 시점에서 수행되어 작업의 효율성을 높임

![](./image/rdd.PNG)

> RDD는 Action이 호출될 때마다 새로운 연산을 처리함
>
> 이때, persist() 메소드를 사용하면 처리 결과를 유지, 재사용할 수 있음





###### spark shell로 RDD 생성?

###### jar 파일로 RDD 생성?



DataFrame = Dataset[Row]

RDD -> DS, DF 변환 가능



## DataFrame

###### 개념설명?



## Row

개념설명?



## Dataset

개념설명?



## Hadoop InputFormat 사용

sparkconf = sc

sc.hadoopRdd()

https://knight76.tistory.com/entry/Spark-HadoopRDD



## Spark SQL 사용

SQL을 이용하여 RDD, DataSet, DataFrame 작업을 생성하고 처리

- UDF 만들어서 실행

  UDF : 사용자정의함수

  

 

## Json 파일 사용

- org.apache.spark.sql.execution.datasources.json.JsonFileFormat

## Parquet 파일 사용

- org.apache.spark.sql.execution.datasources.parquet.ParquetFileFormat

## Orc 파일 사용

- org.apache.spark.sql.execution.datasources.orc.OrcFileFormat
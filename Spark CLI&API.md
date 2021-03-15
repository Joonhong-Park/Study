# Spark CLI/API

## Spark로 csv 파일 읽기



#### Local에 저장 후 hdfs에 저장

```
hdsf dfs -mkdir /user/root/flightdata

cd ~/filghtdata

hdfs dfs -put * /user/root/flightdata

hdfs dfs -ls /user/root/flightdata
```



### 1.Spark-shell 에서 CSV 읽기



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

  ![](./image/format.PNG)
  
  CSV파일 외에도 위의 Format의 파일은 모두 읽을 수 있으며, format을 이용하여 직접 만들 수도 있음
  
  

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

Spark Core에서는 RDD는 변경할 수 없는(Immutable) 데이터의 집합으로 클러스터 내의 여러 노드에 분산되어 있음



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



#### RDD를 사용하는 경우

- low-level API인 Transformation, Action을 사용할 때

- 데이터가 미디어와 같이 비구조화 형태로 되어 있을 때

- 데이터를 함수형 프로그래밍으로 조작하고 싶을 때

- 데이터를 처리할 때, 칼럼 형식과 같은 스키마를 굳이 따지고 싶지 않을 때

- 최적화를 굳이 신경쓰지 않을 때



#### RDD 객체 생성

RDD는 SparkContext 객체를 이용하여 생성할 수 있음



- ##### Spark-shell 에서 생성

  ###### spark-shell 실행

  ```
  spark-shell
  ```

  ![sparkshell_start](./image/sparkshell_start.png)

  `sc`(SparkContext 객체)가 자동으로 초기화되고 이를 이용하여 RDD 생성 가능

  

  설정이 다를 경우 아래와 같이 직접 초기화해줄 수 있음

  ```scala
  import org.apache.spark.SparkContext
  import org.apache.spark.SparkConf
  
  // SparkContext 객체 초기화 
  // 클러스터 매니저의 타입 지정
  val conf = new SparkConf().setAppName("sample").setMaster("yarn")
  val sc = new SparkContext(conf)
  ```

  

  1. ###### 내부데이터 이용(Parallelized Collections)

     parallelize() 메소드를 이용하여 직접 데이터를 입력

     ```scala
     scala> val data = Array(1,2,3,4,5)
     data: Array[Int] = Array(1, 2, 3, 4, 5)
     
     scala> val disData = sc.parallelize(data, 5)
     disData: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[2] at parallelize at <console>:26
     
     scala> disData.filter(_ >= 4).reduce(_ + _)
     res0: Int = 9
     ```

     

  2. ###### 외부데이터 이용

     ```scala
     val flightdata = sc.textFile("/user/root/flightdata/*")
     ```

     ![sparkshell_rdd_outdata](C:\Users\joonhong\Documents\Study\image\sparkshell_rdd_outdata.png)

     RDD객체가 생성됨 

     

- ##### jar파일로 RDD객체 생성 후 출력

```
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.SparkSession;

import java.lang.reflect.Array;
import java.util.Arrays;
import java.util.List;

public class CreateRDD {

    public static void main(String[] args) throws Exception {

        if (args.length < 1){
            System.err.println("Usage : ReadCSV <input file path>");
            System.exit(1);
        }
        SparkSession spark = SparkSession.builder()
                .appName("Java Spark Read CSV")
                .getOrCreate();

        CSVtoRDD(spark,args);
    }

    private static void CSVtoRDD(SparkSession spark, String[] args) throws Exception {
        JavaSparkContext sc = new JavaSparkContext(spark.sparkContext());
        JavaRDD<String> rdd = sc.textFile(args[0]);

        JavaRDD<String[]> rdd_split = rdd.map(line -> line.split(",",0));
        JavaRDD<Row> rddOfRows = rdd_split.map(RowFactory::create);

        rddOfRows.take(10).forEach(System.out::println);
    }
}
```

```
cd ~/jar/spark
spark-submit --class CreateRDD sbj.jar /user/root/flightdata/*
```

![](./image/rdd_take.PNG)



## DataFrame

RDD와 같이 변경 할 수 없는(Immutable) 데이터 집합.

대신,  RDB의 관계형 테이블처럼 칼럼명과 데이터타입을 명시한 Schema를 가지고 있음



Spark SQL 옵티마이저로 데이터를 추출할 수 있고, optimization(최적화)를 할 수 있다.



Spark 2.0에서 Dataframe API는 Datasets API와 통합됨

![](./image/df+ds.PNG)



#### DataFrame을 사용하는 경우

- 높은 추상화의 API를 사용하고 싶을 때
- map, filters 등 다양한 Spark API를 사용하고 싶을 때
- JVM을 지원하지않는 R, Python 사용자일 때



잘 이해가 되지않는다면 Python Pandas의 DataFrame을 참고해볼 것

https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html



#### DataFrame 객체 생성

- ##### spark-shell에서 생성

  1. ###### 배열 RDD  --> DataFrame 

     ```scala
     // 위에서 만든 disData RDD 객체 사용
     
     scala> val disDF = disData.toDF()
     disDF: org.apache.spark.sql.DataFrame = [value: int]
     
     scala> disDF.show()
     +-----+
     |value|
     +-----+
     |    1|
     |    2|
     |    3|
     |    4|
     |    5|
     +-----+
     
     // Column명 설정
     scala> val disDF = disData.toDF("Number")
     disDF: org.apache.spark.sql.DataFrame = [Number: int]
     
     scala> disDF.show()
     +------+
     |Number|
     +------+
     |     1|
     |     2|
     |     3|
     |     4|
     |     5|
     +------+
     ```

  

  2. ###### 복합구조 RDD -> DataFrame

     ```scala
     scala> val coffeeRDD = sc.parallelize(
     	Seq(("Americano", 4300),
         ("CafeLatte", 5300),
         ("CafeMocha", 5800))
     )
     
     coffeeRDD: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[12] at parallelize at <console>:24
     
     scala> val coffeeDF = coffeeRDD.toDF("menu","price")
     
     coffeeDF: org.apache.spark.sql.DataFrame = [menu: string, price: int]
     
     scala> coffeeDF.show()
     +---------+-----+
     |     menu|price|
     +---------+-----+
     |Americano| 4300|
     |CafeLatte| 5300|
     |CafeMocha| 5800|
     +---------+-----+
     
     ```

     

  3. ###### RDD + Schema -> DataFrame

     ```scala
     import org.apache.spark.sql._
     import org.apache.spark.sql.types._
     
     scala> val trainRDD = sc.parallelize(
       Seq(
            Row("Seoul", "Busan", 46000),
            Row("Seoul", "Gwangju", 51000),
            Row("Seoul", "Daegu", 39000)
       )
     )
     
     trainRDD: org.apache.spark.rdd.RDD[org.apache.spark.sql.Row] = ParallelCollectionRDD[3] at parallelize at <console>:30
     
     scala> val trainSchema = new StructType().
     add(StructField("depart",   StringType, true)).
     add(StructField("arrive", StringType, true)).
     add(StructField("price", IntegerType, true))
     
     trainSchema: org.apache.spark.sql.types.StructType = StructType(StructField(depart,StringType,true), StructField(arrive,StringType,true), StructField(price,IntegerType,true))
     
     scala> val trainDF = spark.createDataFrame(trainRDD, trainSchema)
     
     trainDF: org.apache.spark.sql.DataFrame = [depart: string, arrive: string ... 1 more field]
     
     scala> trainDF.show()
     +------+-------+-----+
     |depart| arrive|price|
     +------+-------+-----+
     | Seoul|  Busan|46000|
     | Seoul|Gwangju|51000|
     | Seoul|  Daegu|39000|
     +------+-------+-----+
     
     ```

     

  4. ###### 외부데이터 이용

     [위의 csv 예제](#1.Spark-shell 에서 CSV 읽기) 참고



## Dataset

- Row타입의 Dataset == DataFrame, 즉, Dataset의 집합이 DataFrame

- JVM 기반으로 만들어져 R과 Python에서는 사용할 수 없음

- Spark 1.6에 도입되었으며 2.0에서 DataFrame API와 통합되면서 2가지 특징을 가지게 됨

  - Strongly-typed API	

    | Language |                Main Abstraction                 |
    | :------: | :---------------------------------------------: |
    |  Scala   | Dataset[T] & DataFrame (alias for Dataset[Row]) |
    |   Java   |                   Dataset[T]                    |
    |  Python  |                    DataFrame                    |
    |    R     |                    DataFrame                    |

    

  - Untyped API

    Schema의 Tpye이 정의되지 않고 Generic 타입으로 데이터 구조가 형성됨

    Spark에서는 아래와 같이 표현함

    ```
    DataFrame = Dataset[Row] 
    `Row` : generic untyped JVM objects.
    ```



#### Dataset을 사용하는 경우

- 높은 추상화의 API를 사용하고 싶을 때

- map, filters 등 다양한 Spark API를 사용하고 싶을 때

- 컴파일 단계에서 더 높은 수준의 type-safety를 확보하고 싶을 때

  ![](./image/typesafety.png)











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
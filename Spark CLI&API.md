# Spark CLI/API

## Spark로 csv 파일 읽기

Local에 저장 후 hdfs에 저장

```
hdsf dfs -mkdir /user/root/flightdata

cd ~/filghtdata

hdfs dfs -put * /user/root/flightdata

hdfs dfs -ls /user/root/flightdata
```



Spark-shell 실행하여 data 불러오기

```
spark-shell
```



```
val flightdata = spark.read.option("inferSchema", true).option("header",true).csv("hdfs:///user/root/flightdata/*")
```



```
import org.apache.spark.sql.types._
 
val customSchema = StructType(Array(StructField("DEST_COUNTRY_NAME",StringType,true),
   StructField("ORIGIN_COUNTRY_NAME",StringType,true),
   StructField("count",IntegerType,true)))
 
val flightdata = spark.read.format("csv").option("header", true).schema(customSchema).load("hdfs:///user/root/flightdata/*")
```



#### Delimiter가 Comma가 아닌 파일 읽기

```
val flightdata = spark.read.option("inferSchema", true).option("header",true).option("delimiter","\t").

csv("hdfs:///user/root/flightdata/*")
```



###### jar 파일로 만들어서 실행?? spark-submit



## RDD

RDD (Resilient Distributed Dataset) 

Spark에서 사용되는 가장 기본적인 데이터 객체



RDD를 제어하는 연산 종류

1. Transformation : 외부 데이터로 RDD를 로드하여 연산을 수행한 뒤 새로운 RDD를 생성하는 작업
2. Action : 궁극적으로 원하는 결과물(연산 결과가 RDD가 아닌 다른 타입인 경우)이 출력되는 작업



> RDD로 데이터를 로드하거나 Transformation 연산을 수행해도 실제로 데이터는 처리되지 않음
>
> 단지 계산을 위한 수행 계획이 만들어질 뿐, 실제 계산은 RDD를 처리하는 foreach()와 같은 Action을 호출할 때 수행됨

![](./image/rdd.PNG)





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
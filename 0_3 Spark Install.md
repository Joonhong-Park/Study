# Spark 설치

HDFS 설정이 완료된 계정에서 수행한다.



#### 다운로드

------

다운로드 안될시 해당 사이트 참조 : http://apache.mirror.cdnetworks.com/spark/

```shell
cd /path/to/spark/download
wget http://apache.mirror.cdnetworks.com/spark/spark-3.0.1/spark-3.0.1-bin-hadoop3.2.tgz
tar zxvf spark-3.0.0-preview2-bin-hadoop3.2.tgz
ln -s spark-3.0.0-preview2-bin-hadoop3.2 default
```



#### 환경변수 설정

------

##### vim ~/.bashrc

```shell
export SPARK_HOME="/path/to/spark/download/default"
export SPARK_CONF_DIR="${SPARK_HOME}/conf"
export PATH="${PATH}:${SPARK_HOME}/bin:${SPARK_HOME}/sbin"
```



#### 기본환경 설정

------

##### vim ${SPARK_CONF_DIR}/spark-defaults.conf

```scala
spark.master local[1]
spark.eventLog.enabled true
spark.eventLog.dir /path/to/spark/eventLog
spark.serializer org.apache.spark.serializer.KryoSerializer
spark.driver.memory 2g
```



#### 실행

```shell
# hdfs 실행
start-dfs.sh
 
# spark 실행
start-all.sh
 
# hdfs 올라왔는지 확인
hdfs dfs -ls /
 
# hdfs option 확인
hdfs -help
 
# spark-shell 실행
spark-shell
 
# spark-shell option 확인
spark-shell -h help
 
# spark-shell --master "local[*]" --executor-memory 32G --driver-memory 32G (환경을 고려하여 옵션 설정)
```



# spark 데이터 로드

##### 윈도우 로컬 >> 리눅스  파일 전송

- 윈도우콘솔창(cmd) >> scp 보낼파일경로 리눅스계정@IP주소:/home/리눅스계정



#### HDFS에 data 적재

------

```shell
# testdata 경로 생성
mkdir ./testdata
 
# 방금 만든 testdata 경로로 dataset 이동
mv ./데이터셋명* ./testdata/
 
# hdfs에 data 경로 생성
hdfs dfs -mkdir /data
 
# 방금 만든 data 경로에 dataset 적재
hdfs dfs -put ./데이터셋명* /data/
 
# 확인
hdfs dfs -ls /data/
```



#### spark-shell 실행하여 데이터 로드

------

```shell
spark-shell #spark-shell 실행
val data = spark.read.option("inferSchema", true).option("header", true).csv("hdfs:///data/데이터셋명*")
data.show(10)
```


# HPL/SQL

#### HIVE가 설치된 서버(hdm2)에 설치한다.



설치 문서 : http://hplsql.org/start

###### JDK가 설치되어 있어야 하며, JAVA_HOME, PATH가 등록되어 있어야 한다.



```
cd ~/Downloads

wget http://www.hplsql.org/downloads/hplsql-0.3.31.tar.gz

tar -zxf hplsql-0.3.31.tar.gz /opt

ln -s /opt/hplsql-0.3.31 /opt/hplsql

chmod +x /opt/hplsql
```



#### hplsql 실행파일 수정

##### vim /opt/hplsql

> 공식문서에 나온대로 export HADOOP_CLASSPATH="/opt/cloudera/parcels/CDH/jars/* 작성시
>
> arguments too long 오류가 발생되었음

###### 아래의 방법으로 작성하여 해결 함

```
#!/bin/bash

export JAVA_HOME="/usr/java/default/"
export HPLSQL_HOME="/opt/hplsql"
export PATH="${PATH}:${JAVA_HOME}/bin:${HPLSQL_HOME}"

#export HADOOP_CLASSPATH="/opt/cloudera/parcels/CDH/jars/*"
HADOOP_CLASSPATH="/opt/cloudera/parcels/CDH/jars/commons-cli-1.2.jar"
for f in $(find /opt/cloudera/parcels/CDH/jars/ -name 'hadoop*') ; do
  HADOOP_CLASSPATH="${HADOOP_CLASSPATH}:${f}"
done
for f in $(find /opt/cloudera/parcels/CDH/jars/ -name 'hive*') ; do
  HADOOP_CLASSPATH="${HADOOP_CLASSPATH}:${f}"
done
export HADOOP_CLASSPATH
#export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=/usr/lib/hadoop/lib/native"
export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=/opt/cloudera/parcels/CDH/lib/hadoop/lib/native"

SCRIPTPATH=${0%/*}

#java -cp $SCRIPTPATH:$HADOOP_CLASSPATH:$SCRIPTPATH/hplsql-0.3.31.jar:$SCRIPTPATH/antlr-runtime-4.5.jar $HADOOP_OPTS org.apache.hive.hplsql.Hplsql "$@"
java -cp $SCRIPTPATH:$HADOOP_CLASSPATH:$SCRIPTPATH/hive-hplsql-3.1.3.jar:$SCRIPTPATH/antlr-runtime-4.5.jar $HADOOP_OPTS org.apache.hive.hplsql.Hplsql "$@"
```



#### hplsql-site.xml 수정

default로 hive2conn이 설정되어있으며 hive2conn과 관련된 property들만 수정해주었다.

##### vim /opt/hplsql/hplsql-site.xml

```
...

<property>
  <name>hplsql.conn.init.hive2conn</name>
  <value>
     set mapred.job.queue.name=default;
     set hive.execution.engine=spark;
     use ED1DW;
  </value>
  <description>Statements for execute after connection to the database</description>
</property>

...
  <value>com.ibm.db2.jcc.DB2Driver;jdbc:db2://hdm2.cdp.jh.io:50001/metastore;hive;hive</value>
  <description>IBM DB2 connection</description>
</property>

...
```



### 확인

```
cd /opt/hplsql

./hplsql -e "CURRENT_DATE+1"
```

###### 내일 날짜가 출력되면 성공



##### 환경변수로 등록해놓으면 해당 경로로 가지않아도 명령어 사용 가능

```
export PATH=$PATH:/opt/hplsql
```



##### hive의 metastore에 test_table 생성 후 확인

```
hplsql -e "SELECT * FROM test_table LIMIT 1"
```

###### 테이블이 조회되면 성공



#### hive 한글명 table 생성?

##### 한글 Column, Comment는 작성가능하나 Table Name은 불가능해보임.


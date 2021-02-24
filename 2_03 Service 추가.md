# 서비스 추가

1. [Spark  추가](#Spark)
2. [Hive 추가](#Hive)
3. [Impala 추가](#Impala)
4. [Solr 추가](#Solr)
5. [Ranger 추가](#Ranger)



# Spark

- YARN 필요



#### 클러스터 명 > 작업 > 서비스 추가 > Spark 선택



#### Select Dependencies

YARN이 설치되어 있으면 그냥 넘어감

###### YARN이 없으면 어떻게 뜨는지는 확인 안해봄, 다음에 다시 설치해볼 시간이 있으면 해볼 것



#### 역할 할당 사용자 지정

History Server : hdm2

Gateway : edge



#### 변경 내용 검토

Auto-TLS disabled 시 해당 property들 공백으로 진행



#### 완료 후 구성 재배포 및 재시작



# Hive

- HDFS 필요



#### 클러스터 명 > 작업 > 서비스 추가 > Hive 선택



#### Select Dependencies

YARN, YARN Queue Manager 선택



#### 역할 할당 사용자 지정

Gateway : edge

Hive metastore server : hdm2

Hive server 2 : (Spark이 설치되어 있을시) hdm2



#### DB설정

###### 1_09 CDP_PostgreSQL에서 만든 DB 참고

유형 : PostgreSQL

Use JDBC URL Override : 아니오

호스트 이름 : hdm2.cdp.jh.io

데이터베이스 이름 : metastore

사용자이름 : hive

암호 : hive



#### 변경 내용 검토

수정 사항 없음



#### 완료 후 접속 확인

```bash
beeline -u "jdbc:hive2://hdm2.cdp.jh.io:10000/default;" -n hive -p hive
```



# Impala

- HDFS, Hive 필요



#### 클러스터 명 > 작업 > 서비스 추가 > Impala선택



#### Select Dependencies

HDFS, Hive 있을 시 그냥 넘어감



#### 역할 할당 사용자 지정

State Store : hdm2

Catalog Server : hdm2

Daemon : hdw[1-4]



#### 변경 내용 검토

수정할 것 없음



#### 완료 후  시작



Impala-shell 연결

```
impala-shell -i hdw1.cdp.jh.io -d default
```



# Solr



#### 클러스터 명 > 작업 > 서비스 추가 > Spark 선택



#### Select Dependencies

YARN이 설치되어 있으면 그냥 넘어감

###### YARN이 없으면 어떻게 뜨는지는 확인 안해봄, 다음에 다시 설치해볼 시간이 있으면 해볼 것



#### 역할 할당 사용자 지정

History Server : hdm2

Gateway : edge



#### 변경 내용 검토

Auto-TLS disabled 시 해당 property들 공백으로 진행



#### 완료 후 구성 재배포 및 재시작



# Ranger



#### 클러스터 명 > 작업 > 서비스 추가 > Spark 선택



#### Select Dependencies

YARN이 설치되어 있으면 그냥 넘어감

###### YARN이 없으면 어떻게 뜨는지는 확인 안해봄, 다음에 다시 설치해볼 시간이 있으면 해볼 것



#### 역할 할당 사용자 지정

History Server : hdm2

Gateway : edge



#### 변경 내용 검토

Auto-TLS disabled 시 해당 property들 공백으로 진행



#### 완료 후 구성 재배포 및 재시작


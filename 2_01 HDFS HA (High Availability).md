# HDFS HA (High Availability)

hadoop 2.0 부터 NameNode를 2대로 구성한 HA를 지원



##### Cloudera Manager > 클러스터 > HDFS > 작업 > High Availability 설정



#### 시작하기

Name Service : nn



#### 역할 할당

NameNode : hdm2

JournalNode : adm1, hdm1, hdm2

> NameNode : 데이터 블럭들에 대한 조회를 실시간으로 처리하기위한 메타 데이터 관리
>
> DataNode : 데이터 블럭을 읽고 씀
>
> JournalNode : 데이터 블럭들에 대한 이력(Journal)들 관리

###### 	JournalNode는 3개 이상 홀수 개를 선택해야 함



#### 변경내용 검토

##### dfs.journalnode.edits.dir

host 3개 모두 `/dfs/jn`으로 설정



#### 완료

> 설치 진행중 Failed to format NameNode Warning은 무시가능

##### Secondary NameNode 지워지고 NameNode가 2개, JournalNode가 3개가 됨



##### HA가 설정된 후 인스턴스 목록

![HDFS_HA](.\image\HDFS_HA.PNG)
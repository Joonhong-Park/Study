# 0. Hadoop & Spark



# 1. CDP Install

설치 전에 앞서..

Cloudera Manager 설치 시 초기 메모리를 기준으로 JAVA Heap 등을 할당하므로 메인이 될 adm1 서버는 여유있게 메모리를 할당할 것

> 설치 후에도 각 메모리를 재할당해줄 수 있으나 직접해본 결과 그리 바람직하지 않다고 느껴졌다. 각 메모리마다 구체적으로 할당해줄 수 없다면 처음부터 여유있게 할당해주는 편이 좋을 것 같다.

Test Installation Layout

| hostname | component                                                    | CDP Add component                                            |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| adm1     | DNS(master)<br />NTP Server<br />Kerberos(master)<br />LDAP Server<br />Postgres(for cm)<br />Cloudera Manager[cm]<br />Parcel Repository[cdh]<br />JDBC Driver(postgres) | Zookeeper #1<br />JournalNode #1                             |
| edge     | DNS(slave)<br />NTP Server<br />Kerberos(slave)              |                                                              |
| hdm1     | NTP Client                                                   | Namenode #1<br />HDFS balancer<br />Zookeeper #2<br />ZK failover controller<br />Resource Manager #2<br />JournalNode #2<br />Solr Server<br />Ranger [admin, usersync, tagsync] |
| hdm2     | NTP Client<br />Postgres(for hive)                           | Resource Manager #1<br />Yarn Queue Manager<br />Zookeeper #3<br />ZK failover controller<br />NameNode #2<br />Hive metastore Server<br />JournalNode #3<br />Impala[state store, catalog server] |
| hdw[1-4] | NTP Client                                                   | DataNode<br />NodeManager<br />Impala daemon                 |



1. DNS 설정
2. SSH 키 교환
3. Kerberos 설치 ( 사용할 경우 )
4. LDAP 설치 ( 사용할 경우 )
5. NTP 설정
6. JDK 설치
7. Prerequisites check를 위한 설정
8. Prerequisites check
9. Cloudera Manager 설치
10. PostgreSQL 설치
# Cloudera Product Install(7.1.4ver)

adm1에서 수행



### 1. Cloudera Manager를 위한 Repository 설정

#### httpd 설치

```
yum install -y httpd
```



#### httpd.conf 수정

**vim /etc/httpd/conf/httpd.conf**

```
...
AddType application/x-gzip .gz .tgz .parcel
...
```

##### httpd 시작

```
systemctl enable httpd; systemctl start httpd; systemctl status httpd;
```



#### Cloudera Manager 파일 다운로드

```
cd ~/Downloads
wget https://archive.cloudera.com/cm7/7.1.4/repo-as-tarball/cm7.1.4-redhat7.tar.gz
```

```
mkdir -p /var/www/html/cm7/7.1.4
cd /var/www/html/cm7/7.1.4
tar zxf ~/Downloads/cm7.1.4-redhat7.tar.gz
mv cm7.1.4/* .
rm -rf cm7.1.4/
```



#### CDH Parcel 다운로드

```
cd ~/Downloads
wget https://archive.cloudera.com/cdh7/7.1.4/parcels/CDH-7.1.4-1.cdh7.1.4.p0.6300266-el7.parcel
wget https://archive.cloudera.com/cdh7/7.1.4/parcels/CDH-7.1.4-1.cdh7.1.4.p0.6300266-el7.parcel.sha256
wget https://archive.cloudera.com/cdh7/7.1.4/parcels/manifest.json
```

```
mkdir -p /var/www/html/cdh/7.1.4.0/parcels
cd /var/www/html/cdh/7.1.4.0/parcels
mv ~/Downloads/CDH-7.1.4-1.cdh7.1.4.p0.6300266-el7.parcel* .
mv ~/Downloads/manifest.json .
```



#### Cloudera-manager repo file 추가

> yum install 시 repo에 등록해놓은 url로 설치됨

**vim /etc/yum.repos.d/cloudera-manager.repo**

```
[cloudera-manager]
name=Cloudera Manager 7.1.4
baseurl=http://adm1.cdp.jh.io/cm7/7.1.4/
gpgkey=http://adm1.cdp.jh.io/cm7/7.1.4/RPM-GPG-KEY-cloudera
gpgcheck=1
enabled=1
autorefresh=0
type=rpm-md
```

##### repository 추가

```
yum repolist
```



### 2. JDK 설치

###### 1_5 CDP_JDK에서 이미 수행함



### 3. Cloudera Manager 서비스 설치

######  repo에 등록된 url를 기준으로 설치함

```bash
yum install -y cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server
```



### 4. DB 설치 및 설정

###### 1_9 PostgreSQL에서 이미 수행함



### 5. Cloudera Manager DB Set up

```bash
/opt/cloudera/cm/schema/scm_prepare_database.sh -h adm1.cdp.jh.io postgresql scm scm scm
```



### 6. Runtime 및 Software 설치

```bash
systemctl start cloudera-scm-server
```

```bash
tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log

# INFO WebServerImpl:com.cloudera.server.cmf.WebServerImpl: Started Jetty server. 가 뜨면 성공
```

> Auto-TLS가 Enable되어 있는 경우 이 과정에서 KeyStore오류가 발생할 수 있음
>
> 또한 인증서가 없는 외부 Service와 연계가 힘드므로 Auto-TLS는 Disable로 두고 진행하도록 함

###### 

#### http://adm1.cdp.jh.io:7180 또는 7183 으로 접속

##### 초기 ID, PW : admin, admin

> ##### windows hosts파일에 도메인 주소를 등록해놔야 위의 주소로 접속할 수 있음
>
> 메모장 관리자 권한으로 실행 >  C:\Windows\System32\drivers\etc\hosts 파일 열기 > `IP주소 도메인주소` 등록

1. ##### 로그인

2. ##### 라이선스 선택

3. ##### 클러스터 이름 설정

4. ##### 참여 호스트 추가

5. ##### 레포지토리 선택

   - Cloudera Manger Agent : http://adm1.cdp.jh.io/cm7/714/

   - Parcel : 원격 레포지토리 URL 모두 삭제 > http://adm1.cdp.jh/io/cdh/7.1.4.0/parcels/ > Save

     연결 실패 시 경로 다시 확인 해볼 것

6. ##### JDK 선택

   - Manually manange JDK

7. ##### 로그인 정보 입력

   - root
   - 모든 호스트가 동일한 개인키 허용
   - Direct Input 
     - root@adm1.cdp.datadynamics.io:~/.ssh/ 경로의 id_rsa 파일 넣기

8. ##### 설치 진행

9. ##### Inspect Cluster

   - Inspect Network performance 수행
   - Inspect Hosts 수행

10. ##### 초기 서비스 선택

    - 최소한으로 설치할 경우 : HDFS, YARN(MR2 included), YARN Queue Manager, Zookeeper
    - Ranger 서비스를 이용할 경우 : Solr, Ranger 같이 초기부터 설치

    서비스마다 필요한 Dependency

    | Service       | Dependencies        |
    | ------------- | ------------------- |
    | HDFS          | -                   |
    | Zookeeper     | -                   |
    | YARN          | HDFS<br />Zookeeper |
    | Spark on YARN | YARN                |
    | Hive          | HDFS                |
    | HBase         | HDFS<br />Zookeeper |
    | Impala        | HDFS<br />Hive      |
    | Hue           | HDFS<br />Hive      |
    | Solr          | HDFS<br />Zookeeper |
    | Ranger        | HDFS<br />Solr      |
    | Oozie			| YARN				  |
    | Kafka	    	| Zookeeper		      |

11. ##### Assign Roles

    - HDFS
      - NN : hdm1
      - SNN : hdm2
      - Balancer : hdm1
      - DN : hdw[1-4]
      - Gateway : edge
    - CMS
      - SM : adm1
      - HM : adm1
      - RM : adm1
      - Event Server : adm1
      - Alert Publisher : adm1
    - YARN Queue Manager
      - YQMW : hdm2
    - YARN (MR2 inc)
      - RM : hdm2
      - JobHistory : hdm2
      - NM : DN
      - Gateway : edge
    - ZooKeeper
      - Server : adm1, hdm[1-2]

12. ##### DB 연결

    - 1_09 CDP_PostgreSQL 에서 만들어 둔 rman DB정보 입력

13. ##### Property 최종 확인

14. ##### 설치 진행 및 완료


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


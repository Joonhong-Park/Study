# PostgreSQL



### PostgreSQL server 설치

```
cd ~/Downloads

wget http://www.hadoop-professionals.org/download/PostgreSQL/9.6.17/redhat7/x86_64/postgresql96-9.6.17-1PGDG.rhel7.x86_64.rpm
wget http://www.hadoop-professionals.org/download/PostgreSQL/9.6.17/redhat7/x86_64/postgresql96-contrib-9.6.17-1PGDG.rhel7.x86_64.rpm
wget http://www.hadoop-professionals.org/download/PostgreSQL/9.6.17/redhat7/x86_64/postgresql96-libs-9.6.17-1PGDG.rhel7.x86_64.rpm
wget http://www.hadoop-professionals.org/download/PostgreSQL/9.6.17/redhat7/x86_64/postgresql96-server-9.6.17-1PGDG.rhel7.x86_64.rpm

yum localinstall -y postgresql96-*
```



### psycopg2 설치

```
cd ~/Downloads
wget http://www.hadoop-professionals.org/download/PostgreSQL/psycopg2-2.7.5-cp27-cp27mu-manylinux1_x86_64.whl
wget http://www.hadoop-professionals.org/download/PostgreSQL/python2-pip-8.1.2-10.el7.noarch.rpm

yum localinstall -y python2-pip-8.1.2-10.el7.noarch.rpm
pip install psycopg2-2.7.5-cp27-cp27mu-manylinux1_x86_64.whl --ignore-installed
```



### postgresql server 시작

```
echo 'LC_ALL="en_US.UTF-8"' >> /etc/locale.conf
/usr/pgsql-9.6/bin/postgresql96-setup initdb
```



### pg_hba.conf postgresql.conf 수정

##### vim /var/lib/pgsql/9.6/data/pg_hba.conf

```
...
# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            ident
host    all             all             IP주소.1/24              md5
# IPv6 local connections:
host    all             all             ::1/128                 ident
...
```



**vim /var/lib/pgsql/9.6/data/postgresql.conf**

> 클러스터의 사이즈게 맞게 설정함. 여기에서는 small 규모의 클러스터로 구성하였음

```
...
listen_addresses = '*'
max_connection = 100 # 추후에 50 씩 증가
shared_buffers = 256MB
wal_buffers = 8MB
checkpoint_completion_target = 0.9
...
```



### 서비스 시작

```
systemctl enable postgresql-9.6;systemctl start postgresql-9.6;systemctl status postgresql-9.6
```



##### 위의 과정을 hdm2에서도 진행해준다.



### CDP에서 사용할 DB 생성

##### amd1

```
su - postgres
psql
```

```
create role scm login password 'scm';
create database scm owner scm encoding 'UTF-8';
create role amon login password 'amon';
create database amon owner amon encoding 'UTF-8';
create role rman login password 'rman';
create database rman owner rman encoding 'UTF-8';
create role hue login password 'hue';
create database hue owner hue encoding 'UTF-8';
create role oozie login password 'oozie';
create database oozie owner oozie encoding 'UTF-8';
create role rangeradmin login password 'rangeradmin';
create database ranger owner rangeradmin encoding 'UTF-8';
grant all privileges on database ranger to rangeradmin;
```



##### hdm2

```
su - postgres
psql
```

```
create role hive login password 'hive';
create database metastore owner hive encoding 'UTF-8';
```



### Ranger 사용을 위한 connector 설정

##### adm1에서 진행

```
cd ~/Downloads
wget http://www.hadoop-professionals.org/download/PostgreSQL/jdbc/postgresql-42.2.12.jar

mkdir -p /usr/share/java
cp ~/Downloads/postgresql-42.2.12.jar /usr/share/java/postgresql-connector-java.jar
chmod 644 /usr/share/java/postgresql-connector-java.jar
```



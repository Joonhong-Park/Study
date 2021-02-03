# HDFS 설정

#### 실습을 위한 유저 추가

- root 계정에서 유저 추가 후 비밀번호 설정 (CentOS)

  ```shell
  useradd ddadmin
  passwd ddadmin
  
  # 유저목록 확인
  cat /etc/passwd
  ```

- ddadmin 계정으로 로그인

  ```shell
  su - ddadmin
  ```

- 심볼릭 링크 소유자 변경

  ```shell
  # root가 소유하고 있는 default 링크 소유자를 ddadmin으로 변경
  chown -h ddadmin:ddadmin default
  ```

  - root 이외의 계정에서 permission denied 발생 시 해당 파일 외에 상위 디렉토리의 접근 권한들도 확인해볼 것 

- 환경변수 설정

  ##### vim ~/.bashrc

  ```shell
  export JAVA_HOME="/opt/java/default"
  export HADOOP_HOME="/opt/hadoop/default"
  export HADOOP_COMMON_HOME="${HADOOP_HOME}"
  export HADOOP_HDFS_HOME="${HADOOP_HOME}"
  export HADOOP_YARN_HOME="${HADOOP_HOME}"
  export HADOOP_MAPRED_HOME="${HADOOP_HOME}"
  export HADOOP_CONF_DIR="${HADOOP_HOME}/etc/hadoop"
  export PATH="${JAVA_HOME}/bin:${PATH}:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin"
  ```

- SSH Key 생성

  ```
  ssh-keygen -t rsa
  ```

- Key 교환

  ```shell
  ssh-copy-id ddadmin@도메인주소
  # 타겟서버의 .ssh 폴더안에 authorized_keys 파일에 public key값이 추가됨
  ```

  

#### 리소스제한(vim /etc/security/limits.conf)

------

- 리소스 제한에 대한 내용은 vim /etc/security/limits.conf 에 작성할 수 있으며

  vim /etc/security/limits.d/임의의파일명 으로 도메인마다 할당할 수도 있다.

  ```shell
  ddadmin - nofile 131072 # 해당 도메인이 오픈할 수 있는 최대 파일 개수
  ddadmin - nproc  65536 # 해당 도메인의 최대 프로세스 개수
  ```

  

#### core-site.xml 수정

------

- ##### core-site.xml

  - HDFS와 MR에서 공통적으로 사용할 환경정보 설정

  ##### vim ${HADOOP_CONF_DIR}/core-site.xml

  ```xml
  <configuration>
          <property>
                  <name>fs.defaultFS</name>
                  <value>hdfs://host-domain:8020/</value>
          </property>
          <property>
                  <name>io.file.buffer.size</name>
                  <value>131072</value>
          </property>
  </configuration>
  ```

  

#### hdfs directory 생성

------

```shell
mkdir -p /hadoop/nn
mkdir -p /hadoop/dn
chown -R ddadmin:ddadmin /hadoop
```

​	chown 파일의 소유자 변경

​	R : 파일과 디렉토리에 재귀적용

​	/hadoop의 소유유저를 ddadmin으로, 그룹을 ddadmin으로 변경



#### hdfs-site.xml 수정

------

- ##### hdfs-site.xml

  - HDFS에서 사용할 환경 정보

  ##### vim ${HADOOP_CONF_DIR}/hdfs-site.xml

  ```xml
  <configuration>
          <property>
                  <name>dfs.namenode.name.dir</name> <!-- ","를 이용해 여러개 지정 가능 -->
                  <value>file:///hadoop/nn</value>
          </property>
          <property>
                  <name>dfs.replication</name> 
                  <value>1</value> <!-- 3 node 이상일 경우 3 지정 가능 -->
          </property>
          <property>
                  <name>dfs.blocksize</name>
                  <value>134217728</value> 
          </property>
          <property>
                  <name>dfs.namenode.rpc-bind-host</name>
                  <value>0.0.0.0</value>
          </property>
          <property>
                  <name>dfs.namenode.handler.count</name>
                  <value>100</value>
          </property>
          <property>
                  <name>dfs.datanode.data.dir</name> <!-- ","를 이용해 여러개 지정 가능 -->
                  <value>file:///hadoop/dn</value> 
          </property>
  </configuration>
  ```



#### Namenode 포맷

------

```shell
hdfs namenode -format
```

- dfs.namenode.name.dir 경로의 fsimage, edits 초기화
- 클러스터id 신규 생성 ( dfs.namenode.name.dir/VERSION 에 존재)



#### hdfs 프로세스 실행

------

```shell
${HADOOP_HOME}/sbin/start-dfs.sh
```

- jps 명령어로 확인



#### 방화벽 해제

------

```shell
firewall-cmd --permanent --add-port=8020/tcp
firewall-cmd --permanent --add-port=9864/tcp
firewall-cmd --permanent --add-port=9870/tcp
systemctl reload firewalld
firewall-cmd --list-all
```

- http:// hostname:9870/ 접속



# YARN 설정

#### mapred-site.xml 수정

------

- MR에서 사용할 환경정보 설정

##### vim ${HADOOP_CONF_DIR}/mapred-site.xml

```xml
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
        <property>
                <name>mapreduce.application.classpath</name>               		   <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
        </property>
</configuration>
```



#### yarn-site.xml 수정

------

- MR에서 사용하는 셔플 서비스 지정

##### vim ${HADOOP_CONF_DIR}/yarn-site.xml

```xml
<configuration>
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
        </property>
</configuration>
```



#### yarn 프로세스 실행

------

```shell
${HADOOP_HOME}/sbin/start-yarn.sh
```

- jps 명령어로 확인



#### 방화벽 해제

------

```shell
firewall-cmd --permanent --add-port=8088/tcp
systemctl reload firewalld # 방화벽 재실행
firewall-cmd --list-all # 방화벽 목록 
```

- http:// hostname:8088/ 접속



# 실행 스크립트

```shell
# 전체 시작
${HADOOP_HOME}/sbin/start-all.sh
# hdfs 시작
${HADOOP_HOME}/sbin/start-dfs.sh
# yarn 시작
${HADOOP_HOME}/sbin/start-yarn.sh
# 전체 중지
${HADOOP_HOME}/sbin/stop-all.sh
# hdfs 중지
${HADOOP_HOME}/sbin/stop-dfs.sh
# yarn 중지
${HADOOP_HOME}/sbin/stop-yarn.sh
```


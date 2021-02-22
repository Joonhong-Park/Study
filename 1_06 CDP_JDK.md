# JDK



oracle JDK는 라이선스가 필요하므로 open JDK 1.8 ver을 설치함

JDK는 참여하는 모든 노드(amd1,edge,hdm[1-2],hdw[1-4])에 설치함

```bash
seq 1 8 | xargs -P 8 -I {} ssh IP주소{} "mkdir -p /usr/java"
cd ~/Downloads

wget http://www.hadoop-professionals.org/download/OpenJDK/zulu8.46.0.19-ca-jdk8.0.252-linux_x64.tar.gz

seq 1 8 | xargs -P 8 -I {} scp zulu8.46.0.19-ca-jdk8.0.252-linux_x64.tar.gz IP주소{}:/usr/java

seq 1 8 | xargs -P 8 -I {} ssh IP주소{} "cd /usr/java ; tar zxf zulu8.46.0.19-ca-jdk8.0.252-linux_x64.tar.gz ; rm -rf zulu8.46.0.19-ca-jdk8.0.252-linux_x64.tar.gz ; chown -R root:root zulu8.46.0.19-ca-jdk8.0.252-linux_x64 ; ln -s zulu8.46.0.19-ca-jdk8.0.252-linux_x64 default"
```



version check

```
for i in {1..8} ; do ssh IP주소${i} "/usr/java/default/bin/java -version" ; done
```

 

JAVA_HOME 설정

Cloudera Manager에 추가되는 서비스 중 JAVA_HOME을 사용하는 서비스가 일부 있으므로 모든 서버에 JAVA_HOME 환경 변수를 추가해줌

```
for i in {1..8} ; do ssh IP주소${i} "export JAVA_HOME=/usr/java/default/jre/" ; done

# 설정 확인
for i in {1..8} ; do ssh IP주소${i} "hostname; echo ${JAVA_HOME}" ; done
```

+ ##### export로 환경변수 설정 시 리부팅 후 초기화되므로 영구적으로 사용하고자 하면 ./bashrc 파일에 JAVA_HOME경로를 추가하면 된다.
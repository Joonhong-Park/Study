## 1. 가상환경 생성

1. Citrix Hypervisor (XenServer) 다운로드

   https://www.citrix.com/ko-kr/downloads/citrix-hypervisor/

2. Add Server 

3. VM 추가 ( VM Template 선택 )

4. Storage, CPU, Network 설정

   - 필요시 VIM 설치 (Centos OS)

     ```bash
     yum -y install vim-enhanced
     ```

     ```shell
     # 명령어를 OS에 등록하여 모든사용자가 사용가능하게 설정
     vi /etc/profile
     
     # 마지막 행에 alias 추가
     alias vi = 'vim'
     
     # 수정 내용 반영
     source /etc/profile
     ```

     

## 2. Hadoop 설치

1. #####  hadoop 다운로드

   ```bash
   cd /opt/hadoop
   wget http://apache.mirror.cdnetworks.com/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz
   tar zxf hadoop-3.2.1.tar.gz
   ln -s hadoop-3.2.1 default
   ```

   - wget 설치 (Centos OS)

     ```bash
     yum install wget
     ```

2. ##### 자바 없을시 자바 설치

   - openjdk 압축파일 다운로드 : https://jdk.java.net/java-se-ri/8-MR3

   ```shell
   wget --no-cookies --no-check-certificate 복사한 주소
   ```

   - 압축 해제

   ```shell
   tar xzf 다운된 압축 파일명
   ```

   - 심볼릭 링크(해제된 파일명을 default라고 칭함)

   ```bash
   ln -s 압축 해제된 파일명 default				
   ```

3. ##### 환경변수 설정

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

4. ##### 설치 확인

   ```
   hdfs dfs -ls /
   ```

   root 디렉토리 리스트가 나오면 성공

   

5. ##### Passwordless

   hadoop을 다수의 노드에 설치하고 실행하기 위해서 모든 노드는 SSH Key를 교환해야 함

   - SSH Key 생성

     ```bash
     ssh-keygen -t rsa
     ```

   - 생성된 키 확인

     ```bash
     ls -al ~/.ssh/
     ```

     authorized_keys, id_rsa, id_rsa.pub 가 생겨있으면 성공

     | 종류            | 설명                                                         |
     | :-------------- | :----------------------------------------------------------- |
     | id_rsa          | private key, 절대로 타인에게 노출되면 안된다.                |
     | id_rsa.pub      | public key, 접속하려는 리모트 머신의 authorized_keys에 입력한다. |
     | authorized_keys | 리모트 머신의 .ssh 디렉토리 아래에 위치하면서 id_rsa.pub 키의 값을 저장한다. |

     

   ## 리눅스 파일 권한

   d --- --- --- : 디렉토리/루트권한/그룹권한/타유저권한 (2진수)

   r : read 

   w : write

   x : excute 

   drw-r--r--(644) : 그룹, 타유저 read 권한을 가지므로 보안성 낮음 >> chmod 600로 그룹, 타유저 권한 제외

   chmod + x : 실행 권한 추가 부여

   chmod g+w : 그룹에 write 권한 부여

   chmod o-rwx : 타유저의 모든 권한 박탈
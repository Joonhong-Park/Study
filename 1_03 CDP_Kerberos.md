# Kerberos 설치

> (Master, Slave) 는 명령어가 수행되는 위치를 뜻함

#### 커베로스 다운로드 (Master, Slave)

------

```bash
mkdir -p ~/Downloads/krb5/rpms
cd ~/Downloads/krb5/rpms
# 한번에 다운로드
wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-appl-clients-1.0.3-10.el7.x86_64.rpm;wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-appl-servers-1.0.3-10.el7.x86_64.rpm;wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-devel-1.15.1-37.el7_7.2.x86_64.rpm;wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-libs-1.15.1-37.el7_7.2.x86_64.rpm;wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-pkinit-1.15.1-37.el7_7.2.x86_64.rpm;wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-server-1.15.1-37.el7_7.2.x86_64.rpm;wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-server-ldap-1.15.1-37.el7_7.2.x86_64.rpm;wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-workstation-1.15.1-37.el7_7.2.x86_64.rpm;wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/libkadm5-1.15.1-37.el7_7.2.x86_64.rpm;
# 로컬 설치
yum localinstall -y krb5-* libkadm5-*
```



#### krb5.conf 수정(Master)

---

```
vim /etc/krb5.conf
```

```
# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/
 
[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log
 
[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
 pkinit_anchors = /etc/pki/tls/certs/ca-bundle.crt
 default_realm = CDP.JH.IO
 default_ccache_name = KEYRING:persistent:%{uid}
 
[realms]
 CDP.JH.IO = {
  kdc = adm1.cdp.jh.io
  kdc = edge.cdp.jh.io
  admin_server = adm1.cdp.jh.io
 }
 
[domain_realm]
 .cdp.jh.io = CDP.JH.IO
  cdp.jh.io = CDP.JH.IO
```



#### kdc.conf 수정(Master)

---

```
vim /var/kerberos/krb5kdc/kdc.conf
```

```
[kdcdefaults]
 kdc_listen = 88
 kdc_tcp_listen = 88
 
[realms]
 CDP.JH.IO = {
  kadmind_port = 749
  max_life = 12h 0m 0s
  max_renewable_life = 7d 0h 0m 0s
  master_key_type = aes256-cts
  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
 }
 
[logging]
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmin.log
 default = FILE:/var/log/krb5lib.log
```



#### DB생성(Master)

---

```bash
kdb5_util create -r CDP.JH.IO -s

# 생성된 파일 확인
ls -alF /var/kerberos/krb5kdc/
```



#### ACL 파일에 관리자 추가 (Master)

---

##### vim /var/kerberos/krb5kdc/kadm5.acl

```
*/admin@CDP.JH.IO *
```



#### DB에 관리자 추가(Master)

---

```bash
kadmin.local
addprinc admin/admin@CDP.JH.IO
```

- ##### 추가 - Keytab 생성 

  ```
  ktadd -k /var/kerberos/krb5kdc/kadm5.keytab kadmin/admin kadmin/changepw
  
# 나가기 : q 또는 quit
  ```

  ##### 생성된 Keytab 확인
  
  ```
  ls -alF /var/kerberos/krb5kdc/
  ```



#### 마스터 KDC에서 커베로스 데몬 시작(Master)

---

```
systemctl enable krb5kdc;systemctl start krb5kdc;systemctl status krb5kdc
systemctl enable kadmin;systemctl start kadmin;systemctl status kadmin
```

- ##### log 확인

  ```
  cat /var/log/krb5kdc.log
  cat /var/log/kadmin.log
  cat /var/log/kadmind.log
  ```

  ###### log에 에러 발생시 아래 명령어 수행

  ###### (이라고는 하는데 Kadmin 명령어를 쓰기 위해서는 무조건 해줘야 하는 것 같음)
  
  ```
kinit admin/admin@CDP.JH.IO
  ```
  
  

#### Slave KDC를 위한 호스트키 생성 (Master)

---

```
kadmin
addprinc -randkey host/adm1.cdp.jh.io
addprinc -randkey host/edge.cdp.jh.io
```

- ##### keytab생성

  ```
  ktadd host/adm1.cdp.jh.io
  ktadd host/edge.cdp.jh.io
  q
  ```

  

#### DB Propagation을 위한 Slave KDCs 셋업(Masters)

---

##### 설정한 파일들을 slave인 edge노드로 복사하는 과정

```bash
scp /etc/krb5.conf root@Edge IP주소:/etc
scp /var/kerberos/krb5kdc/{kdc.conf,kadm5.acl,.k5.CDP.JH.IO} root@Edge IP주소:/var/kerberos/krb5kdc
```



#### ACL 파일에 마스터 추가(Slave)

---

##### vim /var/kerberos/krb5kdc/kpropd.acl

```bash
host/adm1.cdp.jh.io@CDP.JH.IO
```



#### DB Propagation을 위한 Key 추가(Slave)

---

###### 등록하라고 나오지만 위 과정에서 이미 Key가 등록되는 것으로 보임 (혹시 모르니 해줌)

```bash
kadmin -p admin/admin
addprinc -randkey host/edge.cdp.jh.io
ktadd host/edge.cdp.jh.io
quit
```



#### kprop 실행(Slave)

---

```bash
systemctl enable kprop;systemctl start kprop;systemctl status kprop
```



#### Slave KDC로 DB 전파(Master)

---

##### db dump 생성

```bash
kdb5_util dump /var/kerberos/krb5kdc/slave_datatrans
ls -alF /var/kerberos/krb5kdc
```



##### 각 Slave KDC로 DB 전송

```
kprop -f /var/kerberos/krb5kdc/slave_datatrans edge.cdp.jh.io
```

- ###### 주기적인 dump를 위해 cron job으로 등록하기도 함

  ```bash
  vim ~/cron_kdb_dump.sh
  ```

  ```bash
  TODAY=$(date +'%Y%m%d')
  kdb5_util dump /var/kerberos/krb5kdc/slave_datatrans_${TODAY}
  kprop -f /var/kerberos/krb5kdc/slave_datatrans_${TODAY} slave호스트명
  ```

  ```bash
  # 실행 권한 추가
  chmod 755 ~/cron_kdb_dump.sh
  ```

  

#### Slave KDCs 설치 마무리 (Slave)

```bash
kdb5_util stash

# Using existing stashed keys to update stash file. 이 뜨면 업데이트 된것
```



#### 각각의 KDC에서 krb5kdc 데몬 시작(Slave)

---

```
systemctl enable krb5kdc;systemctl start krb5kdc;systemctl status krb5kdc
```


# Kerberos 설치

> (M,S) 는 명령어가 수행되는 위치를 뜻함

#### 커베로스 다운로드 (Master, Slave)

------

```bash
mkdir -p ~/Downloads/krb5/rpms
cd ~/Downloads/krb5/rpms
wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-appl-servers-1.0.3-10.el7.x86_64.rpm
wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-devel-1.15.1-37.el7_7.2.x86_64.rpm
wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-libs-1.15.1-37.el7_7.2.x86_64.rpm
wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-pkinit-1.15.1-37.el7_7.2.x86_64.rpm
wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-server-1.15.1-37.el7_7.2.x86_64.rpm
wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-server-ldap-1.15.1-37.el7_7.2.x86_64.rpm
wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/krb5-workstation-1.15.1-37.el7_7.2.x86_64.rpm
wget http://www.hadoop-professionals.org/download/kerberos/1.15.1/libkadm5-1.15.1-37.el7_7.2.x86_64.rpm
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
 default_realm = 도메인명
 default_ccache_name = KEYRING:persistent:%{uid}
  
[realms]
 SKY.LOCAL = {
  kdc = Master 호스트명
  kdc = Slave 호스트명
  admin_server = Master 호스트명
 }
  
[domain_realm]
 .도메인(소문자) = 도메인(대문자)
  도메인(소문자) = 도메인(대문자)
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
 SKY.LOCAL = {
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

```
kdb5_util create -r 도메인명(대문자) -s
```



#### ACL 파일에 관리자 추가 (Master)

---

```
vim /var/kerberos/krb5kdc/kadm5.acl
```

```
*/admin@도메인명(대문자) *
```



#### DB에 관리자 추가(Master)

---

```
kadmin.local
addprinc admin/admin@도메인명(대문자)
```

- ##### 추가 - Keytab 생성 

  ```
  ktadd -k /var/kerberos/krb5kdc/kadm5.keytab kadmin/admin kadmin/changepw
  ```

  ##### 생성된 Keytab 확인

  ```
  ls -alF /var/kerberos/krb5kdc/
  ```



#### 마스터 KDC에서 커베로스 데몬 시작(Master)

---

```
systemctl status krb5kdc
systemctl enable krb5kdc
systemctl start krb5kdc
systemctl status krb5kdc
systemctl status kadmin
systemctl enable kadmin
systemctl start kadmin
systemctl status kadmin
```

- ##### log 확인

  ```
  cat /var/log/krb5kdc.log
  cat /var/log/kadmin.log
  cat /var/log/kadmind.log
  ```

  ###### log에 에러 발생시 아래 명령어 수행

  ```
  kinit admin/admin@도메인(대문자)
  ```

  

#### Slave KDC를 위한 호스트키 생성 (Master)

---

```
kadmin
addprinc -randkey host/master호스트명
addprinc -randkey host/slave호스트명
```

- ##### keytab생성

  ```
  ktadd host/master호스트명
  ktadd host/slave호스트명
  quit
  ```

  

#### DB Propagation을 위한 Slave KDCs 셋업(Masters)

---

```
scp /etc/krb5.conf root@slave IP주소:/etc
scp /var/kerberos/krb5kdc/{kdc.conf,kadm5.acl,.k5.XXX.XXX.XXX} root@slave IP주소:/var/kerberos/krb5kdc
```



#### ACL 파일에 마스터 추가(Slave)

---

```
vim /var/kerberos/krb5kdc/kpropd.acl
```

```
host/master호스트명@도메인명(대문자)
```



#### DB Propagation을 위한 Key 추가(Slave)

---

```
kadmin -p admin/admin
addprinc -randkey host/slave호스트명
ktadd host/slave호스트명
quit
```



#### kprop 실행(Slave)

---

```
systemctl status kprop
systemctl enable kprop
systemctl start kprop
systemctl status kprop
```



#### Slave KDC로 DB 전파(Master)

---

##### db dump 생성

```
kdb5_util dump /var/kerberos/krb5kdc/slave_datatrans
ls -alF /var/kerberos/krb5kdc
```



##### 각 Slave KDC로 DB 전송

```
kprop -f /var/kerberos/krb5kdc/slave_datatrans slave호스트명
```

- ###### 주기적인 dump를 위해 cron job으로 등록하기도 함

  ```
  vim ~/cron_kdb_dump.sh
  ```

  ```
  TODAY=$(date +'%Y%m%d')
  kdb5_util dump /var/kerberos/krb5kdc/slave_datatrans_${TODAY}
  kprop -f /var/kerberos/krb5kdc/slave_datatrans_${TODAY} slave호스트명
  ```

  ```
  # 실행 권한 추가
  chmod 755 ~/cron_kdb_dump.sh
  ```

  

#### Slave KDCs 설치 마무리 (Slave)

```
kdb5_util stash
```



#### 각각의 KDC에서 krb5kdc 데몬 시작(Slave)

---

```
systemctl status krb5kdc
systemctl enable krb5kdc
systemctl start krb5kdc
systemctl status krb5kdc
```


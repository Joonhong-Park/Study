# NTP

network로 시간을 동기화 하는 프로토콜

상시 켜져있는 서버는 chronyd 대신 ntpd를 사용해야 한다.

>adm1, edge 서버를 ntp server로
>
>hdm1~2, hdw1~4 서버를 ntp client로 설정하는것을 목표로 한다.



#### chronyd 중지

---

```
systemctl disable chronyd
systemctl stop chronyd
systemctl status chronyd
```

- ##### 모든 서버에 적용시

  ```
  for i in {1..8} ; do ssh xxx.xx.x.xx${i} "systemctl disable chronyd ; systemctl stop chronyd ; systemctl status chronyd" ; done
  ```

  

#### NTP 설치

---

##### 모든서버에 설치

```
yum install -y ntp
```

```bash
# 한번에 설치되지만 순서대로 진행되므로 아래의 스크립트 사용
for i in {1..8} ; do ssh IP주소${i} "yum install -y ntp" ; done
```

```bash
# 한번에 설치
seq 1 8 | xargs -P 8 -I {} ssh IP주소{} "yum install -y ntp"
```



#### ntp.conf 작성

---

##### 기존 설정은 백업해둔다

```bash
mv /etc/ntp.conf /etc/ntp.conf.bak
```



###### ntp server에서 작성할 내용

```bash
vim /etc/ntp.conf
```

```bash
# time.bora.net, kr.pool.ntp.org 는 ntp server 가 이미 존재하는 경우 그곳으로 지정한다

driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict ::1
restrict IP주소.30.1.0 mask 255.255.255.0 nomodify notrap
server time.bora.net iburst prefer
server kr.pool.ntp.org iburst
server 127.127.1.0 iburst
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
```

다른 서버(edge로 복사)

```
scp /etc/ntp.conf root@edge.cdp.jh.io:/etc
```



###### ntp client에서 작성할 내용

```
vim /etc/ntp.conf
```

```
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1
restrict ::1
restrict 도메인주소.0 mask 255.255.255.0 nomodify notrap
server adm1-ip주소 iburst prefer
server edge-ip주소 iburst
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
```

```bash
for i in {4..8} ; do scp /etc/ntp.conf IP주소${i}:/etc/ntp.conf ; done
```



설정확인

```
for i in {1..8} ; do ssh IP주소${i} "hostname ; cat /etc/ntp.conf ; echo" ; done
```



#### 방화벽 중지 또는 ntp 허용

---

- ##### 중지

  ```
  systemctl stop firewalld
  systemctl disable firewalld
  ```

- ##### ntp 허용

  ```
  firewall-cmd --add-service=ntp --permanent
  firewall-cmd --reload
  ```



#### NTP 서비스 시작

---

```
systemctl enable ntpd
systemctl start ntpd
systemctl status ntpd
```

- ##### 모든 서버 실행 시

  ```
  for i in {1..8} ; do ssh xxx.xx.x.xx${i} "systemctl enable ntpd ; systemctl start ntpd ; systemctl status ntpd" ; done
  ```

  

#### 동기화 확인(Server, Client)

---

```
ntpq -p
ntpstat
```


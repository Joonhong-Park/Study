# CDP (클라우데라 데이터 플랫폼)

## CDP 설치를 위한 사전준비

adm1, edge, hdm[1-2], hdw[1-4]  총 8개의 노드 준비 (CentOS)



#### DNS 구조 변경

------

Master : adm1

Slave : edge

##### hostname 변경 및 고정 IP 설정

- hostname 변경

  ```bash
  hostnamectl set-hostname 변경할호스트명 # 영구 변경, 리부팅 후 적용
  ```

- 고정 IP 설정

  ```shell
  vim /etc/sysconfig/network-scripts/ifcfg-eth0
  ```

  ```sh
  TYPE="Ethernet"
  PROXY_METHOD="none"
  BROWSER_ONLY="no"
  BOOTPROTO="static" # IP고정, dhcp = 유동IP
  DEFROUTE="yes"
  IPV4_FAILURE_FATAL="no" 
  IPV6INIT="no" 
  IPV6_AUTOCONF="yes"
  IPV6_DEFROUTE="yes"
  IPV6_FAILURE_FATAL="no"
  IPV6_ADDR_GEN_MODE="stable-privacy"
  NAME="eth0"
  UUID="유니크ID"
  DEVICE="eth0"
  ONBOOT="yes"
  IPADDR="xxx.x.xx.xxx" # 변경할 IP
  DNS1="1.1.1.1" # 클라우드페어 DNS
  DNS2="1.1.1.2"
  DNS3="8.8.8.8" # 구글 DNS
  DNS4="8.8.4.4"
  DNS5="168.126.63.1" # KT DNS
  DNS6="168.126.63.2"
  GATEWAY="xxx.x.xx.xxx" # 게이트웨이
  ```

  ```shell
  systemctl restart network # 네트워크 재실행
  ```



#### Bind 설치

------

master, slave 노드 모두 설치

```shell
yum install -y bind
```



#### named 설정

------

##### Master

###### vim /etc/named.conf

```shell
...
    listen-on port 53 { any; };
...
    listen-on-v6 port 53 { none; };
...
    allow-query     { any; };
...
zone "도메인(abc.abc.ab)" IN {
    type master;
    file "도메인.zone";
    allow-update { none; };
};
 
zone "고정ip(1.11.111).in-addr.arpa" IN {
    type master;
    file "도메인.rev.zone";
    allow-update { none; };
};
...
```

##### Slave

###### vim /etc/named.conf

```shell
...
    listen-on port 53 { any; };
...
    listen-on-v6 port 53 { none; };
...
    allow-query     { any; };
...
zone "도메인(abc.abc.ab)" IN {
    type slave;
    file "도메인(abc.abc.ab).zone";
    masters { 마스터 IP주소; };
};
 
zone "고정ip(1.11.111).in-addr.arpa" IN {
    type slave;
    file "도메인(abc.abc.ab).rev.zone";
    masters { 마스터 IP주소; };
};
...
```



#### zone 작성

------

- master에서만 작성

- hostname은 반드시 .로 끝냄,  _ 사용 금지

##### vim /var/named/도메인.zone

```shell
$TTL 6H
; SOA: Indicates authority for this zone
@   IN  SOA  adm1.도메인. adm1.도메인. ( ; {primary-name-server}  {hostmaster-email}
                                             2020041423 ; serial
                                             1D ; refresh
                                             1H ; retry
                                             1W ; expire
                                             3H ) ; minimum
; NS: Lists a nameserver for this zone
    IN  NS      adm1.도메인
    IN  NS      edge.도메인
    IN  MX  10  adm1.도메인
 
; A: Name-to-address mapping
; PTR: Address-to-name mapping
; CNAME: Canonical name (for aliases)
; Hosts Here
adm1    IN  A   IP주소
edge    IN  A   IP주소
hdm1    IN  A   IP주소
hdm2    IN  A   IP주소
hdw1    IN  A   IP주소
hdw2    IN  A   IP주소
hdw3    IN  A   IP주소
hdw4    IN  A   IP주소
 
; kerberos 적용시에만 아래 부분을 추가함
_kerberos                TXT                    "도메인(대문자)"
kerberos1             IN CNAME                  adm1.도메인
kerberos2             IN CNAME                  edge.도메인
 
; _service._proto.name TTL class SRV  priority  weight  port  target
_kerberos._udp                   SRV  0         0       88    adm1.도메인
_kerberos-master._udp            SRV  0         0       88    adm1.도메인
_kerberos-adm._tcp               SRV  0         0       749   adm1.도메인
_kpasswd._udp                    SRV  0         0       464   adm1.도메인
```



#### reverse zone 작성

------

- master에서만 작성

##### vim /var/named/도메인.rev.zone

```shell
$TTL 6H
@   IN  SOA adm1.도메인. adm1.도메인. (
                                            2020041423 ; serial
                                            1D ; refresh
                                            1H ; retry
                                            1W ; expire
                                            3H ) ; minimum
    IN  NS  adm1.도메인.
    IN  NS  edge.도메인.
 
; IP-Domain mapping here
231 IN  PTR adm1.도메인.
232 IN  PTR edge.도메인.
233 IN  PTR hdm1 도메인.
234 IN  PTR hdm2.도메인.
235 IN  PTR hdw1.도메인.
236 IN  PTR hdw2.도메인.
237 IN  PTR hdw3.도메인.
238 IN  PTR hdw4.도메인.
```



#### zone 파일 권한 수정

------

```shell
chown named:root /var/named/도메인.zone
chown named:root /var/named/도메인.rev.zone
```



#### named 서비스 시작 / 방화벽 해제

------

- master, slave 모두 수행

  ```shell
  systemctl enable named
  systemctl start named
  systemctl status named
  systemctl stop firewalld
  systemctl disable firewalld
  ```

  

#### 네트워크 인터페이스 수정

------

```shell
...
DNS1="Master IP"
DNS2="Slave IP"
...
```

###### 네트워크 재시작

```shell
systemctl restart network
```



#### Ping test로 확인

------

```shell
ping -c 1 adm1
ping -c 1 호스트도메인
ping -c 1 IP주소

# 모두 가능해야 제대로 설정된 것
```


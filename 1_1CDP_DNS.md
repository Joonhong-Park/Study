# DNS 설정

Master : adm1

Slave : edge

#### hostname 변경 및 고정 IP 설정

---

- ##### hostname 변경

  ```bash
  hostnamectl set-hostname 변경할호스트명 # 영구 변경, 리부팅 후 적용
  ```

- ##### 고정 IP 설정

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
  # 필요한 DNS만 사용
  DNS1="1.1.1.1" # 클라우드페어 DNS
  DNS2="1.1.1.2"
  DNS3="8.8.8.8" # 구글 DNS
  DNS4="8.8.4.4"
  DNS5="168.126.63.1" # KT DNS
  DNS6="168.126.63.2"
  GATEWAY="xxx.x.xx.xxx" # 게이트웨이
  ```
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
zone "cdp.jh.io" IN {
    type master;
    file "cdp.jh.io.zone";
    allow-update { none; };
};
 
zone "IP주소 역순.in-addr.arpa" IN {
    type master;
    file "cdp.jh.io.rev.zone";
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
zone "cdp.jh.io" IN {
    type slave;
    file "cdp.jh.io.zone";
    masters { adm1 IP주소; };
};
 
zone "IP주소 역순.in-addr.arpa" IN {
    type slave;
    file "cdp.jh.io.rev.zone";
    masters { adm1 IP주소; };
};
...
```



#### zone 작성

------

- master에서만 작성

- hostname은 반드시 .로 끝냄,  _ 사용 금지

##### vim /var/named/cdp.jh.io.zone

```shell
$TTL 6H
; SOA: Indicates authority for this zone
@   IN  SOA  adm1.cdp.jh.io. adm1.cdp.jh.io. ( ; {primary-name-server}  {hostmaster-email}
                                             2020041423 ; serial
                                             1D ; refresh
                                             1H ; retry
                                             1W ; expire
                                             3H ) ; minimum
; NS: Lists a nameserver for this zone
    IN  NS      adm1.cdp.jh.io.
    IN  NS      edge.cdp.jh.io.
    IN  MX  10  adm1.cdp.jh.io.
 
; A: Name-to-address mapping
; PTR: Address-to-name mapping
; CNAME: Canonical name (for aliases)
; Hosts Here
adm1    IN  A   IP Addr
edge    IN  A   IP Addr
hdm1    IN  A   IP Addr
hdm2    IN  A   IP Addr
hdw1    IN  A   IP Addr
hdw2    IN  A   IP Addr
hdw3    IN  A   IP Addr
hdw4    IN  A   IP Addr
 
; kerberos 적용시에만 아래 부분을 추가함
_kerberos                TXT                    "CDP.JH.IO"
kerberos1             IN CNAME                  adm1.cdp.jh.io.
kerberos2             IN CNAME                  edge.cdp.jh.io.
 
; _service._proto.name   TTL class SRV    priority    weight    port    target
_kerberos._udp                     SRV    0           0         88      adm1.cdp.jh.io.
_kerberos-master._udp              SRV    0           0         88      adm1.cdp.jh.io.
_kerberos-adm._tcp                 SRV    0           0         749     adm1.cdp.jh.io.
_kpasswd._udp                      SRV    0           0         464     adm1.cdp.jh.io.
```



#### reverse zone 작성

------

- ##### master에서만 작성

  ##### vim /var/named/cdp.jh.io.rev.zone
  
  ```bash
$TTL 6H
  @   IN  SOA adm1.cdp.jh.io. adm1.cdp.jh.io. (
                                              2020041423 ; serial
                                              1D ; refresh
                                              1H ; retry
                                              1W ; expire
                                              3H ) ; minimum
      IN  NS  adm1.cdp.jh.io.
      IN  NS  edge.cdp.jh.io.
   
  ; IP-Domain mapping here
  IP주소 끝 3자리 IN  PTR adm1.cdp.jh.io.
  IP주소 끝 3자리 IN  PTR edge.cdp.jh.io.
  IP주소 끝 3자리 IN  PTR hdm1.cdp.jh.io.
  IP주소 끝 3자리 IN  PTR hdm2.cdp.jh.io.
  IP주소 끝 3자리 IN  PTR hdw1.cdp.jh.io.
  IP주소 끝 3자리 IN  PTR hdw2.cdp.jh.io.
  IP주소 끝 3자리 IN  PTR hdw3.cdp.jh.io.
  IP주소 끝 3자리 IN  PTR hdw4.cdp.jh.io.
  ```
  
  

#### zone 파일 권한 수정 

------

```bash
chown named:root /var/named/cdp.jh.io.zone
chown named:root /var/named/cdp.jh.io.rev.zone
named-checkconf /etc/named.conf
# 확인
cd /var/named
named-checkzone cdp.jh.io cdp.jh.io.zone
```



#### named 서비스 시작 / 방화벽 해제

------

- ##### master, slave 모두 수행

  ```shell
  systemctl enable named; systemctl start named; systemctl status named
  systemctl stop firewalld; systemctl disable firewalld; systemctl status firewalld
  ```
  
  

#### 네트워크 인터페이스 수정

------

##### vim /etc/sysconfig/network-scripts/ifcfg-eth0

```shell
...
DNS1="Master IP"
DNS2="Slave IP"

...
```

###### 바로 수정하고 확인하는 스크립트

```bash
sed -i 's/DNS1="1.1.1.1"/DNS1="Master IP"/g' /etc/sysconfig/network-scripts/ifcfg-eth0;sed -i 's/DNS2="1.1.1.2"/DNS2="Slave IP"/g' /etc/sysconfig/network-scripts/ifcfg-eth0;cat /etc/sysconfig/network-scripts/ifcfg-eth0
```

##### 네트워크 재시작

```shell
systemctl restart network
```

모든 서버 한번에 재시작 스크립트

```
seq 1 8 | xargs -P 8 -I {} ssh IP주소{} "systemctl restart network; systemctl status network"
```



#### Ping test로 확인

------

```shell
ping -c 1 adm1
ping -c 1 호스트도메인
ping -c 1 IP주소

# 모두 가능해야 제대로 설정된 것
```


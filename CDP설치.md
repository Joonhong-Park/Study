# CDP (클라우데라 데이터 플랫폼)

## 테스트를 위한 사전준비

adm1, edge, hdm[1-2], hdw[1-4]  총 8개의 노드 준비 (CentOS)



#### DNS 구조 변경

------

Master : adm1

Slave : edge

Domain : 그 외

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


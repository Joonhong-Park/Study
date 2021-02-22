# Prerequisites check



#### ansible 설치

```
yum install -y ansible
```



#### prereq-checks-master 설치

```bash
cd ~/Downloads

wget http://www.hadoop-professionals.org/download/cloudera/cdh7/prereq-checks-master.zip

unzip prereq-checks-master.zip
```



#### Hosts 파일 백업 후 수정

##### 백업

```
mv ~/Downloads/prereq-checks-master/ansible/inventory/hosts ~/Downloads/prereq-checks-master/ansible/inventory/hosts_back
```

##### vim ~/Downloads/prereq-checks-master/ansible/inventory/hosts

```
adm1.cdp.jh.io
edge.cdp.jh.io
hdm1.cdp.jh.io
hdm2.cdp.jh.io
hdw1.cdp.jh.io
hdw2.cdp.jh.io
hdw3.cdp.jh.io
hdw4.cdp.jh.io
```



#### check 수행

```
cd ~/Downloads/prereq-checks-master/ansible

ansible-playbook prereq-check.yml
```

##### 수행 결과 확인

```
cat out/adm1.cdp.jh.io.out
cat out/edge.cdp.jh.io.out
cat out/hdm1.cdp.jh.io.out
cat out/hdm2.cdp.jh.io.out
cat out/hdw1.cdp.jh.io.out
cat out/hdw2.cdp.jh.io.out
cat out/hdw3.cdp.jh.io.out
cat out/hdw4.cdp.jh.io.out
```



##### 넘겨도 되는 경고

> - adm1.cdp.jh.io 와 edge.cdp.jh.io 에서는 ntp 를 local time 으로 맞췄기 때문에 FAIL System: ntpd clock NOT synced. Check 'ntpstat' 라는 메시지가 출력될 수 있음
>   -  각 서버마다 ntpstat로 Synchronised 여부를 확인해보자
> - vm 일 경우 BareMetal 이 아니라면서 WARN 출력됨
> - Active Directory, ldap 등을 설치하지 않아서 sssd(Single Sign-on Services Daemon) service 를 실행하지 않았을 경우 WARN 출력됨
> - OpenJDK 1.8.0_242 사용할 경우 WARN 출력됨 → 보다 높은 버전 사용
> - MySQL, MySQL JDBC Driver 설치하지 않았을 경우 WARN 출력됨

##### 그 외의 경고들은 지금까지의 설정 단계에서 오류가 발생한 경우이므로 수정하고 넘어가야 함



##### output

```
PASS  System: /proc/sys/vm/swappiness should be 1
 PASS  System: /proc/sys/vm/overcommit_memory should be 1
 PASS  System: tuned is not running
 PASS  System: tuned does not auto-start on boot
 PASS  System: /sys/kernel/mm/transparent_hugepage/defrag should be disabled
 PASS  System: /sys/kernel/mm/transparent_hugepage/enabled should be disabled
 PASS  System: /etc/default/grub should have 'transparent_hugepage=never' appended to GRUB_CMDLINE_LINUX
 PASS  System: SELinux should be disabled
 PASS  System: ntpd is running
 PASS  System: ntpd auto-starts on boot
 PASS  System: ntpd clock synced
 PASS  System: chronyd is not running
 PASS  System: chronyd does not auto-start on boot
 PASS  System: Only 64bit packages should be installed
 PASS  System: bluetooth is not running
 PASS  System: bluetooth does not auto-start on boot
 PASS  System: cups is not running
 PASS  System: cups does not auto-start on boot
 PASS  System: ip6tables is not running
 PASS  System: ip6tables does not auto-start on boot
 PASS  System: postfix is not running
 PASS  System: postfix does not auto-start on boot
 PASS  System: /tmp mounted with noexec fails for CM versions older than 5.8.4, 5.9.2, and 5.10.0
 PASS  System: Entropy is 3726
 WARN  System: Non BareMetal deployments should follow appropriate Reference Architecures -- Please see https://bit.ly/2CTLeWB
 PASS  Network: IPv6 is not supported and must be disabled
 PASS  Network: Hostname looks good (FQDN, no uppercase letters)
 PASS  Network: /etc/hosts entries should be <= 2 (use DNS). Actual: 2
 PASS  Network: nscd is running
 PASS  Network: nscd auto-starts on boot
 WARN  Network: sssd is not running
 WARN  Network: sssd does not auto-start on boot
 PASS  Network: Consistent name resolution of adm1.cdp.jh.io
 PASS  Network: firewalld is not running
 PASS  Network: firewalld does not auto-start on boot
 PASS  Java: Supported OpenJDK (CDH 5.16.1+ or 6.1.0+ only): /usr/java/default/bin/java
 WARN  Database: MySQL server not installed, skipping version check
 WARN  Database: MySQL JDBC Driver is not installed
```


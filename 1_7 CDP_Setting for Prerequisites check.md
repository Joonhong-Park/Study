# Prerequisites check PASS를 위해 필요한 설정

모든 서버에서 실행함

#### Swap 설정

```bash
echo "vm.swappiness = 1" >> /etc/sysctl.conf
echo 1 > /proc/sys/vm/swappiness
```

###### 한번에 수행

```bash
for i in {1..8} ; do ssh IP주소${i} "echo "vm.swappiness = 1" >> /etc/sysctl.conf" ; done
for i in {1..8} ; do ssh IP주소${i} "echo 1 > /proc/sys/vm/swappiness" ; done
```



#### Overcommit 설정

```bash
echo "vm.overcommit_memory = 1" >> /etc/sysctl.conf
echo 1 > /proc/sys/vm/overcommit_memory
```

```bash
for i in {1..8} ; do ssh IP주소${i} "echo "vm.overcommit_memory = 1" >> /etc/sysctl.conf" ; done
for i in {1..8} ; do ssh IP주소${i} "echo 1 > /proc/sys/vm/overcommit_memory" ; done
```



#### 방화벽 해제

```bash
for i in {1..8} ; do ssh IP주소${i} "systemctl stop firewalld ; systemctl disable firewalld" ; done
```



#### Selinux 중지

**vim /etc/selinux/config**

```bash
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

```bash
for i in {2..8} ; do scp /etc/selinux/config IP주소${i}:/etc/selinux ;done
```



#### postfix 중지

###### centos에는 cups 서비스가 없음

```bash
for i in {1..8} ; do ssh IP주소${i} "systemctl stop cups ; systemctl disable cups ; systemctl stop postfix ; systemctl disable postfix ; systemctl stop tuned ; systemctl disable tuned" ; done
```



#### grub 수정

**vim /etc/default/grub**

```bash
...
GRUB_CMDLINE_LINUX="... transparent_hugepage=never"
...
```

```bash
for i in {2..8} ; do scp /etc/default/grub IP주소${i}:/etc/default ; done
```



#### rc.local 수정

vim /etc/rc.d/rc.local

```bash
touch /var/lock/subsys/local
echo never > /sys/kernel/mm/*transparent_hugepage/defrag
echo never > /sys/kernel/mm/*transparent_hugepage/enabled
```

```bash
for i in {2..8} ; do scp /etc/rc.d/rc.local IP주소${i}:/etc/rc.d ; done
# 권한 변경후 실행
for i in {1..8} ; do ssh IP주소${i} "chmod 755 /etc/rc.d/rc.local ; systemctl enable rc-local ; systemctl start rc-local" ; done
```



#### Network Service Switch 수정

**vim /etc/nsswitch.conf**

```bash
...
hosts: files dns
...
```

```bash
for i in {2..8} ; do scp /etc/nsswitch.conf IP주소${i}:/etc ; done
```



#### host.conf 수정

**vim /etc/host.conf**

```bash
multi on
order hosts, bind
```

```bash
for i in {2..8} ; do scp /etc/host.conf IP주소${i}:/etc ; done
```



#### rngd 설치

> 설치하지 않으면 고른 random number 는 생성되지만 숫자가 낮아서 Cloudera Manager 에서 Warning 이 발생한다

```bash
yum install -y rng-tools
systemctl start rngd
```

```bash
seq 1 8 | xargs -P 8 -I {} ssh IP주소{} "yum install -y rng-tools ; systemctl start rngd"
```



#### bind-utils 설치

```bash
yum install -y bind-utils
dig @<DNSSERVER> <HOSTNAME>
host <IP_ADDRESS>
dig -x <IP_ADDRESS>
nslookup <HOSTNAME>
```

```bash
seq 1 8 | xargs -P 8 -I {} ssh IP주소{} "yum install -y bind-utils"
for i in {1..8} ; do ssh IP주소${i} "dig @adm1 hdw1" ; done
for i in {1..8} ; do ssh IP주소${i} "host IP주소4" ; done
for i in {1..8} ; do ssh IP주소${i} "dig -x IP주소5" ; done
for i in {1..8} ; do ssh IP주소${i} "nslookup hdm1" ; done
```



#### nscd 설치

```bash
yum install -y nscd
systemctl enable nscd
systemctl start nscd
```

```bash
seq 1 8 | xargs -P 8 -I {} ssh IP주소{} "yum install -y nscd ; systemctl enable nscd ; systemctl start nscd"
```



#### Disable IPv6

##### vim /etc/sysctl.conf

```bash
vm.swappiness = 1
vm.overcommit_memory = 1

#disable ipv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

```bash
for i in {2..8} ; do scp /etc/sysctl.conf IP주소${i}:/etc ; done
```



##### vim /etc/sysconfig/network

```bash
NETWORKING_IPV6=no
IPV6INIT=no
```

```bash
for i in {2..8} ; do scp /etc/sysconfig/network IP주소${i}:/etc/sysconfig ; done
```



#### 설정 적용을 위해 전체 재부팅

```bash
for i in {8..1} ; do ssh IP주소${i} "hostname ; reboot" ; done
```


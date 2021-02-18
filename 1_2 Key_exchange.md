SSHPass를 이용한 SSH Key 교환

Install sshpass

```
yum install -y sshpass
```



ssh-key 생성

```bash
ssh-keygen -t rsa -N '' -f ~/.ssh/id_rsa
```



key 교환 

```
export SSHPASS=<root password>
# 한번에 교환하는 방법
for i in {1..8} ; do sshpass -e ssh -o StrictHostKeyChecking=no root@IP주소${i} "mkdir -p ~/.ssh ; chmod 700 ~/.ssh ; touch ~/.ssh/authorized_keys ; echo '$(cat ~/.ssh/id_rsa.pub)' >> ~/.ssh/authorized_keys ; chmod 600 ~/.ssh/authorized_keys" ; done 
```


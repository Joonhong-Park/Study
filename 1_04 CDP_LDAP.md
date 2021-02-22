# LDAP

#### TCP/IP 위에서 디렉터리 서비스를 조회하고 수정하는 응용 프로토콜

##### 트리구조로 구성되며 초기 구성시에는 큰 단위부터, 사용시에는 작은 단위부터 작성함

> DN :  특정한 한 사용자를 식별할 수 있는 이름 
>
> DC : 도메인 컴포넌트, 도메인 주소의 일부 [ ex) dd.io ]
>
> OU : 부서명, 그룹
>
> CN : 일반적인 이름 [ ex) kildong.Hong ]
>
> SN : 사람의 성 [ ex) Lee, Hong .. ]
>
> UID :  유저아이디,  사람의 고유 아이디
>

######  

###### 사용시 예제

​	dn: uid=jhpark,ou=People,ou=Developer,dc=dd,dc=io



#### OpenLdap 설치 (adm1에만 설치함)

```bash
yum install -y openldap-servers openldap-clients
```

```bash
# slapd 서비스 시작
systemctl start slapd
# slapd 서비스 자동 시작
systemctl enable slapd
```



#### 패스워드 등록

```bash
slappasswd

{SSHA}생성된 비밀번호 # 기록해둘것
```



#### Ldap구조 정의 및 업데이트

##### vim db.ldif

```
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=dd,dc=io
 
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=ldaproot,dc=dd,dc=io
 
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}생성된 비밀번호
```

###### ldap server에 update

```bash
ldapmodify -Y EXTERNAL -H ldapi:/// -f db.ldif

# modifying이 되지않을 시 ldif파일에 공백이 있는지 확인해볼 것
```



#### 모니터 접속 계정 설정

**vim monitor.ldif**

```
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=ldaproot,dc=dd,dc=io" read by * none
```

###### ldap server에 update

```
ldapmodify -Y EXTERNAL -H ldapi:/// -f monitor.ldif
```



#### LDAP DB 설정

```
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
chown ldap:ldap /var/lib/ldap/DB_CONFIG
```



#### LDAP 스키마 적용

```
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif;ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif;ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
```



#### 도메인 정보 변경

##### vim base.ldif

```
dn: dc=dd,dc=io
dc: dd
objectClass: top
objectClass: domain
 
dn: cn=ldaproot,dc=dd,dc=io
objectClass: organizationalRole
cn: ldaproot
description: LDAP Manager
 
dn: ou=People,dc=dd,dc=io
objectClass: organizationalUnit
ou: People
 
dn: ou=Group,dc=dd,dc=io
objectClass: organizationalUnit
ou: Group
```



#### 디렉토리 구조 빌드

##### 위에 작성한 구조(base.ldif)를 LDAP에 추가

```
ldapadd -x -W -D "cn=ldaproot,dc=dd,dc=io" -f base.ldif
```



#### 계정생성

##### vim jhpark.ldif

```
# 위에 추가한 People 그룹 밑에 계정 생성
dn: uid=jhpark,ou=People,dc=dd,dc=io
objectClass: top
objectClass: person
objectClass: inetOrgPerson
uid: jhpark
cn: Joonhong
sn: Park
mail: qkrwns394@naver.com
userPassword: {crypt}x
```

##### password 설정

```
ldappasswd -s changeme -W -D "cn=ldaproot,dc=dd,dc=io" -x "uid=jhpark,ou=People,dc=dd,dc=io"
```



#### 계정조회

```
ldapsearch -x cn=Joonhong -b dc=dd,dc=io
```



#### Window에서 Ldap 조회

##### 아파치 디렉토리 스튜디오 > File > New > LDAP Connection 으로 LDAP 연결 후 조회 가능



##### 또한 ldap 프로토콜이므로 ldap://adm1.cdp.jh.io:389/uid=jhpark,ou=People,ou=Developer,dc=dd,dc=io와 같은 방식으로도 확인할 수 있다.


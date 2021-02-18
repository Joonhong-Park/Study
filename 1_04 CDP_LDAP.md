# LDAP

#### TCP/IP 위에서 디렉터리 서비스를 조회하고 수정하는 응용 프로토콜

##### (Ranger LDAP 설정 시 필요하므로 구성도를 기록해둘 것)

> DC : 도메인 컴포넌트, 도메인 주소의 일부
>
> OU : 부서명, 그룹단위
>
> SN : 사람의 성
>
> UID :  유저아이디,  사람의 고유 아이디
>
> DN : 사람 한명을 식별할 수 있는 이름 

######  작은 단위부터 작성

​	dn: uid=jhpark,ou=People,ou=Developer,dc=datadynamics,dc=io



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

{SSHA}WHcFSXAFQRTqZRsrE2OOImgoDfNmB2nc # 기록해둘것
```



#### Ldap구조 정의 및 업데이트

##### vim db.ldif

```
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=datadynamics,dc=io
 
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=ldaproot,dc=datadynamics,dc=io
 
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}WHcFSXAFQRTqZRsrE2OOImgoDfNmB2nc
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
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=ldaproot,dc=datadynamics,dc=io" read by * none
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

```
dn: dc=datadynamics,dc=io
dc: datadynamics
objectClass: top
objectClass: domain
 
dn: cn=ldaproot,dc=datadynamics,dc=io
objectClass: organizationalRole
cn: ldaproot
description: LDAP Manager
 
dn: ou=People,dc=datadynamics,dc=io
objectClass: organizationalUnit
ou: People
 
dn: ou=Group,dc=datadynamics,dc=io
objectClass: organizationalUnit
ou: Group
```



#### 디렉토리 구조 빌드

```
ldapadd -x -W -D "cn=ldaproot,dc=datadynamics,dc=io" -f base.ldif
```



#### 계정생성

```

```



#### 계정조회

```

```


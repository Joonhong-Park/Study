LDAP

TCP/IP 위에서 디렉터리 서비스를 조회하고 수정하는 응용 프로토콜



DC : 도메인 컴포넌트, 도메인 주소의 일부

OU : 부서명, 그룹단위

SN : 사람의 성

UID :  유저아이디,  사람의 고유 아이디

DN : 사람 한명을 식별할 수 있는 이름 



 작은 단위부터 작성

dn: uid=jhpark,ou=People,ou=Developer,dc=datadynamics,dc=io



OpenLdap 설치

```bash
yum install -y openldap
```

```bash
# slapd 서비스 시작
systemctl start slapd
# slapd 서비스 자동 시작
systemctl enable slapd
```

패스워드 등록

```bash
slappasswd

{SSHA}xxxxxxxx # 기록해둘것
```

도메인 환경설정 파일 생성

vim /etc/openldap/domain.ldif

```bash
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=dd,dc=io    #dc 설정 ( dd.io ) 

dn: olcDatabase={2}hdb,cn=config               
changetype: modify
replace: olcRootDN
olcRootDN: cn=ldaproot,dc=dd,dc=io # dc 설정 ( Manager계정 = ldaproot ) 

dn: olcDatabase={2}hdb,cn=config
changetype: modify               // 원본 설정 수정 선언
replace: olcRootPW               // RootPW 설정 선언
olcRootPW: {SSHA}xxxxxxxxxxx #slappassword 에서 만든 패스워드 값 
```

- 환경설정파일에 해당 내용 업로드

  ```bash
  ldapmodify -H ldapi:/// -f domain.ldif
  ```

  ldif 파일에는 공백이 없어야 함

  

  Ldap 모니터링을 관리자만 볼 수 있도록 설정

  ```bash
  dn: olcDatabase={1}monitor,cn=config
  changetype: modify
  replace: olcAccess
  olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=ldaproot,dc=dd,dc=io" read by * none
  ```

  

USER DB 설정

```bash
# ldapuser 디렉토리 설정 템플릿 복사
cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG

chown -R ldap:ldap /var/lib/ldap # 해당 폴더 ldap 소유 변경
```



기본 디렉토리 DB 구조 추가

```
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldifSASL/EXTERNALauthentication started
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cnis.ldif
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetOrgPerson
```



각 부서(디렉토리) 구분 스키마 업데이트

vim 

```
dn: dc=dd,dc=io
dc: dd
objectClass: top
objectClass: domain

dn: cn=ldaproot,dc=dd,dc=io
objectClass: organizationalRole
cn: ldaproot
description: LDAP Manager

dn: ou=hq,dc=dd,dc=io
objectClass: organizationalUnit
ou:hq

dn: ou=Group,dc=dd,dc=io
objectClass: organizationalUnit
ou:Group

dn: ou=People,dc=dd,dc=io
objectClass: organizationalUnit
ou:People
```


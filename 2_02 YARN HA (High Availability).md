# YARN HA (High Availability)

#### YARN 동작 구조

![yarn_architecture](\image\yarn_architecture.png)



hadoop 2.4 전에는 Yarn Resource Manager 역할을 하는 서버가 죽으면

Yarn 시스템 전체가 뻗어버리는 문제점이 있었음

하지만 2.4 이후 부터는 Yarn RM에도 HA가 지원되어 이 문제를 해결하였음



##### Cloudera Manager > 클러스터 > YARN (MR2 included) > 작업 > High Availability 설정



#### 시작하기

Resource Manager : hdm1



#### 완료![YARN_HA](\image\YARN_HA.PNG)
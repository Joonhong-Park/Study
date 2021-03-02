# MapReduce

데이터 처리 솔루션 중 하나, Hadoop 뿐만 아닌 일반 DB에도 존재하는 개념



Map + Reduce 두 단계로 데이터를 처리

- Map : 특정 데이터를 가져와 <Key, Value>쌍으로 묶음
- Reduce : Map에서 묶은 <Key, Value>쌍을 이용하여 내가 필요한 정보로 다시 <Key, Value>쌍으로 묶음



MR 작업을 위해 필요한 3가지 Class

1. Map 
2. Reduce
3. Driver : Map, Reduce를 실행시켜줄 main함수가 작성된 곳







jar 생성 방법

Maven > Lifecycle > clean > package





scp <jar Path> root@hdm2.cdp.jh.io



 HADOOP_USER_NAME=hdfs yarn jar mr.jar io.datadynamics.bigdata.mapreduce.sample.WordCountDriver -input /tmp/chat/input -output /tmp/chat/output -delimiter "\t" -reducer 4





기본적으로 파일 1개당 1개의 map이 생성되나, 파일 size가 block size를 넘을 경우 맵을 추가로 생성한다.

하지만 gzip은 파일 size가 block size를 넘어도 맵은 하나만 생긴다.

이는 gzip이 DEFLATE 알고리즘을 사용하여 gzip 스트림이 각 block split이 특정 위치에서 읽기를 지원하지 않으므로 mapreduce 과정에서 각 block별로 split을 생성할 수 없기 때문임



압축포맷별 split 가능 여부

| 압축포맷 | 도구  | 알고리즘 | 파일 확장명 | 분할 가능 |
| -------- | ----- | -------- | ----------- | --------- |
| DEFLATE  | N/A   | DEFLATE  | .deflate    | No        |
| gzip     | gzip  | DEFLATE  | .gz         | No        |
| bzip2    | bzip2 | bzip2    | .bz2        | Yes       |
| LZO      | lzop  | LZO      | .lz0        | No        |
| LZ4      | N/A   | LZ4      | .lz4        | No        |
| Snappy   | N/A   | Snappy   | .snappy     | No        |


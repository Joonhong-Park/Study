# MapReduce

데이터 처리 솔루션 중 하나, Hadoop 뿐만 아닌 일반 DB에도 존재하는 개념



Map + Reduce 두 단계로 데이터를 처리

- Map : 특정 데이터를 가져와 <Key, Value>쌍으로 묶음
- Reduce : Map에서 묶은 <Key, Value>쌍을 이용하여 내가 필요한 정보로 다시 <Key, Value>쌍으로 묶음



MR 작업을 위해 필요한 3가지 Class

1. Map 
2. Reduce
3. Driver : Map, Reduce를 실행시켜줄 main함수가 작성된 곳



![](./image/mapreduce.PNG)



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



-- 정리 필요 --

yarn app -kill application ID



yarn jar hadoop-mapreduce-examples-3.1.1.3.0.1.0-187.jar terasort -Dmapreduce.map.memory.mb=1024 -Dmapreduce.reduce.memory.mb=1024 -Dmapreduce.terasort.num-rows=100000000 -Dmapreduce.terasort.num.partitions=4 /tmp/teragen /tmp/terasort\


yarn jar hadoop-mapreduce-client-jobclient-3.1.1.7.1.4.0-203-tests.jar TestDFSIO -Dmapreduce.map.memory.mb=1536 -Dmapreduce.reduce.memory.mb=1536 -write -nrFiles 4 -size 384MB

yarn jar hadoop-mapreduce-client-jobclient-3.1.1.7.1.4.0-203-tests.jar TestDFSIO -read -nrFiles 4 -size 384MB

yarn -> am -> application memory -> 1g -> 256mb

컨테이너 메모리
yarn.nodemanager.resource.memory-mb -> 2g 

최대 컨테이너 메모리 ( mapreduce에서 사용할 용량)
yarn.scheduler.maximum-allocation-mb-> 2g

nodemanager memory - aplication  = 가용가능 한 메모리

ex> datanode가 4개이고 nodemanager가 서버당 가용메모리를 2g로 할당하면 ,총 가용가능한 메모리 = 8g,
여기서 application메모리로 1g를 사용하면, 테스트에 사용할 수 있는 총 메모리는 7g가 됨


테스트할 datanode disk 갯수가 4이고 size가 1g이면 테스트에는 총 4g만 사용하게되므로 남은 3g에 대한 테스트는 이루어지지않는 것과 같은 결과가 됨
그러므로 TestDFSIO를 할때는 사용가능메모리, 테스트할메모리 들을 잘 계산하여 사용할 필요가 있음  

21/03/03 15:25:16 INFO fs.TestDFSIO: ----- TestDFSIO ----- : write
21/03/03 15:25:16 INFO fs.TestDFSIO:             Date & time: Wed Mar 03 15:25:16 KST 2021
21/03/03 15:25:16 INFO fs.TestDFSIO:         Number of files: 4
21/03/03 15:25:16 INFO fs.TestDFSIO:  Total MBytes processed: 1536
21/03/03 15:25:16 INFO fs.TestDFSIO:       Throughput mb/sec: 60.85
21/03/03 15:25:16 INFO fs.TestDFSIO:  Average IO rate mb/sec: 73.79
21/03/03 15:25:16 INFO fs.TestDFSIO:   IO rate std deviation: 38.74
21/03/03 15:25:16 INFO fs.TestDFSIO:      Test exec time sec: 43.23

21/03/03 15:26:12 INFO fs.TestDFSIO: ----- TestDFSIO ----- : read
21/03/03 15:26:12 INFO fs.TestDFSIO:             Date & time: Wed Mar 03 15:26:12 KST 2021
21/03/03 15:26:12 INFO fs.TestDFSIO:         Number of files: 4
21/03/03 15:26:12 INFO fs.TestDFSIO:  Total MBytes processed: 1536
21/03/03 15:26:12 INFO fs.TestDFSIO:       Throughput mb/sec: 1590.06
21/03/03 15:26:12 INFO fs.TestDFSIO:  Average IO rate mb/sec: 1720.92
21/03/03 15:26:12 INFO fs.TestDFSIO:   IO rate std deviation: 406.85
21/03/03 15:26:12 INFO fs.TestDFSIO:      Test exec time sec: 19.25

블록사이즈를 다르게 주면 읽음
하지만 성능이 많이 저하됨
21/03/03 15:28:11 INFO fs.TestDFSIO: ----- TestDFSIO ----- : read
21/03/03 15:28:11 INFO fs.TestDFSIO:             Date & time: Wed Mar 03 15:28:11 KST 2021
21/03/03 15:28:11 INFO fs.TestDFSIO:         Number of files: 4
21/03/03 15:28:11 INFO fs.TestDFSIO:  Total MBytes processed: 1024
21/03/03 15:28:11 INFO fs.TestDFSIO:       Throughput mb/sec: 714.09
21/03/03 15:28:11 INFO fs.TestDFSIO:  Average IO rate mb/sec: 893.1
21/03/03 15:28:11 INFO fs.TestDFSIO:   IO rate std deviation: 480.42

21/03/03 15:28:11 INFO fs.TestDFSIO:      Test exec time sec: 20.89

파일갯수를 다르게 주면 fail

read를 위해선 write가 선행되어야함을 확인



SelectAllMapper.java

```
/*
 * Licensed to the Apache Software Foundation (ASF) under one
 * or more contributor license agreements.  See the NOTICE file
 * distributed with this work for additional information
 * regarding copyright ownership.  The ASF licenses this file
 * to you under the Apache License, Version 2.0 (the
 * "License"); you may not use this file except in compliance
 * with the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package io.datadynamics.bigdata.mapreduce.sample;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * Wordcount Mapper
 *
 * @author Data Dynamics
 * @version 0.1
 */
public class SelectAllMapper extends Mapper<LongWritable, Text, NullWritable, Text> {

    private String delimiter;

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        Configuration configuration = context.getConfiguration();
        delimiter = configuration.get("delimiter", "\t");
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
         String row = value.toString();
        // String[] columns = row.split(delimiter);
        context.write(NullWritable.get(),new Text(row));
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
    }
}
```



selectAllDriver.java

```
/*
s * Licensed to the Apache Software Foundation (ASF) under one
 * or more contributor license agreements.  See the NOTICE file
 * distributed with this work for additional information
 * regarding copyright ownership.  The ASF licenses this file
 * to you under the Apache License, Version 2.0 (the
 * "License"); you may not use this file except in compliance
 * with the License.  You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package io.datadynamics.bigdata.mapreduce.sample;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.util.ToolRunner;

import java.io.IOException;

/**
 * Wordcount 예제.
 *
 * @author Data Dynamics
 * @version 0.1
 */
public class SelectAllDriver extends org.apache.hadoop.conf.Configured implements org.apache.hadoop.util.Tool {

    public static void main(String[] args) throws Exception {
        int res = ToolRunner.run(new SelectAllDriver(), args);
        System.exit(res);
    }

    public int run(String[] args) throws Exception {
        GenericOptionsParser parser = new GenericOptionsParser(this.getConf(), args);
        String[] remainingArgs = parser.getRemainingArgs();
        Job job = Job.getInstance(this.getConf());
        parseArguments(remainingArgs, job);

        job.setJarByClass(SelectAllDriver.class);

        // Mapper & Reducer Class
        job.setMapperClass(SelectAllMapper.class);

        // Mapper Output Key & Value Type after Hadoop 0.20
        job.setMapOutputKeyClass(NullWritable.class);
        job.setMapOutputValueClass(Text.class);

        // Run a Hadoop Job
        return job.waitForCompletion(true) ? 0 : 1;
    }

    private void parseArguments(String[] args, Job job) throws IOException {
        for (int i = 0; i < args.length; ++i) {
            if ("-input".equals(args[i])) {
                FileInputFormat.addInputPaths(job, args[++i]);
            } else if ("-output".equals(args[i])) {
                FileOutputFormat.setOutputPath(job, new Path(args[++i]));
            }
        }
    }
}
```


# SQL을 MapReduce로 구현

sql문의 출력 결과와 똑같은 결과를 얻는 것을 목표로 하며 실제 sql과는 차이가 있음



## 1. select * from table

hdfs파일을 읽어 모든 출력 내용을 output으로 저장하는 것을 목표로 함

이때, Mapper로 읽은 내용을 그대로 저장하면 되므로 Reducer가 필요 없음

> Setup과 CleanUp 과정은 Mapper 또는 Reducer 호출 시 시작과 끝에 한번식 호출되며
>
> Map, Reduce 과정은 **key 개수 만큼 반복**됨
>
> 이점에 유의하여 작성할 것



##### Mapper

```java
package jh.hadoop.mapreduce.sample;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

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

map()에서 hdfs파일을 한줄 씩 읽어들임

이때 Key는 Longwritable, value는 Text형으로 읽어오며 이 값은 따로 설정해주지 않는 이상 고정으로 사용됨

key의 경우, pos값을 읽어오게 되며 이는 Row Number에 해당됨

value는 /n까지 읽은 내용들에 해당 됨



Select * 은 읽어들인 모든 내용을 Key값으로 Filter할 필요없이 그대로 출력해주면 되므로 

Key는 NullWritable로, value는 읽어들인 한 줄로 설정하였음

이때 Output의 형은 상속받은 Mapper의 형과 일치 시켜줘야 함



##### Driver

```java
package jh.hadoop.mapreduce.sample;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.util.ToolRunner;

import java.io.IOException;

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

Reducer 과정이 필요없으므로 Job에는 Mapper,MapOutput key, value Class만 설정해주면 된다.

이때, ReduceOutput이 없으므로 Mapper에서의 Output이 최종 결과물이 됨



## 2. select * from table where a = 'v'

hdfs 파일에서 특정 값 'v'가 있는 row만 output으로 저장하는 것을 목표로 함



##### Mapper

읽은 데이터에서 찾을 위치와 찾을 내용을 특정하기 위해 where, what 변수를 설정해주었음

where, what 파라매터는 Driver에서 받아옴

```java
package jh.hadoop.mapreduce.sample;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class SelectWhereMapper extends Mapper<LongWritable, Text, NullWritable, Text> {

    private String delimiter;
    private String where;
    private String what;
    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        Configuration configuration = context.getConfiguration();
        delimiter = configuration.get("delimiter", ",");
        where = configuration.get("where", "0");
        what = configuration.get("what");
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String row = value.toString();
        String[] columns = row.split(delimiter);
        int index = Integer.parseInt(where);
        if (columns[index].equals(what)){
            context.write(NullWritable.get(),new Text(row));
        }
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
    }
}
```



##### Driver

```java
package jh.hadoop.mapreduce.sample;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.util.ToolRunner;

import java.io.IOException;

public class SelectWhereDriver extends org.apache.hadoop.conf.Configured implements org.apache.hadoop.util.Tool {

    public static void main(String[] args) throws Exception {
        int res = ToolRunner.run(new SelectWhereDriver(), args);
        System.exit(res);
    }


    public int run(String[] args) throws Exception {
        GenericOptionsParser parser = new GenericOptionsParser(this.getConf(), args);
        String[] remainingArgs = parser.getRemainingArgs();
        Job job = Job.getInstance(this.getConf());
        parseArguments(remainingArgs, job);

        job.setJarByClass(SelectWhereDriver.class);

        // Mapper & Reducer Class
        job.setMapperClass(SelectWhereMapper.class);

        // Mapper Output Key & Value Type after Hadoop 0.20
        job.setMapOutputKeyClass(NullWritable.class);
        job.setMapOutputValueClass(Text.class);

        // set reduce task 0
        job.setNumReduceTasks(0);

        // Run a Hadoop Job
        return job.waitForCompletion(true) ? 0 : 1;
    }

    private void parseArguments(String[] args, Job job) throws IOException {
        for (int i = 0; i < args.length; ++i) {
            if ("-input".equals(args[i])) {
                FileInputFormat.addInputPaths(job, args[++i]);
            } else if ("-output".equals(args[i])) {
                FileOutputFormat.setOutputPath(job, new Path(args[++i]));
            } else if ("-where".equals(args[i])) {
                job.getConfiguration().set("where", args[++i]);
            } else if ("-delimiter".equals(args[i])) {
                job.getConfiguration().set("delimiter", args[++i]);
            } else if("-what".equals(args[i])){
                job.getConfiguration().set("what", args[++i]);
            }
        }
    }
}
```



## 3. select a, count(*)  from table where a = 'v'

hdfs 파일에서 특정 값 'v'가 있는 row의 수를 count하여 output으로 저장하는 것을 목표로 함

row 수를 count하여 출력해줄 Reducer가 필요



##### Mapper

찾을려는 값 (= what) 을 찾을 때마다 <what, 1>을 context에 write함

```java
package jh.hadoop.mapreduce.sample;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class WhereCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {

    private String delimiter;
    private String where;
    private String what;
    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        Configuration configuration = context.getConfiguration();
        delimiter = configuration.get("delimiter", ",");
        where = configuration.get("where", "0");
        what = configuration.get("what");
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String row = value.toString();
        String[] columns = row.split(delimiter);
        int index = Integer.parseInt(where);
        context.write(new Text(what), new IntWritable(1));
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
    }
}
```



##### Reducer

Mapper의 context에 저장된 내용은 <what, [1,1,1,1,1,1 ... ]> 과 같은 형태로 Reducer로 전송되며

Reducer에서는 1,1,1,1 ... 을 Sum하는 방식으로 Count를 하게 됨

```java
package jh.hadoop.mapreduce.sample;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;
import java.util.Iterator;

public class WhereCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {

    }

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        Iterator<IntWritable> iterator = values.iterator();
        int sum = 0;
        while (iterator.hasNext()) {
            IntWritable one = iterator.next();
            sum += one.get();
        }
        context.write(key, new IntWritable(sum));
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
    }
}
```



##### Driver

```java
package jh.hadoop.mapreduce.sample;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.util.ToolRunner;

import java.io.IOException;

public class WhereCountDriver extends org.apache.hadoop.conf.Configured implements org.apache.hadoop.util.Tool {

    public static void main(String[] args) throws Exception {
        int res = ToolRunner.run(new WhereCountDriver(), args);
        System.exit(res);
    }

    public int run(String[] args) throws Exception {
        GenericOptionsParser parser = new GenericOptionsParser(this.getConf(), args);
        String[] remainingArgs = parser.getRemainingArgs();
        Job job = Job.getInstance(this.getConf());
        parseArguments(remainingArgs, job);

        job.setJarByClass(WhereCountDriver.class);

        // Mapper & Reducer Class
        job.setMapperClass(WhereCountMapper.class);
        job.setReducerClass(WhereCountReducer.class);

        // Mapper Output Key & Value Type after Hadoop 0.20
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        // Reducer Output Key & Value Type
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        // Run a Hadoop Job
        return job.waitForCompletion(true) ? 0 : 1;
    }

    private void parseArguments(String[] args, Job job) throws IOException {
        for (int i = 0; i < args.length; ++i) {
            if ("-input".equals(args[i])) {
                FileInputFormat.addInputPaths(job, args[++i]);
            } else if ("-output".equals(args[i])) {
                FileOutputFormat.setOutputPath(job, new Path(args[++i]));
            } else if ("-where".equals(args[i])) {
                job.getConfiguration().set("where", args[++i]);
            } else if ("-delimiter".equals(args[i])) {
                job.getConfiguration().set("delimiter", args[++i]);
            } else if("-what".equals(args[i])){
                job.getConfiguration().set("what", args[++i]);
            }
        }
    }
}
```

Reducer를 작성했으므로 Mapoutput은 Reducer의 Input값으로 들어가게 되며, setOutput Key/Value를 job에 추가해줘야 Reducer로부터 결과를 받아와 저장할 수 있음



## 4. select a, array_agg(b) from table group by a

array_agg : 해당되는 값들을 배열로 출력해줌

여기에서는 일정 방송 기간마다 만들어진 방송 제목을 배열로 만들어 출력하는 것을 목표로 함

또한 btime 파라매터를 받아오면 해당 시간의 제목들만, 그렇지않으면 모든 기간의 제목들을 가져오도록 하였음

value에 해당하는 값들을 배열로 만들어주는 과정은 Reducer, Mapper에서 모두 수행 가능함



#### 1. Reducer단계에서 array_agg 수행

------

##### Mapper

방송시작시간(btime)을 받아오면 해당 기간의 방송 제목들만, 그렇지않으면 모든 기간의 방송 제목들을 < 방송시작시간, 방송 제목 > 형태로 Reducer로 전송함

```java
package jh.hadoop.mapreduce.sample.ArrayAgg;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class ArrayAggMapper extends Mapper<LongWritable, Text, Text, Text> {

    private String delimiter;
    private String btime;
    private int index;

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        Configuration configuration = context.getConfiguration();
        delimiter = configuration.get("delimiter", ",");
        btime = configuration.get("btime");
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String row = value.toString();
        String[] columns = row.split(delimiter);
        if (btime.isEmpty()){
            context.write(new Text(columns[3]), new Text((columns[2])));
        }else{
            if(columns[3].equals(btime)){
                context.write(new Text(columns[3]), new Text((columns[2])));
            }
        }
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
    }
}
```



##### Reducer

중복되는 방송 제목들을 제거하기 위해 중복을 허용하지 않는 특성이 있는 HashSet을  사용하였음

HashSet, HashMap과 같은 자료구조는 메모리를 많이 잡아먹는 문제가 있으므로 다룬 데이터의 양이 크다면 쓰지 않는 것이 바람직하다.

또한 Set을 바로 출력해주면 배열과 같은 형태로 출력된다.

```java
package jh.hadoop.mapreduce.sample.ArrayAgg;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;
import java.util.HashSet;
import java.util.Iterator;

public class ArrayAggReducer extends Reducer<Text, Text, Text, Text> {
    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
    }

    @Override
    protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        Iterator<Text> iterator = values.iterator();
        HashSet<String> set = new HashSet<>();
        while (iterator.hasNext()) {
            Text one = iterator.next();
            set.add(one.toString());
        }
        context.write(key, new Text(set.toString()));
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
    }
}
```



##### Driver

```java
package jh.hadoop.mapreduce.sample.ArrayAgg;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.util.ToolRunner;

import java.io.IOException;

public class ArrayAggDriver extends org.apache.hadoop.conf.Configured implements org.apache.hadoop.util.Tool {

    public static void main(String[] args) throws Exception {
        int res = ToolRunner.run(new ArrayAggDriver(), args);
        System.exit(res);
    }

    public int run(String[] args) throws Exception {
        GenericOptionsParser parser = new GenericOptionsParser(this.getConf(), args);
        String[] remainingArgs = parser.getRemainingArgs();
        Job job = Job.getInstance(this.getConf());
        parseArguments(remainingArgs, job);

        job.setJarByClass(ArrayAggDriver.class);

        // Mapper & Reducer Class
        job.setMapperClass(ArrayAggMapper.class);
        job.setReducerClass(ArrayAggReducer.class);

        // Mapper Output Key & Value Type after Hadoop 0.20
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);

        // Reducer Output Key & Value Type
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        // Run a Hadoop Job
        return job.waitForCompletion(true) ? 0 : 1;
    }

    private void parseArguments(String[] args, Job job) throws IOException {
        for (int i = 0; i < args.length; ++i) {
            if ("-input".equals(args[i])) {
                FileInputFormat.addInputPaths(job, args[++i]);
            } else if ("-output".equals(args[i])) {
                FileOutputFormat.setOutputPath(job, new Path(args[++i]));
            } else if ("-delimiter".equals(args[i])) {
                job.getConfiguration().set("delimiter", args[++i]);
            } else if ("-btime".equals(args[i])) {
                job.getConfiguration().set("btime", args[++i]);
            } else if ("-reducer".equals(args[i])) {
                job.setNumReduceTasks(Integer.parseInt(args[++i]));
            }
        }
    }
}
```



#### 2. Mapper단계에서 array_agg 수행

------

##### Mapper



```java
package jh.hadoop.mapreduce.sample.ArrayAgg;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;
import java.util.ArrayList;
import java.util.HashMap;

public class ArrayAggByMapMapper extends Mapper<LongWritable, Text, Text, Text> {

    private String delimiter;
    private String btime;
    private HashMap<String, ArrayList<String>> map = new HashMap<>();// b_Start_time, b_title


    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        Configuration configuration = context.getConfiguration();
        delimiter = configuration.get("delimiter", ",");
        btime = configuration.get("btime");
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String row = value.toString();
        String[] columns = row.split(delimiter);
        if (map.containsKey(columns[3])){
            if(!map.get(columns[3]).contains(columns[2])){
                map.get(columns[3]).add(columns[2]);
                context.write(new Text(columns[3]), new Text(columns[2]));
            }
        }else{
            map.put(columns[3], new ArrayList<String>());
            map.get(columns[3]).add(columns[2]);
            context.write(new Text(columns[3]), new Text(columns[2]));
        }
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
    }
}
```



##### Reducer

```java
package jh.hadoop.mapreduce.sample.ArrayAgg;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;
import java.util.Iterator;

public class ArrayAggByMapReducer extends Reducer<Text, Text, Text, Text> {
    @Override
    protected void setup(Context context) throws IOException, InterruptedException {

    }

    @Override
    protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        Iterator<Text> iterator = values.iterator();
        String output = "[";
        while (iterator.hasNext()) {
            Text one = iterator.next();
            output += one.toString() + ",";
        }
        output += "]";
        context.write(key, new Text(output));
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
    }
}
```



##### Driver

```java
package jh.hadoop.mapreduce.sample.ArrayAgg;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.util.ToolRunner;

import java.io.IOException;

public class ArrayAggByMapDriver extends org.apache.hadoop.conf.Configured implements org.apache.hadoop.util.Tool {

    public static void main(String[] args) throws Exception {
        int res = ToolRunner.run(new ArrayAggByMapDriver(), args);
        System.exit(res);
    }

    public int run(String[] args) throws Exception {
        GenericOptionsParser parser = new GenericOptionsParser(this.getConf(), args);
        String[] remainingArgs = parser.getRemainingArgs();
        Job job = Job.getInstance(this.getConf());
        parseArguments(remainingArgs, job);

        job.setJarByClass(ArrayAggByMapDriver.class);

        // Mapper & Reducer Class
        job.setMapperClass(ArrayAggByMapMapper.class);
        job.setReducerClass(ArrayAggByMapReducer.class);

        // Mapper Output Key & Value Type after Hadoop 0.20
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);

        // Reducer Output Key & Value Type
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        // Run a Hadoop Job
        return job.waitForCompletion(true) ? 0 : 1;
    }

    private void parseArguments(String[] args, Job job) throws IOException {
        for (int i = 0; i < args.length; ++i) {
            if ("-input".equals(args[i])) {
                FileInputFormat.addInputPaths(job, args[++i]);
            } else if ("-output".equals(args[i])) {
                FileOutputFormat.setOutputPath(job, new Path(args[++i]));
            } else if ("-delimiter".equals(args[i])) {
                job.getConfiguration().set("delimiter", args[++i]);
            } else if ("-btime".equals(args[i])) {
                job.getConfiguration().set("btime", args[++i]);
            } else if ("-reducer".equals(args[i])) {
                job.setNumReduceTasks(Integer.parseInt(args[++i]));
            }
        }
    }
}
```



## 5. Inner Join

Ranker ID List 파일과 Chatting Data 파일을 Inner Join하여 Ranker의 Chatting data만 출력하는 것을 목표로 함

아래의 두 방식은 직접 작성해본 예제일 뿐이며 실제로 Join을 구현하는 방법은 매우 다양함



#### 1. Flag in value + Combiner in Mapper 방식

------

##### Mapper1 (Ranker ID List File)

```java
package jh.hadoop.mapreduce.sample.Join;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class JoinMapper1 extends Mapper<LongWritable, Text, Text, Text> {

    private String delimiter;

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        Configuration configuration = context.getConfiguration();
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String row = value.toString();
        context.write(new Text(row), new Text("R^"));
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
    }
}
```



##### Mapper2 (Chatting Data File)

```java
package jh.hadoop.mapreduce.sample.Join;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;
import jh.hadoop.mapreduce.ChatLog;

import java.io.IOException;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

public class JoinMapper2 extends Mapper<LongWritable, Text, Text, Text> {

    private String delimiter;
    HashMap<String, String> chat_data = new HashMap<>();

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        Configuration configuration = context.getConfiguration();
        delimiter = configuration.get("delimiter", "\t");
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String row = value.toString();
        String[] columns = row.split(delimiter);
        String chat_id = columns[ChatLog.user_id.ordinal()];
        String chat = columns[ChatLog.chat_text.ordinal()];

        if(!chat_data.containsKey(chat_id)){
            chat_data.put(chat_id, "C^"+chat);
        }else{
            String prev_data = chat_data.get(chat_id);
            chat_data.put(chat_id, prev_data + "|" + chat);
        }
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {

        for (Map.Entry<String, String> entry : chat_data.entrySet()) {
            String user_id = entry.getKey();
            String chat_all = entry.getValue();

            context.write(new Text(user_id), new Text(chat_all));
        }
    }
}
```



##### Reducer

```java
package jh.hadoop.mapreduce.sample.Join;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.yarn.webapp.hamlet2.Hamlet;
import org.eclipse.jetty.webapp.MetaDataComplete;

import java.io.IOException;
import java.io.StreamCorruptedException;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.concurrent.TransferQueue;

import static org.eclipse.jetty.webapp.MetaDataComplete.True;

public class JoinReducer extends Reducer<Text, Text, Text, Text> {

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
    }

    @Override
    protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        // key: ranker_id, value: ""
        // key: user_id, value: chat_text[]
        int isR = 0;
        StringBuilder chat_all = new StringBuilder();

        for (Text value : values) {
            String chat_data = value.toString();
            String check = chat_data.substring(0, 2);
            if (check.equals("R^")) {
                isR = 1;
            } else {
                chat_all.append(chat_data.substring(2));
            }
        }
        if (isR == 1 && chat_all.length() >= 1) {
            context.write(key, new Text("|"+chat_all.toString()));
        }
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
    }
}
```



##### Driver

```java
package jh.hadoop.mapreduce.sample.Join;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.MultipleInputs;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.util.ToolRunner;

import java.io.IOException;

public class JoinDriver extends org.apache.hadoop.conf.Configured implements org.apache.hadoop.util.Tool {

    public static void main(String[] args) throws Exception {
        int res = ToolRunner.run(new JoinDriver(), args);
        System.exit(res);
    }

    public int run(String[] args) throws Exception {
        GenericOptionsParser parser = new GenericOptionsParser(this.getConf(), args);
        String[] remainingArgs = parser.getRemainingArgs();
        Job job = Job.getInstance(this.getConf());

        parseArguments(remainingArgs, job);

        job.setJarByClass(JoinDriver.class);

        // Mapper & Reducer Class

        job.setReducerClass(JoinReducer.class);

        // Mapper Output Key & Value Type after Hadoop 0.20
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);

        // Reducer Output Key & Value Type
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        // Run a Hadoop Job
        return job.waitForCompletion(true) ? 0 : 1;
    }

    private void parseArguments(String[] args, Job job) throws IOException {
        for (int i = 0; i < args.length; ++i) {
            if ("-inputone".equals(args[i])) {
                MultipleInputs.addInputPath(job, new Path(args[++i]), TextInputFormat.class, JoinMapper1.class);
            } else if ("-inputtwo".equals(args[i])) {
                MultipleInputs.addInputPath(job, new Path(args[++i]), TextInputFormat.class, JoinMapper2.class);
            } else if ("-output".equals(args[i])) {
                FileOutputFormat.setOutputPath(job, new Path(args[++i]));
            } else if ("-delimiter".equals(args[i])) {
                job.getConfiguration().set("delimiter", args[++i]);
            } else if ("-reducer".equals(args[i])) {
                job.setNumReduceTasks(Integer.parseInt(args[++i]));
            }
        }
    }
}
```



#### 2. Flag in key  + Partitioner 방식

------



##### Mapper1 (Ranker ID List File)

```
package jh.hadoop.mapreduce.sample.Join;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class JoinByKeyMapper1 extends Mapper<LongWritable, Text, Text, Text> {

    private String delimiter;

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        Configuration configuration = context.getConfiguration();
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String row = value.toString();
        context.write(new Text(row + "^A"), new Text(""));
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
    }
}
```



##### Mapper2 (Chatting Data File)

```java
package jh.hadoop.mapreduce.sample.Join;

import jh.hadoop.mapreduce.ChatLog;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class JoinByKeyMapper2 extends Mapper<LongWritable, Text, Text, Text> {

    private String delimiter;
    HashMap<String, String> chat_data = new HashMap<>();

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        Configuration configuration = context.getConfiguration();
        delimiter = configuration.get("delimiter", "\t");
    }

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String row = value.toString();
        String[] columns = row.split(delimiter);
        String chat_id = columns[ChatLog.user_id.ordinal()];
        String chat = columns[ChatLog.chat_text.ordinal()];

        context.write(new Text(chat_id + "^B"), new Text(chat));
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
    }
}
```



##### Partitioner

```java
package jh.hadoop.mapreduce.sample.Join;

import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;

public class JoinByKeyPartitioner extends Partitioner<Text, Text> {
    @Override
    public int getPartition(Text key, Text value, int reducerNum) {
        String id = key.toString().split("\\^")[0];
        return Math.abs(id.hashCode()) % reducerNum;
    }
}
```



##### Reducer

```java
package jh.hadoop.mapreduce.sample.Join;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.output.MultipleOutputs;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;

public class JoinByKeyReducer extends Reducer<Text, Text, Text, Text> {
    private static final Logger logger = LoggerFactory.getLogger(JoinByKeyReducer.class);

    private String pre_flag;
    private StringBuilder chat_all;
    private String pre_id;

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        pre_flag = "";
        pre_id = "";
        chat_all = new StringBuilder();
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
    }

    @Override
    protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        chat_all.setLength(0);
        String[] flag = key.toString().split("\\^");
        String id = flag[0];
        String type = flag[1];

        switch (type) {
            case "A":
                pre_flag = "A";
                pre_id = id;
                break;
            default:
                if (pre_flag.equals("A") && pre_id.equals(id)) {
                    for (Text value : values) {
                        String chat_data = value.toString();
                        chat_all.append(chat_data).append("|");
                        context.getCounter("CUSTOM", "total chat").increment(1);
                    }
                    Text userId = new Text(id);
                    String chats = chat_all.substring(0, chat_all.length() - 1);
                    if (chats.isEmpty()) {
                        logger.info("chats = {}", chats);
                    }
                    Text value = new Text(chats);
                    context.write(userId, value);
                    context.getCounter("CUSTOM", "reduce mo write").increment(1);
                    pre_flag = "";
                    pre_id = "";
                }
                break;
        }
    }
}
```



##### Driver

```java
package jh.hadoop.mapreduce.sample.Join;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.MultipleInputs;
import org.apache.hadoop.mapreduce.lib.input.TextInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.GenericOptionsParser;
import org.apache.hadoop.util.ToolRunner;

import java.io.IOException;

public class JoinDriver extends org.apache.hadoop.conf.Configured implements org.apache.hadoop.util.Tool {

    public static void main(String[] args) throws Exception {
        int res = ToolRunner.run(new JoinDriver(), args);
        System.exit(res);
    }

    public int run(String[] args) throws Exception {
        GenericOptionsParser parser = new GenericOptionsParser(this.getConf(), args);
        String[] remainingArgs = parser.getRemainingArgs();
        Job job = Job.getInstance(this.getConf());

        parseArguments(remainingArgs, job);

        job.setJarByClass(JoinDriver.class);

        // Mapper & Reducer Class

        job.setReducerClass(JoinReducer.class);

        // Mapper Output Key & Value Type after Hadoop 0.20
        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(Text.class);

        // Reducer Output Key & Value Type
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        // Run a Hadoop Job
        return job.waitForCompletion(true) ? 0 : 1;
    }

    private void parseArguments(String[] args, Job job) throws IOException {
        for (int i = 0; i < args.length; ++i) {
            if ("-inputone".equals(args[i])) {
                MultipleInputs.addInputPath(job, new Path(args[++i]), TextInputFormat.class, JoinMapper1.class);
            } else if ("-inputtwo".equals(args[i])) {
                MultipleInputs.addInputPath(job, new Path(args[++i]), TextInputFormat.class, JoinMapper2.class);
            } else if ("-output".equals(args[i])) {
                FileOutputFormat.setOutputPath(job, new Path(args[++i]));
            } else if ("-delimiter".equals(args[i])) {
                job.getConfiguration().set("delimiter", args[++i]);
            } else if ("-reducer".equals(args[i])) {
                job.setNumReduceTasks(Integer.parseInt(args[++i]));
            }
        }
    }
}
```



### 3. Mutiple output

Inner Join 2번째 예제에서 출력되는 결과를 Multiple Output으로 출력

Reducer에서 작성해줘야하며, CleanUp 단계에서 `output.close();`를 해주지않으면 결과 파일들이 0byte로 출력되는 등의 Timing issue로 인한 문제들이 발생하므로 주의할 것



##### Reducer

MultipleOutput 객체를 생성 후 write(`job name`,`key`,`value`,`출력될 파일명`) 로 context를 보냄

```
package jh.hadoop.mapreduce.sample.Join;

import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.output.MultipleOutputs;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.IOException;

public class JoinByKeyReducer extends Reducer<Text, Text, Text, Text> {
    private static final Logger logger = LoggerFactory.getLogger(JoinByKeyReducer.class);

    private String pre_flag;
    private StringBuilder chat_all;
    private String pre_id;
    private MultipleOutputs<Text, Text> outputs;

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        pre_flag = "";
        pre_id = "";
        chat_all = new StringBuilder();
        outputs = new MultipleOutputs<>(context);
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
        outputs.close();
    }

    @Override
    protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
        chat_all.setLength(0);
        String[] flag = key.toString().split("\\^");
        String id = flag[0];
        String type = flag[1];

        switch (type) {
            case "A":
                pre_flag = "A";
                
                
                pre_id = id;
                break;
            default:
                if (pre_flag.equals("A") && pre_id.equals(id)) {
                    for (Text value : values) {
                        String chat_data = value.toString();
                        chat_all.append(chat_data).append("|");
                        context.getCounter("CUSTOM", "total chat").increment(1);
                    }
                    Text userId = new Text(id);
                    String chats = chat_all.substring(0, chat_all.length() - 1);
                    if (chats.isEmpty()) {
                        logger.info("chats = {}", chats);
                    }
                    Text value = new Text(chats);
                    context.getCounter("CUSTOM", "reduce mo write").increment(1);
                    outputs.write("userId", userId, value, key.toString().replaceAll("\\^", "_"));
                    pre_flag = "";
                    pre_id = "";
                }
                break;
        }
    }
}
```



##### Driver

Reducer에서 작성한 job name을 추가

LazyOutputFormat 설정시 0byte의 출력 결과는 생략됨

```
public int run(String[] args) throws Exception { 
...

        // MultipleOutput
        MultipleOutputs.addNamedOutput(job, "userId", TextOutputFormat.class, Text.class, Text.class);
        LazyOutputFormat.setOutputFormatClass(job, TextOutputFormat.class);
        
...
    }
```


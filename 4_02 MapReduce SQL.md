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

이때 내용들은 Text형으로 들어오며 



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



## 2. select * from table where a = 'v'

hdfs 파일에서 특정 값 'v'가 있는 row만 output으로 저장하는 것을 목표로 함



##### Mapper

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



## 4. select a, array_agg(b) from table group by a

array_agg : 해당되는 값들을 배열로 출력해줌



#### 1. Reducer단계에서 array_agg 수행

------

##### Mapper

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




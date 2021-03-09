# SQL을 MapReduce로 구현

sql문의 출력 결과와 똑같은 결과를 얻는 것을 목표로 하며 실제 sql과는 차이가 있음

### 1. select * from table

hdfs파일을 읽어 모든 출력 내용을 output으로 저장하는 것을 목표로 함

이때, Mapper로 읽은 내용을 그대로 저장하면 되므로 Reducer가 필요 없음

> Setup과 CleanUp 과정은 Mapper 또는 Reducer 호출 시 시작과 끝에 한번식 호출되며
>
> Map, Reduce 과정은 **key 개수 만큼 반복**됨
>
> 이점에 유의하여 작성할 것



#### SelectAllMapper

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



#### SelectAllDriver

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



### 2. select * from table where a = 'v'

hdfs 파일에서 특정 값 'v'가 있는 row만 output으로 저장하는 것을 목표로 함



#### SelectWhereMapper

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



#### SelectWhereDriver

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



### 3. select a, count(*)  from table where a = 'v'

hdfs 파일에서 특정 값 'v'가 있는 row의 수를 count하여 output으로 저장하는 것을 목표로 함

row 수를 count하여 출력해줄 Reducer가 필요



#### WhereCountMapper

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



#### WhereCountReducer

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



#### WhereCountDriver

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


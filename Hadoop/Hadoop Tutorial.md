Hadoop从这里开始!和我一起学习下使用Hadoop的基本知识，下文将以[Hadoop Tutorial](http://www.cloudera.com/content/cloudera-content/cloudera-docs/HadoopTutorial/CDH4/Hadoop-Tutorial.html)为主体带大家走一遍如何使用Hadoop分析数据!

## Hadoop教程（一） ##
这个专题将描述用户在使用Hadoop MapReduce(下文缩写成MR)框架过程中面对的最重要的东西。Mapreduce由client APIs和运行时(runtime)环境组成。其中client APIs用来编写MR程序，运行时环境提供MR运行的环境。API有2个版本，也就是我们通常说的老api和新api。运行时有两个版本：MRv1和MRv2。该教程将会基于老api和MRv1。

*其中:老api在org.apache.hadoop.mapred包中,新api在 org.apache.hadoop.mapreduce中。*

### 前提 ###
首先请确认已经正确安装、配置了[CDH](http://www.cloudera.com/content/cloudera/en/products/cdh.html)，并且正常运行。
### MR概览 ###
Hadoop MapReduce 是一个开源的计算框架，运行在其上的应用通常可在拥有几千个节点的集群上并行处理海量数据（可以使P级的数据集）。

MR作业通常将数据集切分为独立的chunk，这些chunk以并行的方式被map tasks处理。MR框架对map的输出进行排序，然后将这些输出作为输入给reduce tasks处理。典型的方式是作业的输入和最终输出都存储在分布式文件系统(HDFS)上。

通常部署时计算节点也是存储节点，MR框架和HDFS运行在同一个集群上。这样的配置允许框架在集群的节点上有效的调度任务，当然待分析的数据已经在集群上存在，这也导致了集群内部会产生高聚合带宽现象（通常我们在集群规划部署时就需要注意这样一个特点）。

MapReduce框架由一个Jobracker（通常简称JT）和数个TaskTracker（TT）组成（在cdh4中如果使用了Jobtracker HA特性，则会有2个Jobtracer，其中只有一个为active，另一个作为standby处于inactive状态）。JobTracker负责在所有tasktracker上调度任务，监控任务并重新执行失败的任务。所有的tasktracker执行jobtracker分配过来的任务。

应用至少需要制定输入、输出路径，并提供实现了适当接口和(或)抽象类的map和reduce函数。这些路径和函数以及其他的任务参数组成了任务配置对象（job configuration）。Hadoop 任务客户端提交任务（jar包或者可执行程序等）和配置对象到JT。JT将任务实现和配置对象分发到数个TT（由JT分配），调度、监控任务，并向客户端返回状态和检测信息。

Hadoop由JavaTM实现,用户可以使用java、基于JVM的其他语言或者以下的方式开发MR应用：

- Hadoop Streaming- 允许用户以任何一种可执行程序（如shell脚本）实现为mapper和(或)reducer来创建和运行MR任务。
- Hadoop Pigs - 一种兼容[SWIG](http://www.swig.org/)(不基于JNITM)的C++实现MapReduce应用。

### 输入和输出 ###

MapReuce框架内部处理的是kv对(key-value pair)，因为MR将任务的输入当做一个kv对的集合，将输出看做一个kv对的集合。输出kv对的类型可以不同于输入对。

key和vaue的类型必须在框架内可序列化(serializable)，所以key value 必须实现[Writable](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/io/Writable.html)接口。同时，key 类必须实现[WritableComparable](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/io/WritableComparable.html)以便框架对key进行排序。

典型的MR任务输入和输出类型转换图为：

    (input) k1-v1 -> map -> k2-v2 -> combine -> k2-v2 -> reduce -> k3-v3 (output)

### 经典的WordCount1.0 ###

玩Hadoop不得不提WordCount，CDH原文里也以这为例，当然这里也以它为例:）

简单说下WordCount,它是计算输入数据中每个word的出现次数。因为足够简单所以经典，和Hello World有的一拼!

上源码：

    package org.myorg;

    import java.io.IOException;
    import java.util.*;

    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.conf.*;
    import org.apache.hadoop.io.*;
    import org.apache.hadoop.mapred.*;
    import org.apache.hadoop.util.*;

    public class WordCount {

      public static class Map extends MapReduceBase implements Mapper<LongWritable, Text, Text, IntWritable> {
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(LongWritable key, Text value, OutputCollector<Text, IntWritable> output, Reporter reporter) throws IOException {
          String line = value.toString();
          StringTokenizer tokenizer = new StringTokenizer(line);
          while (tokenizer.hasMoreTokens()) {
            word.set(tokenizer.nextToken());
            output.collect(word, one);
          }
        }
      }

      public static class Reduce extends MapReduceBase implements Reducer<Text, IntWritable, Text, IntWritable> {
        public void reduce(Text key, Iterator<IntWritable> values, OutputCollector<Text, IntWritable> output, Reporter reporter) throws IOException {
          int sum = 0;
          while (values.hasNext()) {
            sum += values.next().get();
          } 
          output.collect(key, new IntWritable(sum));
        }
      }

      public static void main(String[] args) throws Exception {
        JobConf conf = new JobConf(WordCount.class);
        conf.setJobName("wordcount");

        conf.setOutputKeyClass(Text.class);
        conf.setOutputValueClass(IntWritable.class);

        conf.setMapperClass(Map.class);
        conf.setCombinerClass(Reduce.class);
        conf.setReducerClass(Reduce.class);

        conf.setInputFormat(TextInputFormat.class);
        conf.setOutputFormat(TextOutputFormat.class);

        FileInputFormat.setInputPaths(conf, new Path(args[0]));
        FileOutputFormat.setOutputPath(conf, new Path(args[1]));

        JobClient.runJob(conf);
      }
    }

首先编译WordCount.java

    $ mkdir wordcount_classes $ javac -cp classpath -d wordcount_classes WordCount.java

其中classpath为：

- CDH4 /usr/lib/hadoop/*:/usr/lib/hadoop/client-0.20/*
- CDH3 /usr/lib/hadoop-0.20/hadoop-0.20.2-cdh3u4-core.jar

打成jar包:

    $ jar -cvf wordcount.jar -C wordcount_classes/ .

假定：

- /user/cloudera/wordcount/input 输入HDFS路径
- /user/cloudera/wordcount/output 输出HDFS路径

创建文本格式的数据并移动到HDFS:

    $ echo "Hello World Bye World" > file0
    $ echo "Hello Hadoop Goodbye Hadoop" > file1
    $ hadoop fs -mkdir /user/cloudera /user/cloudera/wordcount /user/cloudera/wordcount/input
    $ hadoop fs -put file* /user/cloudera/wordcount/input

运行wordcount:

    $ hadoop jar wordcount.jar org.myorg.WordCount /user/cloudera/wordcount/input /user/cloudera/wordcount/output

运行完毕后查看输出:

    $ hadoop fs -cat /user/cloudera/wordcount/output/part-00000
    Bye 1
    Goodbye 1
    Hadoop 2
    Hello 2
    World 2

MR应用可以用-files参数指定在当前工作目录下存在的多个文件，多个文件使用英文逗号分隔。-libjars参数可以将多个jar包添加到map和reduce的classpath。-archive参数可以将工作路径下的zip或jar包当做参数传递，而zip或jar包名字被当做一个链接(link)。更加详细的命令信息可以参考[Hadoop Command Guide](http://archive.cloudera.com/cdh/3/hadoop/commands_manual.html).

使用-libjars和-files参数运行wordcount的命令：

    hadoop jar hadoop-examples.jar wordcount -files cachefile.txt -libjars mylib.jar input output
    
详细看下wordcount应用

14-26行实现了Mapper，通过map方法(18-25行)一次处理一行记录，记录格式为指定的TextInputFormat（行49）。然后将一条记录行根据空格分隔成一个个单词。分隔使用的是类StringTokenizer，然后以<word,1>形式发布kv对。

在前面给定的输入中，第一个map将会输出:< Hello, 1> < World, 1> < Bye, 1> < World, 1>

第二个map输出: < Hello, 1> < Hadoop, 1> < Goodbye, 1> < Hadoop, 1>

我们会在本篇文章中深入学习该任务中map的大量输出的数目，并研究如何在更细粒度控制输出。

WordCount 在46行指定了combiner。因此每个map的输出在根据key排序后会通过本地的combiner(实现和reducer一致)进行本地聚合。

第一个map的最终输出:< Bye, 1> < Hello, 1> < World, 2>

第二个map输出:< Goodbye, 1> < Hadoop, 2> < Hello, 1>

Reducer实现(28-36行)通过reduce方法(29-35)仅对值进行叠加，计算每个单词的出现次数。

所以wordcount的最终输出为: < Bye, 1> < Goodbye, 1> < Hadoop, 2> < Hello, 2> < World, 2>

run方法通过JobConf对象指定了该任务的各种参数，例如输入/输出路径，kv类型，输入输出格式等等。程序通过调用 JobClient.runJob (55行)提交并开始监测任务的执行进度。

后面我们将对JobConf、JobClient、Tool做进一步学习。 







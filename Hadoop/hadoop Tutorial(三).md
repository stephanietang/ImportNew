书接上回，继续为大家讲解[MapReduce相关](
http://www.cloudera.com/content/cloudera-content/cloudera-docs/HadoopTutorial/CDH4/Hadoop-Tutorial/ht_topic_6_2.html)

## Hadoop教程(三) ##
### Job Configuration ###
[JobConf](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)是MR任务的配置对象，也是描述MR任务在Mapreduce框架中如何执行的主要途径，框架将如实的以该对象包含的信息来执行MR任务，有以下情况除外：

1 一些配置参数被管理员在hadoop相关配置文件中(比如core-site.xml,mapred-site.xml)设置为[final](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/conf/Configuration.html)，则不能被任务参数值改变。
2 一些参数比如[setNumReduceTasks(int)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)是直接通过客户端api设置，无其他内部隐藏的逻辑。但是一些其他参数比如[setNumMapTasks(int)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)虽然被客户端指定，但框架内部和/或者任务配置隐藏有其他逻辑，参数值最终设定更复杂。

最典型的，JobConf一般应用在确定Mapper、Combiner(如果使用的话)、Partitioner、Reducer、InputFormat、OutputFormat以及OutputCommitter的实现类上。JobConf也可以用来通过setInputPaths(JobConf, Path…)/[ addInputPath(JobConf, Path)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/FileInputFormat.html),或者setInputPaths(JobConf, String)/[addInputPaths(JobConf, String)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/FileInputFormat.html)指定输入路径集合，通过setOutputPath(Path)设置任务结果输出路径。

JobConf也会用来指定一些可选的配置（一般使用在优化或者特殊分析用途）。比如指定作业使用的Comparator(比较器，用于排序或者分组);使用 DistributedCache缓存一些必须的文件;指定作业过程中数据和/或者作业结果是否被压缩和怎样压缩。也可以通过setMapDebugScript(String)/setReduceDebugScript(String)（还没用过:(）对作业进行debug;通过[setMapSpeculativeExecution(boolean)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)/[setReduceSpeculativeExecution(boolean)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)指定任务执行中是否开启推测执行；通过setMaxMapAttempts(int)/setMaxReduceAttempts(int)设置每个任务的最大尝试次数；通过[setMaxMapTaskFailuresPercent(int)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)/[setMaxReduceTaskFailuresPercent(int)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)设置MR任务(map/reduce)可容忍的失败比率。

而且，用户可以用[set(String, String)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/conf/Configuration.html)/[get(String, String](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/conf/Configuration.html) 设置应用需要的参数，但是对于大量数据（只读）请使用 DistributedCache。

### MR运行参数 ###
TaskTracer以独立的JVM子进程方式运行Mapper、Reducer。子任务继承TaskTracker的所有环境参数。用户可以通过在JobConf里 mapred.child.java.opts指定子jvm运行参数，例如通过-Djava.library.path=<>指定一些搜索共享库必须的非标准运行时链接库等等，这是可选的。如果 mapred.child.java.opts属性含有$taskid变量，这个变量值会在运行中被框架插入正在运行的任务id。

下面这个配置例子使用多参数和变量插入特性配置了JVM GC日志，可，通过jconsole工具无密码连接的JMX服务，用来观察子进程的内存、线程，也可以获取线程dump信息。另外配置里设置了子JVM的最大堆尺寸为512MB,通过java.library.path设置了共享库路径。

     <property>
      <name>mapred.child.java.opts</name>
      <value>
         -Xmx512M -Djava.library.path=/home/mycompany/lib -verbose:gc -Xloggc:/tmp/@taskid@.gc
         -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false
      </value>
    </property>

####内存管理  ####
可以通过mapred.child.ulimit参数配置子进程可使用的最大虚拟内存。

注意:该属性对单个进程设置最大限制，单位为KB,值必须大于等于最大堆内存(通过-Xmx设置)。
注意：mapred.child.java.opts只对从tasktracker分配和管理的子JVM进程有效。其他hadoop守护进程内存参数配置详见[Configuring the Environment of the Hadoop Daemons](http://archive.cloudera.com/cdh/3/hadoop/cluster_setup.html)。

MR框架的其他组件可用内存也是可配置的。map和reduce任务的性能可以通过从影响操作并发和数据磁盘IO次数角度来调试。对单个任务检测文件系统的计数器，甚至具体到map输出到reduce的数据字节量对调试影响任务运行性能没有太大价值。

如果内存管理特性启用的话，用户可以选择性的覆盖一些默认配置，比如虚拟内存，RAM。下面是一些对任务有效的参数：
              
mapred.task.maxvmem      类型int  以字节为单位指定单个map或reduce任务的最大虚拟内存。如果任务超过该值就被kill

mapred.task.maxpmem      类型int  以字节为单位指定单个map或reduce任务的最大RAM.这个值被调度器(Jobtracer)参考作为分配map\reduce任务的依据，避免让一个节点超RAM负载使用。

#### Map参数 ####
map读出的一条记录将被序列化到一个buffer，元数据存储在元数据buffer中。如上面所说，当序列化buffer或者元数据buffer超出了设置的阙值，buffer中内容将被排序并在后台写入到磁盘，这个过程同时map持续输出记录行。如果buffer被写满就会发生一个spill过程，spill中map线程被阻塞。当map完成后，buffer中剩余记录写入磁盘并和在磁盘的按段存储的记录合并到一个文件中。减少spill次数可以缩短map时间，但是较大的buffer也会降低可用内存。

io.sort.mb   int 默认100   以MB为单位设置序列化和元数据buffer的大小。

io.sort.record.percent  float 默认0.05  map记录序列化后数据元数据buffer所占总buffer百分比值。为了加速排序，除了序列化后本身尺寸外每条序列化后的记录需要16字节的元数据。io.sort.mb值被占用的百分比值超过设定值机会发生spill。对输出记录较少的map，值越高越可降低spill发生的次数。

io.sort.spill.percent float 默认0.80 元数据和序列化数据buffer空间阀值。当两者任何一个buffer空间达到该阀值，数据将被spill到磁盘。假设io.sort.record.percent=r, io.sort.mb=x,io.sort.spill.percent=q,那么在map线程spill之前最大处理的记录量为r*x*q*2^16。注意：较大的值可能降低spill的次数甚至避免合并，但是也会增加map被阻塞的几率。通过精确估计map的输出尺寸和减少spill次数可有效缩短map处理时间。

其他注意事项：

1 如果spill阙值达到了，就会发生spill，记录收集将继续直到spill完成。例如io.sort.buffer.spill.percent设置为0.33，buffer剩余的部分被填满，spill发生，下一个spill将会包含所有收集的记录，或者是这个buffer的0.66部分，并且不会产生额外的spill过程。换句话说阙值像是定义的触发器，不会阻塞。

2 大于序列化数据buffer的一条记录将首先触发一个spill，然后被spill到一个独立的文件中。没有配置参数来定义记录是否首先通过combiner。

#### Shuffle/Reduce参数 ####
如前面所说，每个reduce通过HTTP获取map输出后读入内存，并周期性合并这些输出到磁盘。如果map输出压缩打开的话，每个输出将会解压后读入内存。下面的配置参数影响reduce处理中的合并和内存分配过程。
1. io.sort.factor  int 默认值10  指定同时可合并的文件片段数目。参数限制了打开文件的数目，压缩解码器。如果文件数超过了该值，合并将分成多次。这个参数一般适用于map任务，大多数作业应该配置该项。
2. mapred.inmem.merge.threshold int 在内存中合并到磁盘前读取已排序map输出文件的数目。类似前面说的spill阙值，该值不是一个用来分区的单元而是一个触发器。通常该值较高(1000)，或者不启用（0）,毕竟内存内合并比磁盘上合并成本更低。这个阙值只影响shuffle过程的内存合并。
3. mapred.job.shuffle.merge.percent float 0.66 内存合并前供map输出享有的内存百分比值，超过该值就会合并数据到磁盘。过大的值会降低获取和合并的并行效率。相反如果输入恰好整个放到内存，则可以设置为1.0。该参数只影响shuffle过程的内存合并频率。
4. mapred.job.shuffle.input.buffer.percent float 0.7 shuffle过程中缓存map输出数据的内存占整个子jvm进程堆最大尺寸(通过mapred.child.java.opts设置)的百分比。该值可以视情况设置较高的值来存储大的较多的map输出。
5. mapred.job.reduce.input.buffer.percent float 0.0 内存合并中从内存刷到磁盘，直到剩余的map输出占用的内存少于jvm最大堆的该百分比值。默认情况下，在reduce开始之前需要保证最大的内存可用，所有的内存中map输出都会被合并到磁盘。对内存不敏感的reduce任务，该值可以适当提高，来避免磁盘IO（一般不会有）.

其他注意事项：
1. 如果map输出占用超过25%的内存去拷贝，这会直接被刷到磁盘而不会经过内存合并阶段。
2. 当combiner运行，前面所说的较高的合阙值和大的buffer不适用。因为在所有map输出都被取到之前合并就开始了，当spill发生时combiner处于运行状态。在一些实践中，用户可以通过合并输出使得磁盘spill足够小并可保证并行的spill和数据拉取来缩短reduce处理时间，而不是去不断的提高buffer尺寸。
3. 内存合并map输出并刷到磁盘时reduce过程开始，如果有多个输出片段spill到磁盘，或者至少有io.sort.factor个片段已经在磁盘上，那么中间合并过程是必须的，并且包含内存中处理map输出。

#### 子JVM重用 ####
可以通过指定mapred.job.reuse.jvm.num.tasks作业配置参数来启用jvm重用。默认是1，jvm不会被重用（每个jvm只处理1个任务）。如果设置为-1，那么一个jvm可以运行同一个作业的任意任务数目。用户可以通过JobConf.setNumTasksToExecutePerJvm(int)指定一个大于1的值。


下面是作业执行时的配置参数：
1. mapred.job.id  string    表示jobid
2. mapred.jar     string    job.jar在job路径下的位置
3. job.local.dir  string    作业共享路径
4. mapred.tip.id  string    taskid
5. mapred.task.id string    task尝试任务id
6. mapred.task.is.map boolean  是否是map任务
7. mapred.task.partition  int   task在job中的id
8. map.input.file string    map输入文件路径
9. map.input.start long     map输入split开始偏移量
10. map.input.length   long map输入分片的字节数
11. mapred.work.output.dir string 任务临时输出路径


任务的标准输出和错误流日志由TaskTracker读入并写入${HADOOP_LOG_DIR}/userlogs路径。

[DistributedCache](http://www.cloudera.com/content/cloudera-content/cloudera-docs/HadoopTutorial/CDH4/Hadoop-Tutorial/ht_topic_6_7.html#topic_6_7_2_unique_1)可以被用来发布map或者reduce用到的jar包、本地共享库。子JVM进程通常可使用java.library.path和LD_LIBRARY_PATH指定其自身的工作路径。缓存库可以通过[ System.loadLibrary](http://java.sun.com/javase/6/docs/api/java/lang/System.html)或者[System.load](http://java.sun.com/javase/6/docs/api/java/lang/System.html)加载。关于使用distributed cache 加载共享库详细信息可以查看 [Loading native libraries through DistributedCache](http://hadoop.apache.org/common/docs/r0.23.6/native_libraries.html)。
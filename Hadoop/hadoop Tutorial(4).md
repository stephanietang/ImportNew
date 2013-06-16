书接上回，继续为大家讲解[MapReduce相关](http://www.cloudera.com/content/cloudera-content/cloudera-docs/HadoopTutorial/CDH4/Hadoop-Tutorial/ht_topic_6_4.html)

## Hadoop教程(四) MR作业的提交监控、输入输出控制及特性使用##
### 提交作业并监控 ###
[JobClient](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobClient.html)是用户作业与JobTracker交互的主要接口，它提供了提交作业，跟踪作业进度、访问任务报告及logs、以及获取MR集群状态信息等方法。

提交作业流程包括：

* 检查作业的输入输出
* 计算作业的输入分片(InputSplit)
* 如果需要，为DistributedCache设置必须的账户信息
* 将作业用到的jar包文件和配置信息拷贝至文件系统（一般为HDFS）上的MR系统路径中
* 提交作业到JobTracker，并可监控作业状态

作业历史(Job History)文件会记录在hadoop.job.history.user.location指定的位置，默认在作业输出路径下的_logs/history/路径下。因此历史日志默认在mapred.output.dir/_logs/history下。用户可以将hadoop.job.history.user.location值设置为none来不记录作业历史。

使用命令来查看历史日志:

<pre class="brush: java; gutter: true">
$hadoop job -history output-dir
</pre>

上面命令会显示作业的详细信息、失败的被kill的任务（tip）的详细信息。使用下面命令可以查看作业更详细的信息：

<pre class="brush: java; gutter: true">
$hadoop job -history all output-dir
</pre>

可以使用[OutputLogFilter](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/OutputLogFilter.html)从输出路径中过滤日志文件。

一般，我们创建应用，通过JobConf设置作业的各种属性，然后使用JobClient提交作业并监控进度。

#### 作业控制####

有时可能需要一个作业链完成复杂的任务。这点是可以轻松实现的，因为作业输出一般都在分布式文件系统上，作业输出可以当做下个作业的输入，这样就形成了链式作业。

这种作业成功是否依赖于客户端。客户端可以使用以下方式来控制作业的执行：
* [runJob(JobConf)](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/mapred/JobClient.html):提交作业并仅在作业完成时返回
* [submitJob(JobConf)](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/mapred/JobClient.html):提交作业后立即返回一个[RunningJob](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/mapred/RunningJob.html)的引用，使用它可以查询作业状态并处理调度逻辑。
* [JobConf.setJobEndNotificationURI(String)](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/mapred/JobConf.html):设置作业完成时通知

你也可以使用[Oozie](http://archive.cloudera.com/cdh4/cdh/4/oozie/)来实现复杂的作业链。

#### 作业输入####

下面讲作业输入的内容。
[InputFormat](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/InputFormat.html) 描述MR作业的输入信息。InputFormat有以下作用:

1. 验证作业的输入信息
2. 将输入文件拆分为逻辑上的输入分片(InputSplit),每个输入分片被分配到一个独立的Mapper
3. 提供RecordReader实现类从输入分片中搜集输入记录供Mapper处理

基于文件的InputFormat实现类即[FileInputFormal](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/mapred/FileInputFormat.html)是通过计算输入文件的总大小(以字节为单位)来分裂成逻辑分片的。然而文件系统的块大小又作为输入分片大小的上限，下限可以通过mapred.min.split.size来设定。

基于输入大小进行逻辑分片机制在很多情况下是不适合的，还需要注意记录边界。在这些情况下，应用应该实现RecordReader来处理记录边界，为独立的mapper任务提供面向行的逻辑分片。

[TextInputFormat](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/mapred/TextInputFormat.html)是默认的输入格式。

如果作业输入格式为TextInputFormat，MR框架可以检测以.gz扩展名的输入文件并自动使用合适的压缩算法来解压文件。特别注意的是，.gz格式的压缩文件不能被分片，每个文件作为一个输入分片被一个mapper处理。

#### 输入分片(InputSplit)####

[InputSplit](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/mapred/InputSplit.html)的数据会被一个独立的mapper来处理。一般输入分片是对输入基于面向字节为单位的。可以使用RecordReader来处理提供面向行的输入分片。

[FileSplit](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/FileSplit.html)是默认的InputSplit,通过设置map.input.file设置输入文件路径。

#### RecordReader####

[RecordReader](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/RecordReader.html)从InputSplit读入数据对。RecordReader将InputSplit提供得面向字节的的输入转换为面向行的分片给Mapper实现类来处理。RecordReader的责任就是处理行边界并以kv方式将数据传给mapper作业。
#### 作业输出####

[OutFormat](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/OutputFormat.html)描述作业的输出信息。MR框架使用OutputFormat来处理：

* 验证作业的输出信息。例如验证输出路径是否存在.
* 提供RecordWriter实现来写作业的输出文件。输出文件存储在文件系统中。TextOutputFormat是默认的输出格式。

#### OutputCommitter####

[OutputCommitter](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/OutputCommitter.html)是MR作业的输出提交对象。MR框架使用它来处理：

* 初始化时准备作业。例如，作业初始化时创建临时输出路径。作业准备阶段通过一个独立的任务在作业的PREP状态时完成，然后初始化作业的所有任务。一旦准备阶段完成，作业状态切换到Running状态。
* 作业完成后清理作业。例如，在作业完成后清理输出临时路径。作业清理阶段通过一个独立的任务在作业最后完成。作业在清理阶段完成后设置为成功/失败/中止。
* 设置任务的临时输出。任务准备阶段在任务初始化时作为同一个任务的一部分。
* 检查任务是否需要一个提交对象。目的是为了在任务不需要时避免一个提交过程。
* 提交任务输出。一旦任务完成，如果需要的话，任务将提交其输出物。
* 销毁任务的提交。如果任务失败或者被中止，输出将被清理。如果不能被清理（如异常的块），与该任务相同的任务id会启动并处理清理。

FileOutputCommitter是默认的OutputCommiter实现。作业准备及清理任务占用同一个TaskTracker上空闲的map或者reduce槽。作业清理任务、任务清理任务以及作业准备任务按照该顺序拥有最高的优先级。

#### 任务副作用文件####

在一些应用中，任务需要创建和/或写入文件，这些文件不同于实际的作业输出文件。

在这种情况下，相同的2个Mapper或者Reducer有可能同时运行（比如推测执行的任务），并在文件系统上尝试打开和或者写入相同文件或路径。所以你必须为每个任务的执行使用唯一名字（比如使用attapid，attempt_200709221812_0001_m_000000_0）。

为了避免这样的情况，当OutputCommiter为FileOutputCommiter时，MR框架通过${mapred.work.output.dir}为每个任务执行尝试维护一个特殊的路径：${mapred.output.dir}/temporary/${taskid}，这个路径用来存储任务执行的输出。在任务执行成功完成时，${mapred.output.dir}/temporary/${taskid} 下的文件被移到${mapred.output.dir}下。当然框架丢弃不成功的任务执行的子路径。这个进程对用户应用透明。

MR应用开发者可以利用这个特性在执行过程中通过[FileOutputFormat.getWorkOutputPath()](http://hadoop.apache.org/docs/r0.23.6/api/org/apache/hadoop/mapred/FileOutputFormat.html)在${mapred.work.output.dir} 下创建需要的site files，框架会像成功的任务试执行那样处理，这样就避免为每个试执行任务取唯一名字。

*注意*：特殊试执行任务执行过程中${mapred.work.output.dir} 值实际是${mapred.output.dir}/temporary/${taskid}，该值被框架定制。所以直接在FileOutputFormat.getWorkOutputPath()返回的路径中创建site－files，来使用这个特性。

对于只有mapper的作业，side－files将直接进入hdfs。

#### RecordWriter####

[RecordWriter](http://hadoop.apache.org/docs/r0.23.6/api/org/apache/hadoop/mapred/RecordWriter.html)将输出的kv值写入输出文件。RecordWriter实现类将作业输出写入文件系统。

### 其他特性###

#### 提交作业到队列####

用户提交的作业到队列中，队列是一个作业的集合，允许MR提供一些特定功能。队列使用ACL控制哪些用户提交作业。队列一般和Hadoop Scheduler调度器一起使用。

Hadoop 安装后默认配置了名称为default的队列，这个队列是必须的。队列名称可以在hadoop配置文件中mapred.queue.names属性里配置。一些作业调度器比如[Capacity Scheduler](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/CapacityScheduler.html)支持多个队列。

作业可以通过mapred.job.queue.name或者通过[setQueueName(Stirng)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)设置队列名称，这是可选的。如果作业名称没有设置队列名称，则提交到default的队列中。

#### 计数器Counters####

计数器用来描述MR中所有的计数，可以是MR框架定义也可以是应用提供。每个计数器可以是任何Enum类型。一组特定的Enum被聚合成一个组即为Counters.Group.

应用可以定义Enum类型的计数器，并可以通过[Reporter.incrCounter(Enum,long](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/Reporter.html)或者[Reporter.incrCounter(String,String,long](http://hadoop.apache.org/docs/r0.23.6/api/org/apache/hadoop/mapred/Reporter.html)在map和或者reduce中更新这个计数器的值。然后这些计数器统一被框架聚合。

#### DistributedCache####
[DistributedCache](http://hadoop.apache.org/docs/r0.23.6/api/org/apache/hadoop/filecache/DistributedCache.html)可以高效地将应用明细、大的只读文件发布。DistributedCache是MR框架提供的缓存文件（如文本、压缩包、jar包等）的高效工具。

应用需要在JobConf里使用hdfs://指定地址进行缓存。这些文件必须在文件系统中已经存在。

MR作业的任务在某个节点上执行前，MR框架会拷贝必需的文件到这个节点。特别说明的是，作业必需的所有文件只需要拷贝一次，支持压缩包，并在各个节点上解压，有助于提高MR执行效率。

DistributedCache会跟踪所有缓存文件的修改时间戳。作业执行过程中应用或者外部不应该修改被缓存的文件。

通常，DistributedCache被用来缓存简单、只读的数据或者文本文件以及像压缩包/jar包等复杂文件。压缩包比如zip\tar\tgz\tar.gz文件到节点上被解压。所有文件都有执行权限。

属性 mapred.cache.{files|archives}配置的文件和包等可以被发布，多个文件可以使用“,”来分隔。也可以调用 DistributedCache.addCacheFile(URI,conf)或者DistributedCache.addCacheArchive(URI,conf) ，DistributedCache.setCacheFiles(URIs,conf)/DistributedCache.setCacheArchives(URIs,conf) 设置，其中URI为hdfs://host:port/absolute-path#link-name形式。如果使用streaming方式，可以通过 -cacheFile/-cacheArchive来设置。

另外，用户也可以通过[DistributedCache.createSymlink(Configuration)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/filecache/DistributedCache.html) API 将分布式缓存的文件使用符号链接到当前工作路径。或者也可以通过设置mapred.create.symlink 为yes。这样分布式缓存将URI的最末部分作为符号链接。例如hdfs://namenode:port/lib.so.1#lib.so 的符号链接为lib.so.

分布式缓存也可以用作在map，reduce阶段中基本的软件分布式处理机制，可以用来分布式处理jar包或者本地库。[DistributedCache.addArchiveToClassPath(Path, Configuration)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/filecache/DistributedCache.html)或者[DistributedCache.addFileToClassPath(Path, Configuration)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/filecache/DistributedCache.html)用来将需要缓存的文件或者jar包加入到作业运行的子jvm的classpath中。也可以通过mapred.job.classpath.{files|archives}来设置。同样这些分布式缓存的文件或jar包也可以使用符号链接到当前工作路径，当然也支持本地库并可被作业在执行中加载。


#### 工具类Tool####

[Tool接口](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/util/Tool.html)支持常用的Hadoop命令行参数的处理操作。它是MR工具或者应用的标准工具。应用应该通过ToolRunner.run(Tool,Stirng[])将标准命令行的参数使用[GenericOptionsParser](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/util/GenericOptionsParser.html)来处理。只处理应用的自定义参数。

Hadoop中命令行里通用的参数有：

<pre class="brush: plain; gutter: true">
-conf <configuration file>
-D <property=value>
-fs <local|namenode:port>
-jt <local|jobtracker:port>
</pre>

#### IsolationRunner####

[IsolationRunner](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/IsolationRunner.html)是MR应用的调试工具。

要使用它，需要先设置keep.failed.task.files = true，另外请注意属性keep.tasks.files.pattern。

下一步，到失败作业的某个失败节点上，cd到TT的本地路径并运行IsolationRunner。

<pre class="brush: java; gutter: true">
$ cd <local path>/taskTracker/${taskid}/work
$ hadoop org.apache.hadoop.mapred.IsolationRunner ../job.xml
</pre>
    
IsolationRunner 会再单独的jvm中对相同的输入上运行失败的任务并可以debug。

#### 性能分析Profiling####

Profiling可以用来对作业的map，reduce任务使用java内置的性能分析工具进行采样，得出具有代表性(2个或3个试样)的采样结果。

用户可以通过设置mapred.task.profile属性指定框架是否对作业的任务收集性能信息。也可以通过API JobConf.setProfileEnabled(boolean)来指定。如果值为true，任务性能分析就启用了。采样结果保存在用户日志目录中，默认性能采样不开启。

一旦需要性能分析，可以通过属性mapred.task.profile.{maps|reduces} 设置采样任务的范围。使用JobConf.setProfileTaskRange(boolean,String)一样可以设置。默认的range是0-2.

可以设置mapred.task.profile.params确定分析参数，api方式为 JobConf.setProfileParams(String)。如果参数中含有“%s”占位符，MR框架会使用采样结果输出文件名替代它。这些参数被传到子JVM.默认值为-agentlib:hprof=cpu=samples,heap=sites,force=n,thread=y,verbose=n,file=%s.

#### 调试工具debugging####

MR框架支持用户自定义脚本调试工具。例如当MR任务失败时，用户可以运行debug脚本处理任务日志。脚本可以访问任务的输出和错误日志，系统日志以及作业配置信息。调试脚本输出和错误流作为作业输出一部分显示到控制台诊断信息中。

下面讲讨论如何为作业提交调试脚本。当然脚本需要分发并提交到Hadoop中MR中。

**如何分发脚本文件**

使用DistributedCache分发并且为脚本文件建立符号链接。

**如何提交脚本**

一种便捷的途径是通过设置mapred.map.task.debug.script和mapred.reduce.task.debug.script分别实现对map和reduce指定调试脚本。也可以通过API方式，分别是JobConf.setMapDebugScript(String)和JobConf.setReduceDebugScript(String)。如果是streaming方式，可以通过命令行里指定-mapdebug 和-reducedebug实现。

脚本的参数是任务的输出流、错误流、系统日志以及job配置文件。调试模式下，以下命令在任务失败的节点上运行：

<pre class="brush: java; gutter: true">
$script $stdout $stderr $syslog $jobconf 
</pre>

Pipes程序在命令形式上以第五个参数传入。如：

<pre class="brush: java; gutter: true">
$script $stdout $stderr $syslog $jobconf $program
</pre>

**默认行为**

对pipes,使用默认的脚本以gdb方式处理核心转储(core dumps)，打印栈轨迹，并给出正在运行线程的信息。

#### JobControl####

JobControl可以对一组MR作业和他们的依赖环境进行集中管理。

#### 数据压缩####

Hadoop Mapreduce可以对中间结果(map输出)和作业最终输出以指定压缩方式压缩。通常都会启用中间结果的压缩，因为可以显著提高作业运行效率并且不需要对作业本身进行改变。只有MR shuffle阶段生成的临时数据文件被压缩（最终结果可能或者不被压缩）。

Hadoop支持zlib,gzip,snappy压缩算法。推荐Snappy，因为其压缩和解压相比其他算法要高效。

因为性能和java库不可用原因，Hadoop也提供本地的zlib和gzip实现。详情可以参考[这个文档](http://www.cnblogs.com/gpcuster/archive/2011/02/17/1957042.html)

**中间输出物**

应用可以通过[JobConf.setCOmpressMapOutput(boolean)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)设置是否对中间结果进行压缩，使用[JobConf.setMapOutputCompressorClass(Class)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/FileOutputFormat.html)设置压缩算法。

**作业输出**

通过[FileOutputFormat.setCompressOutput(JobConf,boolean)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/FileOutputFormat.html)设置是否压缩最终产出物，通过[FileOutputFormat.setOutputComressorClass(JobConf,Class)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/FileOutputFormat.html)设置压缩方式。

如果作业输出以[SequenceFileOutputFormat](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/SequenceFileOutputFormat.html)格式存储。则可以通过[SequenceFileOutputFormat.setOutputCompressionType(Jobconf,SequenceFile.CompressionType)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/SequenceFileOutputFormat.html) API 设定压缩方式.压缩方式有RECORD/BLOCK默认是RECORD。


#### 跳过损坏的记录####

Hadoop提供了一个选项，在MR处理map阶段时跳过被损坏的输入记录。应用可以通过[SkipBadRecords](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/SkipBadRecords.html)类使用这个特性。

作业处理时可能对确定的输入集上map任务会失败。通常是map函数存在bug，这时需要fix这些bug。但有时却无法解决这种特殊情况。比如这个bug可能是第三方库导致的。这时这些任务在经过若干尝试后仍然无法成功完成，作业失败。这时跳过这些记录集，对作业最终结果影响不大，仍然可以接受（比如对非常大的数据进行统计时）。

默认该特性没有开启。可以通过[SkipBadRecords.setMapperMaxSkipRecords(Configuration,long)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/SkipBadRecords.html)和[SkipBadRecords.setReducerMaxSkipGroups(Configuration,long)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/SkipBadRecords.html)开启。

开启后，框架对一定数目的map失败使用skipping 模式。更多信息可以查看[SkipBadRecords.setAttemptsToStartSkipping(Configuration, int)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/SkipBadRecords.html)。skipping模式中，map任务维护一个正被处理的记录范围值。这个过程框架通过被处理的记录行数计数器实现。可以了解[SkipBadRecords.COUNTER_MAP_PROCESSED_RECORDS](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/SkipBadRecords.html)和[SkipBadRecords.COUNTER_REDUCE_PROCESSED_GROUPS](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/SkipBadRecords.html)。这个计数器让框架知道多少行记录被成功处理了，哪些范围的记录导致了map的失败，当然这些坏记录就被跳过了。

被跳过的记录数目依赖应用被处理的记录计数器统计频率。建议当每行记录被处理时就增加该计数器。但是这个在一些应用中无法实现。这时框架可能也跳过了坏记录附近的记录。用户可以通过[SkipBadRecords.setMapperMaxSkipRecords(Configuration,long)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/SkipBadRecords.html)和[SkipBadRecords.setReducerMaxSkipGroups(Configuration,long)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/SkipBadRecords.html)控制被跳过的记录数。框架会使用类似二分搜索的方式努力去减小被跳过的记录范围。被跳过的范围会分为2部分，并对其中的一部分执行。一旦失败了，框架可以知道这部分含有损坏的记录。任务将重新执行直到遇到可接受的数据或者所有尝试机会用完。如果要提高任务尝试次数，可以通过[JobConf.setMaxMapAttampts(int)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)和J[obConf.setMaxReduceAttempts(int)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)设置。

被跳过的记录随后会以sequence file格式写入hdfs以便于后面可能的分析。路径可以通过[SkipBadRecords.setSkipOutputPath(JobConf,Path)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/SkipBadRecords.html)设定。


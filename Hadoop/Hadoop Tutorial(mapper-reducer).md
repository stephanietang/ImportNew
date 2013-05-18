书接上回，继续为大家讲解[MapReduce用户编程接口](
http://www.cloudera.com/content/cloudera-content/cloudera-docs/HadoopTutorial/CDH4/Hadoop-Tutorial/ht_topic_6.html)
## Hadoop Tutorial(二) ##

### MapReduce - 用户编程接口 ###
下面将着重谈下MapReduce框架中用户经常使用的一些接口或类的详细内容。了解这些会极大帮助你实现、配置和优化MR任务。当然javadoc中对每个class或接口都进行了更全面的陈述，这里只是一个指导。

首先来看下Mapper和Reducer接口，通常MR应用都要实现这两个接口来提供map和reduce方法，这些是MRJob的核心部分。

#### Mapper ####
Mapper 将输入的kv对映射成中间数据kv对集合。Maps 将输入记录转变为中间记录，其中被转化后的记录不必和输入记录类型相同。一个给定的输入对可以映射为0或者多个输出对。

在MRJob执行过程中，MapReduce框架根据提前指定的InputFormat(输入格式对象)产生InputSplit（输入分片），而每个InputSplit将会由一个map任务处理。

总起来讲，Mapper实现类通过[JobConfigurable.configure(JobConf)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConfigurable.html)方法传入JobConf对象来初始化，然后在每个map任务中调用[map(WritableComparable,Writable,OutputCollector,Reporter)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/Mapper.html)方法处理InputSplit的每个kv对。MR应用可以覆盖[Closeable.close](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/io/Closeable.html)方法去处理一些必须的清理工作。

输出对不一定和输入对类型相同。一个给定的输入对可能映射成0或者很多的输出对。输出对是框架通过调用[OutputCollector.colect(WritableComparable,Writable)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/OutputCollector.html)得到。

MR应用可以使用Reporter汇报进度，设置应用层级的状态信息，更新计数器或者只是显示应用处于运行状态等。

所有和给定的输出key关联的中间数据都会随后被框架分组处理，并传给Reducer处理以产生最终的输出。用户可以通过[JobConf.setOutputKeyComparatorClass(Class)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)指定一个Comparator控制分组处理过程。

Mapper输出都被排序后根据Reducer数量进行分区，分区数量等于reduce任务数量。用户可以通过实现自定义的Partitioner来控制哪些keys(记录)到哪个Reducer中去。

此外，用户还可以指定一个Combiner，调用[JobConf.setCombinerClass(Class)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)来实现。这个可以来对map输出做本地的聚合，有助于减少从mapper到reducer的数据量。

经过排序的中间输出数据通常以一种简单的格式(key-len,key,value-len,value)存储。应用可以决定是否或者怎样被压缩以及压缩格式，可以通过JobConf来指定.

#### Map数 ####
通常map数由输入数据总大小决定，也就是所有输入文件的blocks数目决定。

每个节点并行的运行的map数正常在10到100个。由于Map任务初始化本身需要一段时间所以map运行时间至少在1分钟为好。

如此，如果有10T的数据文件，每个block大小128M，最大使用为82000map数，除非使用setNumMapTasks(int)（这个方法仅仅对MR框架提供一个建议值）将map数值设置到更高。
#### Reducer ####
Reducer 根据key将中间数据集合处理合并为更小的数据结果集。
用户可以通过[JobConf.setNumReduceTasks(int)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)设置作业的reducer数目。

整体而言，Reducer实现类通过[JobConfigurable.configure(JobConf)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConfigurable.html)方法将JobConf对象传入,并为Job设置和初始化Reducer。MR框架调用[ reduce(WritableComparable, Iterator, OutputCollector, Reporter) ](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/Reducer.html)来处理以key被分组的输入数据。应用可以覆盖[Closeable.close()](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/io/Closeable.html)处理必要的清理操作。

Reducer由三个主要阶段组成:shuffle,sort,reduce.

#### shuffle ####
输入到Reducer的输入数据是Mapper已经排过序的数据.在shuffle阶段,框架根据partition算法获取相关的mapper地址通过Http协议将数据由reducer拉取到reducer机器上处理。
#### sort ####
框架在这个阶段会根据key对reducer的输入进行分组(因为不同的mapper输出的数据中可能含有相同的key)。
shuffle和sort是同时进行的，同时reducer仍然在拉取map的输出。
#### Secondary Sort ####
如果对中间数据key进行分组的规则和在处理化简阶段前对key分组规则不一致时，可以通过[ JobConf.setOutputValueGroupingComparator(Class)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)设置一个Comparator.因为中间数据的分组策略是通过[ JobConf.setOutputKeyComparatorClass(Class) ](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/JobConf.html)设置的，可以控制中间数据根据哪些key进行分组。而JobConf.setOutputValueGroupingComparator(Class)则可用于在数据连接情况下对值进行二次排序。
#### Reduce(化简) ####
这个阶段框架循环调用[ reduce(WritableComparable, Iterator, OutputCollector, Reporter) ](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/Reducer.html)方法处理被分组的每个kv对。
reduce 任务一般通过[ OutputCollector.collect(WritableComparable, Writable)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/OutputCollector.html)将输出数据写入文件系统[FileSystem](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/fs/FileSystem.html)。
应用可以使用Reporter汇报作业执行进度、设置应用层级的状态信息并更新计数器(Counter),或者只是提示作业在运行。
注意，Reducer的输出不会进行排序。
#### Reducer数目 ####
合适的reducer数目可以这样估算：
(节点数目*mapred.tasktracker.reduce.tasks.maximum)*0.95 或
(节点数目*mapred.tasktracker.reduce.tasks.maximum)*1.75
因子为0.95时，当所有map任务完成时所有reducer可以立即启动，并开始从map机器上拉取数据。因子为1.75时,最快的一些节点将完成第一轮reduce处理，此时框架开始启动第二轮reduce任务，这样可以达到比较好的作业负载均衡。

提高reduce数目会增加框架的运行负担，但有利于提升作业的负载均衡并降低失败的成本。

上述的因子使用最好在作业执行时框架仍然有reduce槽为前提，毕竟框架还需要对作业进行可能的推测执行和失败任务的处理。

#### 不使用Reducer ####
如果不需要进行化简处理，可以将reduce数目设为0。

这种情况下，map的输出会直接写入到文件系统。输出路径通过[setOutputPath(Path)](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/FileOutputFormat.html)指定。框架在写入数据到文件系统之前不再对map结果进行排序。
#### Partitioner ####
[Partitioner](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/Partitioner.html)对数据按照key进行分区，从而控制map的输出传输到哪个reducer上。默认的Partitioner算法是hash（哈希）.分区数目由作业的reducer数目决定。
[HashPartitioner](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/lib/HashPartitioner.html) 是默认的Partitioner.
#### Reporter ####
[Reporter](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/Reporter.html)为MR应用提供了进度报告、应用状态信息设置，和计数器(Counter)更新等功能.

Mapper和Reducer实现可以使用Reporter汇报进度或者提示作业在正常运行。在一些场景下，应用在处理一些特殊的kv对是耗费了过多时间，这个可能会因为框架假定任务超时而强制停止了这些作业。为避免该情况，可以设置mapred.task.timeout 为一个比较高的值或者将其设置为0以避免超时发生。
应用也可以使用Reporter来更新计数(Counter).
#### OutputCollector ####
[OutputCollector](http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/OutputCollector.html)是MR框架提供的通用工具来收集Mapper或者Reducer输出数据(中间数据或者最终结果数据)。

Hadoop MapReduce提供了一些经常使用的mapper、reducer和partioner的实现来。这些工具可以点击http://hadoop.apache.org/common/docs/r0.23.6/api/org/apache/hadoop/mapred/lib/package-summary.html中学习。
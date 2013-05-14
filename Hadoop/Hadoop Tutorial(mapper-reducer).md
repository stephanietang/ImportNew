书接上回，继续为大家讲解[MapReduce用户编程接口](
http://www.cloudera.com/content/cloudera-content/cloudera-docs/HadoopTutorial/CDH4/Hadoop-Tutorial/ht_topic_6.html)
## Hadoop Tutorial(二) ##

### MapReduce - 用户接口 ###
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
未完待续

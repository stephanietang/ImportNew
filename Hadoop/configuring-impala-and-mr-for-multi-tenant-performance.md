原文标题：Configuring Impala and MapReduce for Multi-tenant Performance(http://blog.cloudera.com/blog/2013/06/configuring-impala-and-mapreduce-for-multi-tenant-performance/)

##为多租户场景集群配置Impala和Mapreduce##
[Cloudera Impala](http://blog.cloudera.com/blog/2013/05/cloudera-impala-1-0-its-here-its-real-its-already-the-standard-for-sql-on-hadoop/)包含很多令人惊喜的特性，但是其给人印象最深的应该是支持以多种格式分析HDFS和HBASE中数据的能力，并且不需要ETL。此外，用户可以使用多个框架如mapreduce和impala分析相同的数据。因此，Impala 可以和mapreduce一起运行在相同的物理机器上，支持企业的关键业务。对那些多租户的集群，尽管存在着可能的对集群资源需求的潜在冲突，Impala和Mapreduce都需要运维好。

这篇文章，将分享作者在配置Impala和Mapreduce过程中认为最理想的多租户运维环境的经验。目标就是帮助用户理解怎样去对多租户集群进行调优满足产品的服务水准(SLOs)，并向社区贡献一些有用的，事半功倍的测试方法和性能模型。


###定义实用测试场景###
Cloudera面对的是广泛和多样的客户群体，所以这使得为现实世界场景做测试是头等大事。基于普通的使用案例的现实测试提供了有意义的指导，反之，人为地，不切实际地测试得出的结论在作用到实践中时通常失败。

对于Impala，主要的测试工作量就是直接复制Impala用户的查询，schema，以及数据选取。我们使用这个方法测试几种不同类别的用户案例的查询和schema结构，通常会和客户一起合作，对所有的cdh组件得到相似的实际测试场景。在对他们指定的使用案例获得直接的性能结论过程中，客户也一样受益匪浅。

因为OLAP（online analytical processing）是用户使用Impala案例中最多的一个，我们也运行[TPC-H](http://www.tpc.org/tpch/),一种在OLAP建立的基准测试，是对Impala的第二个工作。尽管TPC-H提供了大量、公共的独立审计结果的资源（这些结果是基于上世纪90年代的一次OLAP用户调查得到的），所以现在我们运行这个著名的TPC-H查询的子集。

对于MapReduce，使用各种基准测试来对不同数据格式，对MR计算管道不同阶段不同压力进行测试。比如TeraSort套件（包括TeraGen，TeraSort和TeraValidate），数据清洗作业，处理数据中随机性不同量值。这些测试不同于其他的MR测试，比如使用类似SWIM的开源工具直接重现客户的成千上万道作业造成的工作负载。这些负载捕捉复杂并且多样，将会在未来添加。与之相比，我们开展的多租户测试，可以对测试进行精确控制，并容易重复进行。

多租户测试包含Impala查询、与MR并行基准测试等整套环节，并循环这个过程（每次，不同的impala查询使用不同的mr作业）。再加上各种体积的数据，不同的数据格式，并行运行Impala查询的不同，MR作业并行的不同，最后的结果会是一个百种参数设置的结果矩阵。这个测试矩阵给了我们信心，这样出现的结果是有意义的，并且Impala可以和MR像期待的那样一起在同一个集群中共用。

###多租户性能期望建模###
多租户资源管理涉及到将不同的资源分配到不同的计算框架的作业中。为数百个测试案例的所有矩阵数据建模是难以实现的。不过，以简单的方式，我们可以将资源分配简化为一句话：“当集群资源被争用时，Impala获得所有资源的x部分，而MR得到其余的”。

如果Impala和MR资源共享和谐，其性能应随着其占有资源的比例下降而下降。也就是说，Impala的多租户性能应不低于其独占所有资源时性能的比例值x部分，MR也不应低于其独占资源时性能的1-x部分。目标就是找到并验证资源管理控制的一系列方法，一道道满足期待的性能或者甚至超出期望。

这是理论上最简单的模型也可能是更复杂的。例如如果MR作业运行时间比Impala查询长，多租户运行放缓只在两者都运行时出现。那个Impala查询将达到前面所说的x比例的性能。而那个MR作业在作业起始时获得1-x比例的性能，而剩下的过程将获取到所有资源时的性能。

这个模型的另外一个扩展是覆盖不均匀的资源分配过程。例如大多数的Impala查询需要大的内存需求，而MR作业需要较多得IO资源。如果分配给Impala 50%的CPU，60%的内存，40%的硬盘IO，所有资源变化可同时影响性能，并且我们期待整体上性能降低范围在40-60%之间。这个比较复杂的伴随着多维度的资源分配模型在cdh是支持的，这里暂时不讲。

###共享资源的控制###
多租户资源管理控制套件在另一篇文章里"[ Setting up a Multi-tenant Cluster for Impala and MapReduce](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CM4Ent/latest/Cloudera-Manager-Installation-Guide/cmig_impala_res_mgmt.html)"(包含在Cloudera Manager 安装指导中)。套间里包含了对每个运行的计算框架的内存管理、CPU、IO管理等。文档中含有几个例子，和指导怎样配置下面的资源：

* **Memory.** 包含Impala内存限制。直接对Impala可消耗内存、最大同时运行的Map任务数、Reduce数设置。间接对MR内存控制。对于内存管理，主要关注点是防止内存的波动，作业或查询失败，或者任何框架对所有资源的独占导致的阻止其他框架运行程序。

为了设置总物理内存的比例x，设置Impala内存限制为（RAM/accounting factor）*x. 其中accounting factor是操作系统常驻内存中实际可用内存的参数常量，可用来调整服务器负载。发现值为1.5是相对安全的值，而1.3则更理想但更危险。

对于Mapreduce，对map和reduce任务数的最大值应为默认值的1-x。例如假设给了Impala所有集群资源的25%，则x=25%，那么map和reduce最大锁业数就为默认值的75%。如果并行map任务数最大默认值为16，reduce最大默认值是8，那么新值就分别应该是12和6。当每个任务在其子jvm容器中运行通过map()和reduce()方法处理数据时，这样可以有效地让Mapreduce使用其余剩余的1-x部分集群的内存资源。对应着这一变化，任何一个作业的调优与槽容量调优有关的都需要随之做变化（因为槽资源减少了）。注意，在MR1中槽是最小的调优单元。对未来的CDH5和YARN来说，反而可能会指定具体的内存使用量。YARN会对应用各主服务器（他们中得一个要运行MR作业）进行资源分配。

**CPU.** 我们通过[Linux Control Groups（Cgroups）](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CM4Ent/latest/Cloudera-Manager-Managing-Clusters/cmmc_resource_mgmt.html)来控制CPU的使用。Cgroups是Linux kernel 提供的一个特性，用户可以通过Cloudera Manager 4.5.x及以上版本配置。CGroup cpu共享给一个角色资源越多，其在资源竞争时共享的cpu资源就越大。例如，一个进程在4核cpu上运行占有的cpu时间是在2核机器上的2倍。这个控制称为soft limits（软限制），当服务器上运行进程既有来自cloudera manager管理的角色，同时有其他系统进程时，那个设置就没有效果了。

在多租户环境中，所有的Impala和Mapreduce进程，包含Impala后台进程，DN、TT等，可能会同时抢占cpu资源。设置cpu资源的x部分给Impala，每个节点上其余的资源的一半给dn和tt，这样即可达到将cpu的x部分给impala，mapreduce占有其余的资源。

这个对mapreduce来讲是保守的设置，这里假定dn和任务计算可以同时运行，最大化cpu的使用。如果你的mr负载远远低于集群资源总量，这个设置就不那么靠谱。例如，当你只有一个mr作业，通常作业输入输出使用的DN和TTcpu使用不会同时。只有当你有非常大的负载，那么整体上可以说dn和tt使用是同时的。


 







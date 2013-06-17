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

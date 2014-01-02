## FileOutputStream vs. FileWriter
  
当我们要用Java写文件时通常有两个方法：使用FileOutputStream或者FileWriter。

使用FileOutputStream:
<pre>
File fout = new File(file_location_string);
FileOutputStream fos = new FileOutputStream(fout);
BufferedWriter out = new BufferedWriter(new OutputStreamWriter(fos));
out.write("something");
</pre>

使用FileWriter:
<pre>
FileWriter fstream = new FileWriter(file_location_string);
BufferedWriter out = new BufferedWriter(fstream);
out.write("something");
</pre>

两种方法都可以写文件，但是使用FileOutputStream和FileWriter有什么分别呢？

对比这两个类有很多讨论，它们都实现了普通操作系统的i/o概念。我们不需要关心它们是怎么设计的，我们仅仅需要知道选择使用哪个，以及为什么使用它。

Java API中的解释：

> FileOutputStream是用来输出原始字节流的，如图像数据。要输出字符流，则使用FileWriter。

如果你对于设计模式也熟悉的话，FileWriter是典型的装饰者模式。我已经写过[教程](http://www.programcreek.com/2012/05/java-design-pattern-decorator-decorate-your-girlfriend/)来解释装饰者模式了，因为它对于很多设计来说都很重要。

FileOutputStream的一个应用就是[将文件转换成字节数组](http://www.programcreek.com/2009/02/java-convert-a-file-to-byte-array-then-convert-byte-array-to-a-file/)。

http://www.programcreek.com/2011/03/fileoutputstream-vs-filewriter/

## FileOutputStream vs. FileWriter
  
When we use Java to write something to a file, we can do it in the following two ways. One uses FileOutputStream, the other uses FileWriter.

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

Both will work, but what is the difference between FileOutputStream and FileWriter?

There are a lot of discussion on each of those classes, they both are good implements of file i/o concept that can be found in a general operating systems. However, we don’t care how it is designed, but only how to pick one of them and why pick it that way.

对比这两个类有很多讨论，它们都实现了普通操作系统的i/o概念。我们不需要关心它们是怎么设计的，我们仅仅需要知道选择使用哪个，以及为什么使用它。

From Java API Specification:

FileOutputStream is meant for writing streams of raw bytes such as image data. For writing streams of characters, consider using FileWriter.

If you are familiar with design patterns, FileWriter is a typical usage of Decorator pattern actually. I have use a simple tutorial to demonstrate the Decorator pattern, since it is very important and very useful for many designs.

One application of FileOutputStream is converting a file to a byte array.

Java API的解释：

FileOutputStream是用来写

http://www.programcreek.com/2011/03/fileoutputstream-vs-filewriter/

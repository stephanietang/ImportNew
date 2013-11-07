## HashMap和HashSet的区别

HashMap和HashSet的区别是Java面试中最常被问到的问题。如果没有涉及到Collection框架以及多线程的面试可以说不完整，而Collection框架的问题不涉及到HashSet和HashMap也可以说不完整。HashMap和HashSet都是collection框架的一部分，它们让我们能够使用对象的集合。collection框架有自己的接口和实现，主要分为Set接口，List接口和Queue接口。它们有各自的特点，Set的集合里不允许对象有重复的值，List允许有重复，它对集合中的对象进行索引，Queue的工作原理是FCFS算法(First Come, First Serve)。

首先让我们来看看什么是HashMap和HashSet，然后再来比较它们之间的分别。

### 什么是HashSet

HashSet实现了Set接口，它不允许集合中有重复的值，当我们提到HashSet时，第一件事情就是在将对象存储在HashSet之前，要先确保对象重写equals()和hashCode()方法，这样才能比较对象的值是否相等，以确保set中没有储存相等的对象。如果我们没有重写这两个方法，将会使用这个方法的默认实现。

public boolean add(Object o)方法用来在Set中添加元素，当元素值重复时则会立即返回false，如果成功添加的话会返回true。

### 什么是HashMap

HashMap实现了Map接口，Map接口对键值对进行映射。Map中不允许重复的键。Map接口有两个基本的实现，HashMap和TreeMap。TreeMap保存了对象的排列次序，而HashMap则不能。HashMap允许键和值为null。HashMap是非synchronized的，但collection框架提供方法能保证HashMap synchronized，这样多个线程同时访问HashMap时，能保证只有一个线程更改Map。

public Object put(Object Key,Object value)方法用来将元素添加到map中。

你可以阅读[这篇文章](http://javarevisited.blogspot.com/2011/02/how-hashmap-works-in-java.html)看看HashMap的工作原理，以及[这篇文章](http://javarevisited.blogspot.com/2010/10/difference-between-hashmap-and.html)看看HashMap和HashTable的区别。


### HashSet和HashMap的区别

<table>
<tr>
  <td>*HashMap*</td>
  <td>*HashSet*</td>
</tr>
<tr>
  <td>HashMap实现了Map接口</td>
  <td>HashSet实现了Set接口</td>
</tr>
<tr>
  <td>HashMap储存键值对</td>
  <td>HashSet仅仅存储对象</td>
</tr>
<tr>
  <td>使用put()方法将元素放入map中</td>
  <td>使用add()方法将元素放入set中</td>
</tr>
<tr>
  <td>HashMap中使用键对象来计算hashcode值</td>
  <td>HashSet使用成员对象来计算hashcode值，</td>
</tr>
<tr>
  <td>HashMap比较快，因为</td>
  <td></td>
</tr>
</table>



In hash map hashcode value is calculated using key object
Here member object is used for calculating hashcode value which can be same for two objects so equal () method is used to check for equality if it returns false that means two objects are different.
HashMap is faster than hashset because unique key is used to access object
HashSet is slower than Hashmap


Please let me know if you need any other difference between HashSet and HashMap in Java and I will add them here.




http://javarevisited.blogspot.hk/2011/09/difference-hashmap-vs-hashset-java.html

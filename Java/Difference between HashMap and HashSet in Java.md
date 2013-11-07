## HashMap和HashSet的区别

HashMap和HashSet的区别是Java面试中最常被问到的问题。如果没有涉及到Collection框架以及多线程的面试可以说不完整，而Collection框架的问题不涉及到HashSet和HashMap也可以说不完整。HashMap和HashSet都是collection框架的一部分，它们让我们能够使用对象的集合。collection框架有自己的接口和实现，主要分为Set接口，List接口和Queue接口。它们有各自的特点，Set的集合里不允许对象有重复的值，List允许有重复，它对集合中的对象进行索引，Queue的工作原理是FCFS算法(First Come, First Serve)。

首先让我们来看看什么是HashMap和HashSet，然后再来比较它们之间的分别。

What is HashSet in Java?

HashSet is implementation of Set Interface which does not allow duplicate value all the methods which are in Collection Framework are also in Set Interface by default but when we are talking about Hash set the main thing is objects which are going to be stored in HashSet must override equals() and hashCode() method so that we can check for equality and no duplicate value are stored in our set.if we have created our own objects we need to implement hashCode() and equal() in such a manner that will be able to compare objects correctly when storing in a set so that duplicate objects are not stored,if we have not override this method objects will take default implementation of this method.


public boolean add(Object o)  Method is used to add element in a set which returns false if it’s a duplicate value in case of  HashSet otherwise returns true if added successfully.

### 什么是HashSet

HashSet实现了Set接口，它不允许集合中有重复的值，Collection框架所有的方法Set接口中都有，但是当我们讨论HashSet时，在将对象存储在HashSet之前，要先要确保对象重写equals()和hashCode()方法，这样才能比较对象的值是否相等，以确保set中没有储存相等的对象。如果我们没有重写这两个方法，将会使用这个方法的默认实现。

What is HashMap?


HashMap is a implementation of Map Interface, which maps a key to value.Duplicate keys are not allowed in a map.Basically map Interface has two implementation classes HashMap and TreeMap the main difference is TreeMap maintains order of the objects but HashMap will not.HashMap allows null values and null keys.HashMap is not synchronized,but collection framework provide methods so that we can make them synchronized if multiple threads are going to access our hashmap and one thread is structurally change our map.

public Object put(Object Key,Object value) method is used to add element in map.

### 什么是HashMap

HashMap实现了Map接口，Map接口对键值对进行映射。Map中不允许重复的键。Map接口有两个基本的实现，HashMap和TreeMap。TreeMap保存了对象的排列次序，而HashMap则不能。HashMap允许键和值为null。HashMap并没有同步(synchronized)，但collection框架提供方法能保证HashMap同步，这样多个线程同时访问HashMap时，能保证只有一个线程更改Map。

You can read more about HashMap in my article How HashMap works in Java and Difference between HashMap and Hashtable in Java

你可以阅读[这篇文章]()看看HashMap和HashTable的区别。

Difference between HashSet and HashMap in Java

Following are some differences between HashMap and Hashset:

### HashSet和HashMap的区别

Hash Map
Hash Set
HashMap  is a implementation of Map interface
HashSet is an implementation of Set Interface
HashMap Stores data in form of  key value pair
HashSet Store only objects
Put method is used to add element in map
Add method is used to add element is Set
In hash map hashcode value is calculated using key object
Here member object is used for calculating hashcode value which can be same for two objects so equal () method is used to check for equality if it returns false that means two objects are different.
HashMap is faster than hashset because unique key is used to access object
HashSet is slower than Hashmap


Please let me know if you need any other difference between HashSet and HashMap in Java and I will add them here.




http://javarevisited.blogspot.hk/2011/09/difference-hashmap-vs-hashset-java.html
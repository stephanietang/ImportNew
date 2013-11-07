Difference between HashMap and HashTable? Can we make hashmap synchronized?

## HashMap和HashTable的区别？我们是否能使HashMap synchronized?
 
HashMap vs Hashtable in Java

Difference between HashMap and Hashtable in Java question oftenly asked in core Java interviews to check whether candidate understand correct usage of collection classes and aware of alternative solutions available. Along with How HashMap internally works in Java and ArrayList vs Vector, this  is one of the oldest question from Collection framework in Java. Hashtable is a legacy Collection class and it's there in Java API from long time but it got refactored to implement Map interface in Java 4 and from there Hashtable became part of Java Collection framework. Hashtable vs HashMap in Java is so popular a question that it can top any list of Java Collection interview Question. You just can't afford not to prepare HashMap vs Hashtable before going to any Java programming interview. In this Java article we will not only see some important differences between HashMap and Hashtable but also some similarities between these two collection classes. Let's first see How different they are :

HashMap和HashTable的比较是Java面试中的常见问题，用来考验程序员是否能够正确使用collection的类以及是否可以随机应变使用多种思路解决问题。HashMap的工作原理、ArrayList与Vector的比较以及这个问题是有关Java Collection框架的最老的问题。

Difference between HashMap and Hashtable in Java

Both HashMap and Hashtable implements Map interface but there are some significant difference between them which is important to remember before deciding whether to use HashMap or Hashtable in Java. Some of them is thread-safety, synchronization and speed. here are those differences :

1. The HashMap class is roughly equivalent to Hashtable, except that it is non synchronized and permits nulls. (HashMap allows null values as key and value whereas Hashtable doesn't allow nulls).

2. One of the major differences between HashMap and Hashtable is that HashMap is non synchronized whereas Hashtable is synchronized, which means Hashtable is thread-safe and can be shared between multiple threads but HashMap can not be shared between multiple threads without proper synchronization. Java 5 introduces ConcurrentHashMap which is an alternative of Hashtable and provides better scalability than Hashtable in Java.

3. Another significant difference between HashMap vs Hashtable is that Iterator in the HashMap is  a fail-fast iterator  while the enumerator for the Hashtable is not and throw ConcurrentModificationException if any other Thread modifies the map structurally  by adding or removing any element except Iterator's own remove() method. But this is not a guaranteed behavior and will be done by JVM on best effort. This is also an important difference between Enumeration and Iterator in Java. 

4. One more notable difference between Hashtable and HashMap is that because of thread-safety and synchronization Hashtable is much slower than HashMap if used in Single threaded environment. So if you don't need synchronization and HashMap is only used by one thread, it out perform Hashtable in Java.

5. HashMap does not guarantee that the order of the map will remain constant over time.

Difference between HashMap and Hashtable in Java | HashMap vs Hashtable




Note on Some Important Terms
1)Synchronized means only one Thread can modify a hash table at one point of time. Basically, it means that any thread before performing an update on a Hashtable will have to acquire a lock on the object while others will wait for lock to be released.

2)Fail-safe is relevant from the context of iterators. If an Iterator or ListIterator has been created on a collection object and some other thread tries to modify the collection object "structurally", a concurrent modification exception will be thrown. It is possible for other threads though to invoke "set" method since it doesn't modify the collection "structurally". However, if prior to calling "set", the collection has been modified structurally, "IllegalArgumentException" will be thrown.

3)Structurally modification means deleting or inserting element which could effectively change the structure of map.

HashMap can be synchronized by

Map m = Collections.synchronizeMap(hashMap);

In Summary there are significant differences between Hashtable and HashMap in Java e.g. thread-safety and speed and based upon that only use Hashtable if you absolutely need thread-safety, if you are running Java 5 consider using ConcurrentHashMap in Java.

http://javarevisited.blogspot.hk/2010/10/difference-between-hashmap-and.html
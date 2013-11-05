Difference between HashMap and HashSet in Java

 
HashMap vs HashSet is the most frequently asked question during any core java interview and interview is not said completed until they will not cover the Collection Framework and multi-threading interview and collections are uncompleted without Covering Hash Set and Hash Map.
Both HashMap and HashSet are part of collection framework which allows us to work with collection of objects. Collection Framework has their own interfaces and implementation classes. Basically collection is divided as Set Interface, List and Queue Interfaces. All this interfaces have their own property also apart from they get from collection like Set allows Collection of objects but forbids duplicate value, List allows duplicate along with indexing.Queue woks on FCFS algorithm.

First we have one look on What HashMap and Hashset is then will go for Differences between HashSet and HashMap

What is HashSet in Java?

HashMap vs HashSet, difference between HashMap and Hashset in JavaHashSet  is implementation of Set Interface which does not allow duplicate value all the methods which are in Collection Framework are also in Set Interface by default but when we are talking about Hash set the main thing is objects which are going to be stored in HashSet must override equals() and hashCode() method so that we can check for equality and no duplicate value are stored in our set.if we have created our own objects we need to implement hashCode() and equal() in such a manner that will be able to compare objects correctly when storing in a set so that duplicate objects are not stored,if we have not override this method objects will take default implementation of this method.


public boolean add(Object o)  Method is used to add element in a set which returns false if it’s a duplicate value in case of  HashSet otherwise returns true if added successfully.


What is HashMap?


HashMap is a implementation of Map Interface, which maps a key to value.Duplicate keys are not allowed in a map.Basically map Interface has two implementation classes HashMap and TreeMap the main difference is TreeMap maintains order of the objects but HashMap will not.HashMap allows null values and null keys.HashMap is not synchronized,but collection framework provide methods so that we can make them synchronized if multiple threads are going to access our hashmap and one thread is structurally change our map.

public Object put(Object Key,Object value) method is used to add element in map.

You can read more about HashMap in my article How HashMap works in Java and Difference between HashMap and hashtable in Java

Difference between HashSet and HashMap in Java

Following are some differences between HashMap and Hashset:


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
How HashMap works in Java

## HashMap的工作原理
 
How HashMap  works in Java

How HashMap works in Java or sometime how get method work in HashMap is common questions on Java interviews now days. Almost everybody who worked in Java knows about HashMap, where to use HashMap or difference between Hashtable and HashMap then why this interview question becomes so special? Because of the depth it offers. It has become very popular java interview question in almost any senior or mid-senior level Java interviews. Investment banks mostly prefer to ask this question and some time even ask to implement your own HashMap based upon your coding aptitude. Introduction of ConcurrentHashMap and other concurrent collections has also made this questions as starting point to delve into more advanced feature. let's start the journey.

HashMap的工作原理是近年来常见的Java面试题。几乎每个Java程序员都知道HashMap，都知道哪里要用HashMap，知道HashTable和HashMap之间的区别，那么为何这道面试题如此特殊呢？是因为这道题考察的深度很深。这题经常出现在高级或中高级面试中。投资银行更喜欢问这个问题，甚至会要求你实现HashMap来考察你的编程能力。ConcurrentHashMap和其它同步集合的引入让这道题变得更加复杂。让我们开始探索的旅程吧！

Questions start with simple statement 


"Have you used HashMap before" or  "What is HashMap? Why do we use it “
Almost everybody answers this with yes and then interviewee keep talking about common facts about HashMap like HashMap accept null while Hashtable doesn't, HashMap is not synchronized, HashMap is fast and so on along with basics like its stores key and value pairs etc. This shows that person has used HashMap  and quite familiar with the functionality HashMap offers but interview takes a sharp turn from here and next set of follow-up questions gets more detailed about fundamentals involved with HashMap in Java . Interviewer struck back with questions like

## 先来个简单的问题

“你用过HashMap吗？” “什么是HashMap？你为什么用到它？”

几乎每个人都会回答“是的”，然后回答HashMap的一些特性，譬如HashMap可以接受null键值和值，而HashTable则不能；HashMap是非synchronized;HashMap很快；以及HashMap储存的是键值对等等。这显示出你已经用过HashMap，而且对它相当的熟悉。但是面试官来个急转直下，从此刻开始问出一些刁钻的问题，关于HashMap的更多基础的细节。面试官可能会问出下面的问题：

"Do you Know how HashMap works in Java” or "How does get () method of HashMap works in Java"
And then you get answers like I don't bother its standard Java API, you better look code on Java source or Open JDK; I can find it out in Google at any time etc. But some interviewee definitely answer this and will say "HashMap works on principle of hashing, we have put(key, value) and get(key) method for storing and retrieving Objects from HashMap. When we pass Key and Value object  to put() method on Java HashMap, HashMap implementation calls hashCode method on Key object and applies returned hashcode into its own hashing function to find a bucket location for storing Entry object, important point to mention is that HashMap in Java stores both key and value object as Map.Entry in bucket which is essential to understand the retrieving logic. If people fails to recognize this and say it only stores Value in the bucket they will fail to explain the retrieving logic of any object stored in Java HashMap . This answer is very much acceptable and does make sense that interviewee has fair bit of knowledge on how hashing works and how HashMap  works in Java. But this is just start of story and confusion increases when you put interviewee on scenarios faced by Java developers on day by day basis. Next question could be about collision detection and collision resolution in Java HashMap  e.g. 

“你知道HashMap的工作原理吗？” “你知道HashMap的get()方法的工作原理吗？”

你也许会回答“我没有详查标准的Java API，你可以看看Java源代码或者Open JDK。”“我可以用Google找到答案。”

但一些面试者可能可以给出答案，“HashMap是基于hashing的原理，我们使用put(key, value)存储对象到HashMap中，使用get(key)从HashMap中获取对象。当我们给put()方法传递键和值时，我们先对键调用hashCode()方法，返回的hashCode用于找到bucket位置来储存Entry对象。”这里关键点在于指出，HashMap是在bucket中储存键对象和值对象，作为Map.Entry。这一点有助于理解获取对象的逻辑。如果你没有意识到这一点，或者错误的认为仅仅只在bucket中存储值的话，你将不会回答如何从HashMap中获取对象的逻辑。这个答案相当的正确，也显示出面试者确实知道hashing以及HashMap的工作原理。但是这仅仅是故事的开始，当面试官加入一些Java程序员每天要碰到的实际场景的时候，会发生很多模棱两可的答案。下个问题可能是关于HashMap中的碰撞探测(collision detection)以及碰撞的解决方法：

"What will happen if two different objects have same hashcode?”
Now from here onwards real confusion starts, Some time candidate will say that since hashcode is equal, both objects are equal and HashMap  will throw exception or not store them again etc, Then you might want to remind them about equals() and hashCode() contract  that two unequal object in Java can have same hashcode. Some will give up at this point and few will move ahead and say "Since hashcode is same, bucket location would be same and collision will occur in HashMap, Since HashMap use LinkedList to store object, this entry (object of Map.Entry comprise key and value )  will be stored in LinkedList. Great this answer make sense though there are many collision resolution methods available this is simplest and HashMap in Java does follow this. But story does not end here and interviewer asks

"How will you retrieve Value object  if two Keys will have same hashcode?”
how HashMap works internally in JavaInterviewee will say we will call get() method and then HashMap uses Key Object's hashcode to find out bucket location and retrieves Value object but then you need to remind him that there are two Value objects are stored in same bucket , so they will say about traversal in LinkedList until we find the value object , then you ask how do you identify value object because you don't  have value object to compare ,Until they know that HashMap  stores both Key and Value in LinkedList node or as Map.Entry they won't be able to resolve this issue and will try and fail.


But those bunch of people who remember this key information will say that after finding bucket location , we will call keys.equals() method to identify correct node in LinkedList and return associated value object for that key in Java HashMap . Perfect this is the correct answer.

In many cases interviewee fails at this stage because they get confused between hashCode() and equals() or keys and values object in Java HashMap  which is pretty obvious because they are dealing with the hashcode() in all previous questions and equals() come in picture only in case of retrieving value object from HashMap in Java. Some good developer point out here that using immutable, final object with proper equals() and hashcode() implementation would act as perfect Java HashMap  keys and improve performance of Java HashMap  by reducing collision. Immutability also allows caching there hashcode of different keys which makes overall retrieval process very fast and suggest that String and various wrapper classes e.g. Integer very good keys in Java HashMap.

Now if you clear this entire Java HashMap interview,  You will be surprised by this very interesting question "What happens On HashMap in Java if the size of the HashMap  exceeds a given threshold defined by load factor ?". Until you know how HashMap  works exactly you won't be able to answer this question. If the size of the Map exceeds a given threshold defined by load-factor e.g. if load factor is .75 it will act to re-size the map once it filled 75%. Similar to other collection classes like ArrayList,  Java HashMap re-size itself by creating a new bucket array of size twice of previous size of HashMap , and then start putting every old element into that new bucket array. This process is called rehashing because it also applies hash function to find new bucket location. 

If you manage to answer this question on HashMap in Java you will be greeted by "do you see any problem with resizing of HashMap  in Java" , you might not be able to pick the context and then he will try to give you hint about multiple thread accessing the Java HashMap and potentially looking for race condition on HashMap  in Java. 

So the answer is Yes there is potential race condition exists while resizing HashMap in Java, if two thread at the same time found that now HashMap needs resizing and they both try to resizing. on the process of resizing of HashMap in Java , the element in bucket which is stored in linked list get reversed in order during there migration to new bucket because java HashMap  doesn't append the new element at tail instead it append new element at head to avoid tail traversing. If race condition happens then you will end up with an infinite loop. Though this point you can potentially argue that what the hell makes you think to use HashMap  in multi-threaded environment to interviewer :)

Few more question on HashMap in Java which is contributed by readers of Javarevisited blog  :
1) Why String, Integer and other wrapper classes are considered good keys ?
String, Integer and other wrapper classes are natural candidates of HashMap key, and String is most frequently used key as well because String is immutable and final,and overrides equals and hashcode() method. Other wrapper class also shares similar property. Immutabiility is required, in order to prevent changes on fields used to calculate hashCode() because if key object return different hashCode during insertion and retrieval than it won't be possible to get object from HashMap. Immutability is best as it offers other advantages as well like thread-safety, If you can  keep your hashCode same by only making certain fields final, then you go for that as well. Since equals() and hashCode() method is used during reterival of value object from HashMap, its important that key object correctly override these methods and follow contact. If unequal object return different hashcode than chances of collision will be less which subsequently improve performance of HashMap.

2) Can we use any custom object as key in HashMap ?
This is an extension of previous questions. Ofcourse you can use any Object as key in Java HashMap provided it follows equals and hashCode contract and its hashCode should not vary once the object is inserted into Map. If custom object is Immutable than this will be already taken care because you can not change it once created.

3) Can we use ConcurrentHashMap in place of Hashtable ?
This is another question which getting popular due to increasing popularity of ConcurrentHashMap. Since we know Hashtable is synchronized but ConcurrentHashMap provides better concurrency by only locking portion of map determined by concurrency level. ConcurrentHashMap is certainly introduced as Hashtable and can be used in place of it but Hashtable provide stronger thread-safety than ConcurrentHashMap. See my post difference between Hashtable and ConcurrentHashMap for more details.

Personally, I like this question because of its depth and number of concept it touches indirectly, if you look at questions asked during interview this HashMap  questions has verified

Concept of hashing
Collision resolution in HashMap
Use of equals () and hashCode () and there importance in HashMap?
Benefit of immutable object?
Race condition on HashMap  in Java
Resizing of Java HashMap

Just to summarize here are the answers which does makes sense for above questions

How HashMap  works in Java
HashMap  works on principle of hashing, we have put() and get() method for storing and retrieving object form HashMap .When we pass an both key and value to put() method to store on HashMap , it uses key object hashcode() method to calculate hashcode and they by applying hashing on that hashcode it identifies bucket location for storing value object. While retrieving it uses key object equals method to find out correct key value pair and return value object associated with that key. HashMap  uses linked list in case of collision and object will be stored in next node of linked list.
Also HashMap  stores both key+value tuple in every node of linked list.

What will happen if two different HashMap  key objects have same hashcode?
They will be stored in same bucket but no next node of linked list. And keys equals () method will be used to identify correct key value pair in HashMap .

In terms of usage Java HashMap is very versatile and I have mostly used HashMap as cache in electronic trading application I have worked . Since finance domain used Java heavily and due to performance reason we need caching HashMap and ConcurrentHashMap  comes as very handy there. You can also check following articles form Javarevisited to learn more about HashMap and Hashtable in Java :


http://javarevisited.blogspot.hk/2011/02/how-hashmap-works-in-java.html
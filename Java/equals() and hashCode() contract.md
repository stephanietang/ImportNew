## Java中的equals()和hashCode()契约

java.lang.Object类中有两个非常重要的方法：

<pre>
public boolean equals(Object obj)
public int hashCode()
</pre>

理解这两个方法非常的重要，尤其是将用户自定义的对象添加到Map中的时候。有时候就算是久经沙场的老程序员也弄不清楚该如何正确使用它们。这篇文章中，我将用一个例子让大家看看大家经常会犯的错误，然后解释equals()和hashCode()的正确的使用方法。

### 1. 常见错误

常见的错误如下：

<pre>
import java.util.HashMap;
 
public class Apple {
	private String color;
 
	public Apple(String color) {
		this.color = color;
	}
 
	public boolean equals(Object obj) {
		if (!(obj instanceof Apple))
			return false;	
		if (obj == this)
			return true;
		return this.color == ((Apple) obj).color;
	}
 
	public static void main(String[] args) {
		Apple a1 = new Apple("green");
		Apple a2 = new Apple("red");
 
		//hashMap stores apple type and its quantity
		HashMap<Apple, Integer> m = new HashMap<Apple, Integer>();
		m.put(a1, 10);
		m.put(a2, 20);
		System.out.println(m.get(new Apple("green")));
	}
}
</pre>

这个例子中，一个“绿苹果”的对象成功添加到hashMap中了，但是当我们要取出这个“绿苹果”的时候，却得不到这个对象，程序返回null。我们使用调试器却发现在hashMap中已经存储了这个对象。

## 2. hashCode()引起的问题

这个问题是因为"hashCode()"方法没有被重写。Java中equals()和hashCode()有一个契约：

- 1. 如果两个对象相等的话，它们的hash code必须相等；
- 2. 但如果两个对象的hash code相等的话，这两个对象不一定相等。

Map的结构能够快速找到一个对象，而不是进行较慢的线性查找。使用hash过的键来定位对象分两步。Map可以看作是数组的数组。第一个数组的索引就是对键采用hashCode()计算出来的值，再在这个位置上查找第二个数组，使用键的equals()方法来进行线性查找，直到找到要找的对象。

Object类中的hashCode()对于不同的对象返回不同的整数，所以上面的例子中，不同的对象（即使相同的类型）也返回不同的hash值。

Hash码就像是一个存储空间的序列，不同的东西放在不同的存储空间中。将不同的东西整理放在不同的空间中（而不是堆积在一个空间中）更高效。所以能够均匀的分散hash码是再好不过了。

上面错误的解决方法就是在类中增加hashCode方法。这里我仅仅使用颜色的长度来计算hash码。

<pre>
public int hashCode(){
	return this.color.length();	
}
</pre>

http://www.programcreek.com/2011/07/java-equals-and-hashcode-contract/

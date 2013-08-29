http://blog.codecentric.de/en/2013/04/again-10-tips-on-java-debugging-with-eclipse/

Eclipse的调试功能的10个小窍门

你可以已经看过一些类似“关于调试的N件事”的文章了。但我想我每天大概在调试上会花掉1个小时，这是非常多的时间了。所以非常值得我们来了解一些用得到的功能，可以帮我们节约很多时间。所以在这个主题上值得我再来写一篇文章。

### 第一条： 不要过分的调试！

有关调试的第一条要牢记的便是这条很疯狂的口号！但是我必须要在这里再说一遍：不要过分的调试！试着将复杂的逻辑分解成独立的小单元，然后写单元测试代码来保证小单元的正确运行。我经常看到某些人会这么做：在一个大型的Web应用上点击，填了几个表单，跳转了多个页面，只是为了确认最后一个页面的结果的正确性，最后在调试视图下来开发代码。

在你开启tomcat之前，应该要先问问自己：有没有什么方法可以用单元测试来检测代码的行为呢？你可以找到很多教你如何写出好的代码。而这里我主要来谈一谈eclipse的调试功能，你可能不知道，或者长时间以来有些淡忘的功能。

### Breakpoint View : Conditional Breakpoints

Extreme useful if you are interested only in a special constellation of your application. For example if you want to see the 13th run in a loop or you are debugging functionality in an abstract super class and you only want to see one concrete implementation. You can setup the conditions in the breakpoint-view or with the contextmenu on the blue breakpoint-marker next to your code (“Breakpoint Properties”). You can choose to suspend when your code-snippet gets true or when the value of your snippet changes.

### Variables View: Show Logical Structure

If you want to see the values of a Map or a List in the variables view, it’s not always that easy with the default setting of eclipse. If you are using a HashMap for example, you have to click through the physical entries and you are confronted with implementation details of a HashMap. But there is a small button above the variables – “Show Logical Structure”. Very handy, especially if you don’t have meaningful toString()-methods for the objects in your structure. My boss showed me this feature a few weeks ago. You know, he is the guy who is working with PowerPoint or Excel the most time. What a shame for a developer like me ;-)

### Variables View: Change Value…

Instead of restarting your debug session with some slightly changed input data, let’s say entered in a web-form – you can also change the values of your variables directly during debugging. Sometimes you can safe some time and sometimes you can simulate some strange behaviour with that feature a bit easier.

### Display View

Do you know the “Display View”? You can activate it during debugging via “Window” -> “Show View” -> “Display”. Now there should be a blank new view in your Eclipse. You can use this view to enter and evaluate new code. The code is executed within the context of the current debugging positions, which means that you can use all your variables and even content assist. To execute your code, just mark it and use the context-menu or CTRL+U (execute) or CTRL+SHIFT+I (inspect).
 
### Navigation: Drop to Frame

I think everybody knows “Step Into”, “Step over” and maybe “Step return”. That are the basics to navigate through your code while you are debugging. I want to name 2 further ways to navigate, which I like very much. The first is “Drop to Frame”. With that feature you have the possibility to go back in time ;-) You can simply go to a point in your java stackframe where you have been before. It happens to me quite often, that I am debugging and then missing the point where I have to pay attention. With the “Drop to Frame”-Feature I can then run the code again very simple.

### Navigation: Step into Selection

The second one is “Step into Selection”. This is a pretty easy one, but not many people are using it. For that, you just have to press Ctrl+Alt and click on a method name, where you want to go. Very handy, very fast. Compared to the usual “Step Into” this is much better for example if you want to enter a method with many parameters and you one step trough all the parameter evaluations. “Run to line” is also a nice feature. Just place your cursor in front of the line where you want to stop and hit “CTRL+R”.
 

### Navigation: Use your keyboard
You are faster, if you avoid to use the mouse. You should know at least the following Shortcuts:

F5 – “Step Into”
F6 – “Step Over”
F7 – “Step Return”
F8 – “Resume”
Ctrl+Shift+B – “Toggle Breakpoint”
Ctrl+Shift+I – “Inspect”

### Breakpoint View: Watchpoints

What the hell is changing this variable?! Sometimes it is usefull to create a watchpoint. Then the debugger stops, whenever the watched field is changed or just read – you can configure that. Just doubleclick on a field, then you can see the watchpoint in the Breakpoint View and edit the properties. You can even configure a hit count, means that the debugger only stops, when the entered hit count is reached. This also works for usual breakpoints.
 
### Human Readable Objects

The Variables View is using the toString-Method of your objects to display the value. Because of this it is very, very helpful
to provide good toString-implementations. The javadoc of the default-toString-implementation in java.lang.Object recommends the same:

<pre>
* Returns a string representation of the object. In general, the
* <code>toString</code> method returns a string that
* "textually represents" this object. The result should
* be a concise but informative representation that is easy for a
* person to read.
* It is recommended that all subclasses override this method.
[...]
</pre>

You can have a look at the ToStringBuilder in commons-lang. It provides some functionality to write – quote from the javadoc – “good and consistent” toString-methods.

If you cannot modify the toString-implementation, for example if you are working with frameworks or you have to use a foreign API, it might be an option to create an “Detail Formatter” in Eclipse. To do that, you have to right click an object in the variables view and click on “New Detail Formatter…”. Then you can provide some code to display this type of Object in the future.

http://javarevisited.blogspot.hk/2012/03/10-object-oriented-design-principles.html

Java程序员应该知道的10个面向对象理论

面向对象理论是面向对象编程的核心，但是我发现大部分Java程序员热衷于像单例模式、装饰者模式或观察者模式这样的设计模式，而并没有十分注意学习面向对象的分析和设计。学习面向编程的基础(如抽象，封装，多态，继承等)是非常重要的，而运用它们来设计干净的模块也同样重要。我也认识很多不同等级的程序员，他们没有听过这些面向对象理论，或者不知道某个设计理论有什么好处，或者如何在编码中使用这些设计理论。

我们起码要设计出高度一致而且松散耦合的代码。Apache和Sun的源代码就是学习Java面向对象理论的非常好的例子。JDK遵循了一些设计模式，譬如在BorderFactory中使用工厂模式，Runtime类中使用单例模式，java.io中的许多类中使用装饰者模式。如果你真的对Java编程感兴趣，请先阅读Joshua Bloch的[Effective Java](http://www.amazon.com/gp/product/0321356683/ref=as_li_ss_tl?ie=UTF8&tag=javamysqlanta-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0321356683)，正是他参与编写了Java API。另外两本我喜欢的关于设计模式的书还有，Kathy Sierra等编写的的[Head First Design Pattern](http://www.amazon.com/gp/product/0596007124/ref=as_li_ss_tl?ie=UTF8&tag=javamysqlanta-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0596007124)和[Head First Object Oriented Analysis and Design](http://www.amazon.com/gp/product/0596008678/ref=as_li_ss_tl?ie=UTF8&tag=javamysqlanta-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0596008678)。这些书帮助理解面向对象理论，并帮助我写出更好的代码。

学习任何设计理论或模式最好的方法就是现实世界中的例子，这篇文章只是要给还在学习阶段的程序员介绍面相对象理论。我想以下每一条都需要用一篇文章来详细介绍，我以后也会逐一介绍的，只是现在先来个快速浏览一下。

*避免重复，DRY(Don't repeat yourself)*

面相对设计理论的第一条就是避免重复，不要写重复的代码，尽量将共同的功能放在一个地方。如果你准备在不同地方写同一段代码，那么只写一个方法。如果你不止一次硬编码某个值，那么将其声明成public final常量。这么做的好处就是容易维护。但是不要滥用这一条，重复不是指代码的重复，而是指功能的重复。譬如你有一段相同的代码来验证OrderID和SSN，但它们代表的意义并不相同。如果你将两个不同的功能合并在一起，当OrderID更改了格式之后，那么检验SSN的代码就会失效。所以要警觉这种耦合，不要讲任何相似但不相关的代码合并在一起。

*将变化封装起来*

在软件领域唯一不变的就是“变化”。所以最好将你觉得将来会有改变的代码封装起来。这样做的好处就是更容易测试和维护正确的被封装的代码。你应该先将变量声明成private，然后有需要的话再扩大访问权限，如将private变成protected。Java中很多设计模式都使用了封装，工厂设计模式就是封装的一个例子，它封装了对象的创建，如果要引入新的“产品”，也不必更改现有的代码。

*开放且封闭的设计理论(Open Closed Design Principle)*

类、方法以及功能应该对扩展开放(新的功能)，而对更改封闭。这是另一个优美的"SOLID"设计理论，这保证了有人更改已经经过测试了的代码。如果你要加入新的功能，你必须要先测试它，这正是开放且封闭的设计理论的目标。另外，Open Closed principle正是SOLID中的O的缩写。

*单一责任原理(Single Responsibility Principle (SRP))*

单一责任原理是另外一条"SOLID"设计理论，代表其中的“S”。每次一个类只有一个更改的原因，或者一个类只应该完成单一的功能。如果你将多过一个功能放在一个类中，它会将两个功能耦合在一起，如果你改变了其中的一个功能，可能会破坏另外一个功能，这样便需要更多的测试以确保上线时不出现什么岔子。

*依赖注入或反转原理*

容器会提供依赖注入，Spring非常好的实现了依赖注入。这条原理的美妙之处在于，每个被注入的类很容易的测试和维护，因为这些对象的创建代码都集中在容器中，有多种方法都可以进行依赖注射，譬如一些AOP框架如AspectJ使用的字节码注入(bytecode instrumentation)，以及Spring中使用的代理器(proxy)。来看看[这个依赖注射的例子]()吧。这一条正是SOLID中的"D"。

*多用组合，少用继承*

如果可能的话，多用组合，少用继承。可能有的人会不同意，但我确实发现组合的灵活性高过继承。组合可以在运行时通过设置某个属性以及通过接口来组合某个类，我们可以使用多态，这样就能随时改变类的行为，大大提高了灵活性。[Effective Java](http://www.amazon.com/gp/product/0321356683/ref=as_li_ss_tl?ie=UTF8&tag=javamysqlanta-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0321356683)也更倾向于使用组合。

*Liskov替代原理(Liskov Substitution Principle (LSP))*

根据Liskov替代原理，子类必须可以替代父类，也就是使用父类的方法，也能够没有任何问题的和子类对象也兼容。LSP和单一责任原则以及接口分离原则的关系紧密。如果一个类比子类的功能要多，子类不能支持父类中的某些功能的话，就违反了LSP。为了遵循LSP原理，子类需要改进父类的功能，而不是减少功能。LSP代表SOLID中的"L"。

*接口分离理论(Interface Segregation principle (ISP))*

接口分离理论强调，如果客户端没有使用一个接口的话，就不要实现它。当一个接口包含两个以上的功能，如果客户端仅仅需要其中某个功能，而不需要另外一个，那么就不要实现它。接口的设计是件非常复杂的工作，因为一旦你发布了接口之后，就再也无法保证不破坏现有实现的情况下更改接口。分离接口的另一个好处就是，因为必须要实现方法才能使用接口，所以如果仅仅只有单一的功能，那么要实现的方法也会减少。

*针对接口编程，而不是针对实现编程*

尽量针对接口编程，这样如果要引入任何新的接口，也有足够的灵活性。在变量的类型、方法的返回类型以及参量类型中使用接口类型。很多程序员都建议这么做，包括[Effective Java](http://www.amazon.com/gp/product/0321356683/ref=as_li_ss_tl?ie=UTF8&tag=javamysqlanta-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0321356683)和[head first design pattern](http://www.amazon.com/gp/product/0596007124/ref=as_li_ss_tl?ie=UTF8&tag=javamysqlanta-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=0596007124)等书。

*代理理论(Delegation principle)*

不要所有的事情都自己做，有时候要将任务代理给相应的类去做。运用代理模式最经典的例子就是equals()和hashCode()方法。为了比较两个对象的相等与否，我们没有用客户端代码去比较，而是让对象自己去比较。这么做的好处就是减少代码的重复，更容易更改行为。

所有的这些面相对象理论都能帮助你写出更灵活、高度一致且低耦合的代码。理论是第一步，更重要的是运用这些设计理论的能力。找出违反这些设计理论的地方，但是就像这个世界上没有什么是完美的一样，不要尝试着用设计模式和理论解决一切问题，因为它们往往是针对大型的企业级项目，有着更长的运行周期。换句话说小型的项目不一定值得这么做。

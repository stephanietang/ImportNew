
SPRING SECURITY JAVA配置：简介

昨天我发布了[Spring Security Java配置支持](http://www.springsource.org/node/22640)，并发布了[Spring Security 3.2.0.M2](http://www.springsource.org/node/22639)，这其中就包含了对Java配置的支持。

Spring Security对Java配置提供支持的本意是提供对[XML命名空间配置](http://static.springsource.org/spring-security/site/docs/3.1.x/reference/appendix-namespace.html)的替代方案。它设计成可扩展的，以便Spring Security的扩展项目可以很好的与Java配置支持配合工作。

这是"Spring Security Java配置"系列的第一篇，我将会作个总体的介绍。

> “需要的版本”

不管你如何集成Spring Security，你都需要使用Spring 3.2.3.RELEASE或者以上的版本，这可以避免[SPR-10546](https://jira.springsource.org/browse/SPR-10546)。

###　在哪儿获取

在你开始之前，我想告诉你有两个模块包含了Spring Security的Java配置支持。

#### 从Spring Security 3.2.0.M2+中获取

Spring Security Java配置已经拷贝到了Spring Security 3.2.0.M2+的源代码中了。这就意味着，如果你用的是Spring Security 3.2.0.M2+，你需要保证spring-security-config jar已经放在classpath中了。例如，你可以这样配置Maven pom.xml：

<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-config</artifactId>
  <version>3.2.0.M2</version>
</dependency>
…
<repository>
  <id>repository.springsource.milestone</id>
  <name>SpringSource Milestone Repository</name>
  <url>http://repo.springsource.org/milestone</url>
</repository>

#### 从Spring Security 3.1.4.RELEASE+中获取

为了鼓励开发者更多的使用Spring Security Java配置，现在也提供一个单独的模块，叫作spring-security-javaconfig。例如，你可以在Maven的pom.xml中进行如下配置：

<dependency>
  <groupId>org.springframework.security</groupId>
  <artifactId>spring-security-javaconfig</artifactId>
  <version>1.0.0.M1</version>
</dependency>
…
<repository>
  <id>repository.springsource.milestone</id>
  <name>SpringSource Milestone Repository</name>
  <url>http://repo.springsource.org/milestone</url>
</repository>

> 将来的更新计划

目前没有更新spring-security-javaconfig的计划。所以，我们希望用户在Spring Security 3.2发布的时候，更新到Spring Security 3.2。

### 欢迎反馈

如果你发现了bug，或者觉得有什么地方值得改进，请你不要犹豫，给我们留言！我们希望听到你的想法，以便在大部分人获得代码之前，我们便确保代码的正确性。很早的尝试新功能是一种简单有效的回馈社区的方法，这样做的好处就是能帮助你获得你希望获得的功能。

请到“Java Config”目录下的[Spring Security JIRA](https://jira.springsource.org/browse/SEC)记录下任何问题。记录了一个JIRA之后，我们希望（当然并不是必须的）你在pull request中提交你的代码。你可以在[贡献者指引](https://github.com/SpringSource/spring-security/blob/master/CONTRIBUTING.md)中阅读更详细的步骤。如果你有任何不清楚的，请使用[Spring Security论坛](http://forum.springsource.org/forumdisplay.php?33-Security)或者[Stack Overflow](http://stackoverflow.com/questions/tagged/spring-security)，并使用"spring-security"标签(我会一直查看这个标签)。如果你针对这个博客有任何意见，也请留言。使用合适的工具对每个人来说都会带来便利。

## 结论

你应该已经知道Spring Security Java配置支持在多个地方都可以获得，你也知道在哪可以找到更新，在哪里记录问题。[下一篇](http://www.importnew.com/)，我们将介绍如何在一个web应用中使用Spring Security Java配置。
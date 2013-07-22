
## SPRING SECURITY JAVA配置：Web Security

在[前一篇](http://www.importnew.com/5520.html)，我已经介绍了Spring Security Java配置，也概括的介绍了一下这个项目方方面面。在这篇文章中，我们来看一看一个简单的基于web security配置的例子。之后我们再来作更多的个人定制。

### Hello Web Security

在这个部分，我们对一个基于web的security作一些基本的配置。可以分成四个部分：

- 更新依赖 - 我们已经在[前一篇](http://www.importnew.com/5520.html)文章中用Maven进行了示范
- 进行Spring Security配置 - 这个例子中，我们采用[WebSecurityConfigurerAdapter](http://blog.springsource.org/2013/07/03/spring-security-java-config-preview-web-security/#wsca)
- 确保Spring Security配置已经被加载了 - 我们采用[AbstractAnnotationConfigDispatcherServletInitializer](http://blog.springsource.org/2013/07/03/spring-security-java-config-preview-web-security/#aacdsi)
- 配置springSecurityFilterChain - 我们采用[AbstractSecurityWebApplicationInitializer](http://blog.springsource.org/2013/07/03/spring-security-java-config-preview-web-security/aswai)

### WebSecurityConfigurerAdapter

@EnableWebSecurity注解以及WebSecurityConfigurerAdapter一起配合提供基于web的security。扩展了WebSecurityConfigurerAdapter之后，再加上几行代码，我们就能实现以下的功能：

- 要求用户在进入你的应用的任何URL之前都进行验证
- 创建一个用户名是“user”，密码是“password”，角色是“ROLE_USER”的用户
- 启用HTTP Basic和基于表单的验证
- Spring Security将会自动生成一个登陆页面和登出成功页面

<pre class="brush: java; gutter: true">
@Configuration
@EnableWebSecurity
public class HelloWebSecurityConfiguration extends WebSecurityConfigurerAdapter {
 
  @Override
  protected void registerAuthentication(AuthenticationManagerBuilder auth) {
    auth.inMemoryAuthentication()
        .withUser("user").password("password").roles("USER");
  }
}
</pre>

作为参考，我们在这里也给出相似的XML配置，不过有几个特殊配置：

- Spring Security会生成一个登陆页面，验证失败页面和登出成功页面
- login-processing-url仅仅处理HTTP POST
- login-page仅仅通过HTTP GET进入

<http use-expressions="true">
  <intercept-url pattern="/**" access="authenticated"/>
  <logout
    logout-success-url="/login?logout"
    logout-url="/logout"
  />
  <form-login
    authentication-failure-url="/login?error"
    login-page="/login"
    login-processing-url="/login"
    password-parameter="password"
    username-parameter="username"
  />
</http>
<authentication-manager>
  <authentication-provider>
    <user-service>
      <user name="user"
          password="password"
          authorities="ROLE_USER"/>
    </user-service>
  </authentication-provider>
</authentication-manager>

### AbstractAnnotationConfigDispatcherServletInitializer

下一步就是保证ApplicationContext包含我们刚刚定义的HelloWebSecurityConfiguration。有[几种方法](http://static.springsource.org/spring/docs/current/javadoc-api/org/springframework/web/WebApplicationInitializer.html)都可行，我们这里使用[Spring的AbstractAnnotationConfigDispatcherServletInitializer](http://static.springsource.org/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-container-config)：
<pre class="brush: java; gutter: true">
  public class SpringWebMvcInitializer extends
  AbstractAnnotationConfigDispatcherServletInitializer {
 
  @Override
  protected Class<?>[] getRootConfigClasses() {
    return new Class[] { HelloWebSecurityConfiguration.class };
  }
  ...
}
</pre>

Spring Security通常在web.xml中包含下面几行代码进行初始化：

<!-- Creates the Spring Container shared by all Servlets and Filters -->
<listener>
  <listener-class>
    org.springframework.web.context.ContextLoaderListener
  </listener-class>
</listener>
 
<!-- Load all Spring XML configuration including our security.xml file -->
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>/WEB-INF/spring/*.xml</param-value>
</context-param>

### AbstractSecurity WebApplicationInitializer

最后一步，我们需要对springSecurityFilterChain定义映射路径。我们很容易通过扩展[AbstractSecurityWebApplicationInitializer](http://static.springsource.org/spring-security/site/docs/3.2.x/apidocs/org/springframework/security/web/context/AbstractSecurityWebApplicationInitializer.html)实现，并可以有选择的通过覆盖方法来定制映射。

下面是最基本的配置，它可以接受默认的映射路径，springSecurityFilterChain具有以下的特性：

- springSecurityFilterChain映射到了"/*"
- springSecurityFilterChain使用ERROR和REQUEST分派类型(dispatch type)
- springSecurityFilterChain插入到其它已经配置的servlet过滤器映射(servlet Filter mapping)之前

<pre class="brush: java; gutter: true">
public class SecurityWebApplicationInitializer
   extends AbstractSecurityWebApplicationInitializer {
}
</pre>

上面的代码等同于将这几行代码放在web.xml中：

<filter>
  <filter-name>springSecurityFilterChain</filter-name>
  <filter-class>
    org.springframework.web.filter.DelegatingFilterProxy
  </filter-class>
</filter>
 
<filter-mapping>
  <filter-name>springSecurityFilterChain</filter-name>
  <url-pattern>/*</url-pattern>
  <dispatcher>ERROR</dispatcher>
  <dispatcher>REQUEST</dispatcher>
</filter-mapping>

> WebApplicationInitializer的次序
在AbstractSecurityWebApplicationInitializer启动之后再加入的servlet过滤器映射，它们有可能会加在springSecurityFilterChain之前。除非这个应用不需要安全验证，否则springSecurityFilterChain需要放在其它所有的过滤器映射之前。@Order可以保证任何WebApplicationInitializer都使用特定的顺序加载。

### CustomWebSecurityConfigurerAdapter

这个HelloWebSecurityConfiguration范例，为我们很好的展示了Spring Security Java配置是如何工作的。让我们来看看更多定制的配置吧。

<pre class="brush: java; gutter: true">
@EnableWebSecurity
@Configuration
public class CustomWebSecurityConfigurerAdapter extends
   WebSecurityConfigurerAdapter {
  @Override
  protected void registerAuthentication(AuthenticationManagerBuilder auth) {
    auth
      .inMemoryAuthentication()
        .withUser("user")  // #1
          .password("password")
          .roles("USER")
          .and()
        .withUser("admin") // #2
          .password("password")
          .roles("ADMIN","USER");
  }
 
  @Override
  public void configure(WebSecurity web) throws Exception {
    web
      .ignoring()
        .antMatchers("/resources/**"); // #3
  }
 
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http
      .authorizeUrls()
        .antMatchers("/signup","/about").permitAll() // #4
        .antMatchers("/admin/**").hasRole("ADMIN") // #6
        .anyRequest().authenticated() // #7
        .and()
    .formLogin()  // #8
        .loginUrl("/login") // #9
        .permitAll(); // #5
  }
}
</pre>

我们也需要更新`[AbstractAnnotationConfigDispatcherServletInitializer](http://blog.springsource.org/2013/07/03/spring-security-java-config-preview-web-security/#aacdsi)`，这样`CustomWebSecurityConfigurerAdapter`可以实现以下功能:

- #1 可以在内存中的验证(memory authentication)叫作"user"的用户
- #2 可以在内存中的验证(memory authentication)叫作"admin"的管理员用户
- #3 忽略任何以"/resources/"开头的请求，这和在XML配置[http@security=none](http://static.springsource.org/spring-security/site/docs/3.1.x/reference/appendix-namespace.html#nsa-http-security)的效果一样
- #4 任何人(包括没有经过验证的)都可以访问"/signup"和"/about"
- #5 任何人(包括没有经过验证的)都可以访问"/login"和"/login?error"。permitAll()是指用户可以访问formLogin()相关的任何URL。
- #6 "/admin/"开头的URL必须要是管理员用户，譬如"admin"用户
- #7 所有其他的URL都需要用户进行验证
- #8 使用Java配置默认值设置了基于表单的验证。使用POST提交到"/login"时，需要用"username"和"password"进行验证。
- #9 注明了登陆页面，意味着用GET访问"/login"时，显示登陆页面

下面的XML配置和上面的Java配置类似：

<http security="none" pattern="/resources/**"/>
<http use-expressions="true">
  <intercept-url pattern="/logout" access="permitAll"/>
  <intercept-url pattern="/login" access="permitAll"/>
  <intercept-url pattern="/signup" access="permitAll"/>
  <intercept-url pattern="/about" access="permitAll"/>
  <intercept-url pattern="/**" access="hasRole('ROLE_USER')"/>
  <logout
      logout-success-url="/login?logout"
      logout-url="/logout"
  />
  <form-login
      authentication-failure-url="/login?error"
      login-page="/login"
      login-processing-url="/login"
      password-parameter="password"
      username-parameter="username"
  />
</http>
<authentication-manager>
  <authentication-provider>
    <user-service>
      <user name="user"
          password="password"
          authorities="ROLE_USER"/>
      <user name="admin"
          password="password"
          authorities="ROLE_USER,ROLE_ADMIN"/>
    </user-service>
  </authentication-provider>
</authentication-manager>

### Java配置和XML命名空间的相同之处

在看过了更复杂的例子之后，你可能已经找到了一些XML命名空间和Java配置的相似之处。我在这里说明几条有用的信息：

- HttpSecurity和http命名空间类似。它可以对于某一部分请求进行特别配置。要看个完整的配置实例，详见[SampleMultiHttpSecurityConfig](https://github.com/SpringSource/spring-security-javaconfig/blob/master/samples-web.md#sample-multi-http-web-configuration)
- WebSecurity和Security的命名空间的元素很类似，后者是针对web的，不需要父节点(security=none, debug等等)。可以对整个web security进行配置。
- WebSecurityConfigurerAdapter方便我们定制WebSecurity和HttpSecurity。我们可以对WebSecurityConfigurerAdapter进行多次扩展，以实现不同的http行为。详细的实例参见[SampleMultiHttpSecurityConfig](https://github.com/SpringSource/spring-security-javaconfig/blob/master/samples-web.md#sample-multi-http-web-configuration)
- 我们以上的Java配置代码作了代码格式化，所以易于阅读。“and()”类似于XML中结束一个元素的结束符。

### Java配置和XML命名空间的不同之处

你已经意识到了XML和Java配置的不同之处

- 当你在"#1"和“#2”中创建用户的时候，我们并没有设置为"ROLE_"前缀，而我们在XML设置成了“ROLE_USER”。因为这是个管理，"roles()"方法会自动添加"ROLE_"。如果你不想要"ROLE_"，你可以使用"authoritites()"方法。
- Java配置有一些不同的默认URL和参数。当要创建自定义的登陆页面的时候要将这一条牢记在心。默认的URL使我们的URL更加RESTful。另外，使用Spring Security可以帮我避免[信息泄露](https://www.owasp.org/index.php/Information_Leak_(information_disclosure))。例如：
  - GET访问`/login`登陆页面，而不是访问`/spring_security_login`
  - POST访问``/login来验证用户，而不是访问`/j_spring_security_check`
  - 用户名默认为parameter，而不是j_username
  - 密码默认是password，而不是j_password
- Java配置可以更容易将多个请求映射到同样的角色上。#4就将两个URL作了映射，以便所有人都可以访问
- Java移除了多余的代码。例如，在XML中我们不得不在`form-login`和`intercept-url`中重复两次"/login"，而在Java配置中，我们靠#5就轻易做到了让用户都能访问到和formLogin()相关的URL。
- #6映射HTTP请求的时候，我们使用了“hasRole()”方法，我们也没有添加"ROLE_"前缀，而在XML中我们则添加了。这也是我们应该知道的惯例:"hasRole()"会自动添加"ROLE_"前缀。如果你不想要“ROLE_”前缀，你可以使用"access()"方法。

### 更多的示例

我们还提供了更多的示例，你应该想跃跃欲试了吧：

- HttpSecurity的[Javadoc](http://static.springsource.org/spring-security/site/docs/3.2.x/apidocs/org/springframework/security/config/annotation/web/builders/HttpSecurity.html)中有一些例子。查看方法的Javadoc有助你学习到不少东西，如[openid](http://static.springsource.org/spring-security/site/docs/3.2.x/apidocs/org/springframework/security/config/annotation/web/builders/HttpSecurity.html#openidLogin()), [remember me](http://static.springsource.org/spring-security/site/docs/3.2.x/apidocs/org/springframework/security/config/annotation/web/builders/HttpSecurity.html#rememberMe())等。
- [入门教程](https://github.com/SpringSource/spring-security-javaconfig/blob/master/quickstart.md)
- [基于Web的示例](https://github.com/SpringSource/spring-security-javaconfig/blob/master/samples-web.md)
- [完整的Web应用](https://github.com/SpringSource/spring-security-javaconfig/tree/master/samples)

### 从XML命名空间到Java配置

如果你觉得从XML转变成Java配置有一定困难，你可以先看看这些测试。这些测试中，XML元素的名称以"Namespace"开头，中间是XML元素的名称，然后以"Tests"结尾。例如，你想学习如何将http元素转换成Java配置，你可以看看[NamespaceHttpTests](https://github.com/SpringSource/spring-security-javaconfig/blob/master/spring-security-javaconfig/src/test/groovy/org/springframework/security/config/annotation/web/builders/NamespaceHttpTests.groovy)；你如果想学习如何将remember-me命名空间转换成Java配置，参见[NamespaceRememberMeTests](https://github.com/SpringSource/spring-security-javaconfig/blob/master/spring-security-javaconfig/src/test/groovy/org/springframework/security/config/annotation/web/configurers/NamespaceRememberMeTests.groovy)

### 欢迎反馈

如果你发现了bug，或者觉得有什么地方值得改进，请你不要犹豫，给我们留言！我们希望听到你的想法，以便在大部分人获得代码之前，我们便确保代码的正确性。很早的尝试新功能是一种简单有效的回馈社区的方法，这样做的好处就是能帮助你获得你希望获得的功能。

请到“Java Config”目录下的[Spring Security JIRA](https://jira.springsource.org/browse/SEC)记录下任何问题。记录了一个JIRA之后，我们希望（当然并不是必须的）你在pull request中提交你的代码。你可以在[贡献者指引](https://github.com/SpringSource/spring-security/blob/master/CONTRIBUTING.md)中阅读更详细的步骤。如果你有任何不清楚的，请使用[Spring Security论坛](http://forum.springsource.org/forumdisplay.php?33-Security)或者[Stack Overflow](http://stackoverflow.com/questions/tagged/spring-security)，并使用"spring-security"标签(我会一直查看这个标签)。如果你针对这个博客有任何意见，也请留言。使用合适的工具对每个人来说都会带来便利。

### 结论

你可能已经对基于web的security的Java配置已经有了一定的认识了。下一篇中，我们将会带你来看下如何用Java配置来配置基于method的security。
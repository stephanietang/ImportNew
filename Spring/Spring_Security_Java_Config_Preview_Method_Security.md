## SPRING SECURITY JAVA配置：Method Security

这是这个系列的第三篇。在[第一篇](http://www.importnew.com/5520.html)中，我介绍了Spring Security Java配置，也概括的介绍了一下这个项目的方方面面。[第二篇](http://www.importnew.com/5641.html)中，我们看了几个基于web的security实例配置。

这篇文章中，我将会介绍如何使用Java配置来创建基于方法的security。同前一篇文章一样，我们循序渐进，开始是个最简单的例子，之后再做些更高级的改动。

#### MethodSecurityService

首先，我们假设已经有了一个MethodSecurityService的service：

<pre class="brush: java; gutter: true">
public interface MethodSecurityService {
    @PreAuthorize("hasRole('ROLE_USER')")
    String requiresUserRole();
}
</pre>

这个implementation很简单，目的是我们将重点放在Spring Security上，而不是service上。

<pre class="brush: java; gutter: true">
public class MethodSecurityServiceImpl implements
      MethodSecurityService {
 
    public String requiresUserRole() {
        return "You have ROLE_USER";
    }
}
</pre>

### Hello Method Security

使用@EnableGlobalMethodSecurity，我们很容易让我们的方法需要进行验证。我们要注意，methodSecurityService并不是Security configuration的一部分，但我们仍旧创建MethodSecurityService实例，然后使得它能应用Security。

<pre class="brush: java; gutter: true">
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled=true)
public class HelloMethodSecurityConfig {
  @Bean
  public MethodSecurityService methodSecurityService() {
    return new MethodSecurityServiceImpl()
  }
 
  @Bean
  public AuthenticationManager authenticationManager() throws Exception {
    return new AuthenticationManagerBuilder()
        .inMemoryAuthentication()
          .withUser("user").password("password").roles("USER").and()
          .withUser("admin").password("password").roles("USER", "ADMIN").and()
        .build();
    }
}
</pre>

以上的配置和下面的XML配置类似：

<global-method-security pre-post-annotations="enabled"/>
<authentication-manager>
  <authentication-provider>
    <user-service>
      <user name="user" password="password" authorities="ROLE_USER"/>
    </user-service>
  </authentication-provider>
</authentication-manager>
<beans:bean id="methodSecuriytService" class="MethodSecurityServiceImpl"/>

我们的配置中，methodSecurityService中的requiresUserRole()需要当前的用户已经使用“ROLE\_USER”进行了验证了。如果当前的用户不是“ROLE\_USER”角色，或者没有进行验证，将会抛出AccessDeniedException异常。

### 定制Method Security

@EnableWebSecurity有许多属性，如果你希望定制更复杂的功能，你需要继承GlobalMethodSecurityConfiguration。下面的例子中，我们定制了一个PermissionEvaluator：

<pre class="brush: java; gutter: true">
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled=true)
public class CustomPermissionEvaluatorWebSecurityConfig extends GlobalMethodSecurityConfiguration {
  @Bean
  public MethodSecurityService methodSecurityService() {
    return new MethodSecurityServiceImpl()
  }
 
  @Override
  protected MethodSecurityExpressionHandler expressionHandler() {
    DefaultMethodSecurityExpressionHandler expressionHandler = new DefaultMethodSecurityExpressionHandler();
    expressionHandler.setPermissionEvaluator(new CustomPermissionEvaluator());
    return expressionHandler;
  }
 
  @Override
  protected void registerAuthentication(AuthenticationManagerBuilder auth)
     throws Exception {
    auth
      .inMemoryAuthentication()
        .withUser("user").password("password").roles("USER");
  }
}
</pre>

以上的配置和下面的XML配置类似：

<global-method-security pre-post-annotations="enabled">
  <expression-handler ref="expressionHandler"/>
</global-method-security>
<authentication-manager>
  <authentication-provider>
    <user-service>
      <user name="user" password="password" authorities="ROLE_USER"/>
    </user-service>
  </authentication-provider>
</authentication-manager>
<beans:bean id="methodSecuriytService" class="MethodSecurityServiceImpl"/>
<beans:bean id="expressionHandler" class="CustomExpressionHandler"/>

### 更多的例子

上面的例子中，我们演示了使用Java配置从方法的层次上为应用进行验证的例子，你可以从spring-security-javaconfig项目的github页面找到更多例子。

- [基于方法的security实例](https://github.com/SpringSource/spring-security-javaconfig/blob/master/samples-method.md)
- [完整的Web应用 (也包含有许多基于方法的security实例)](https://github.com/SpringSource/spring-security-javaconfig/tree/master/samples)

### 欢迎反馈

如果你发现了bug，或者觉得有什么地方值得改进，请你不要犹豫，给我们留言！我们希望听到你的想法，以便在大部分人获得代码之前，我们便确保代码的正确性。很早的尝试新功能是一种简单有效的回馈社区的方法，这样做的好处就是能帮助你获得你希望获得的功能。

请到“Java Config”目录下的[Spring Security JIRA](https://jira.springsource.org/browse/SEC)记录下任何问题。记录了一个JIRA之后，我们希望（当然并不是必须的）你在pull request中提交你的代码。你可以在[贡献者指引](https://github.com/SpringSource/spring-security/blob/master/CONTRIBUTING.md)中阅读更详细的步骤。如果你有任何不清楚的，请使用[Spring Security论坛](http://forum.springsource.org/forumdisplay.php?33-Security)或者[Stack Overflow](http://stackoverflow.com/questions/tagged/spring-security)，并使用"spring-security"标签(我会一直查看这个标签)。如果你针对这个博客有任何意见，也请留言。使用合适的工具对每个人来说都会带来便利。

### 总结

你可能已经对如何使用Java配置对基于方法的security进行配置有了初步的了解。下一篇，我们将会带你来看一个OAuth的例子，来解释Spring Security是如何和其他模块进行扩展的。
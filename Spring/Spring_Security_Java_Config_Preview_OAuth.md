## SPRING SECURITY JAVA CONFIG PREVIEW: OAUTH

这是这个系列的第四篇。我们将会用一个OAuth的例子来看看Java配置是如何扩展的。

### 概念验证（Proof of concept，简称POC）

> 译者注：概念验证(Proof of concept, 简称POC)是对某些想法的较短的不完整的实现，以证明其可行性，示范其原理，其目的是为了验证一些概念或理论。

Spring Security的Java配置和基本的配置合作工作的很好，但这仅仅是一个POC。我们尚不能保证XML命名空间中的所有功能都可以对Java配置进行支持。当然保证Spring Security的Java配置与Spring extension配合良好是非常重要的。但我们已经等不及完成所有的细节，就把Spring Security Java config发布出来了。将来我们会完善OAuth Java配置，但现在这里仅仅是一个POC，而不是完全的解决方法。

### HelloOAuth2ServerConfiguration

Spring Security OAuth Java配置目前支持基本的OAuth 2服务器端配置。最简单的设置如下：

<pre  class="brush: java; gutter: true" >
    @Configuration
    @EnableWebSecurity
    public class HelloOAuth2ServerConfiguration
       extends OAuth2ServerConfigurerAdapter {
        private static final String RESOURCE_ID = "photos";
     
        @Override
        protected void registerAuthentication(AuthenticationManagerBuilder auth)
                throws Exception {
            auth
                .apply(new InMemoryClientDetailsServiceConfigurer())
                    .withClient("my-client")
                        .resourceIds(RESOURCE_ID)
                        .authorizedGrantTypes("authorization_code","implicit")
                        .authorities("ROLE_USER")
                        .scopes("read","write")
                        .secret("secret");
        }
     
        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http
                .authorizeUrls()
                    .anyRequest().authenticated()
                    .and()
                .apply(new OAuth2ServerConfigurer())
                    .resourceId(RESOURCE_ID);
        }
    }
</pre>

现在我们可以处理OAuth请求了。但是仍然有很多限制：

- 如果一个用户获得了OAuth服务器的请求许可，我们仍然需要一个Controller来处理"/oauth/confirm_access"。
- 只有OAuth客户端可以验证用户，所以对于用户来说，没办法确认访问。
- 授权(Authorization)映射到了验证路径(authentication)，这显然是个错误的例子。

### SecurityConfigurer

我们先不要管这些问题，我们来看看发生了什么。OAuth2ServerConfigurer是SecurityConfigurer的实例。实现SecurityConfigurer时最好是继承SecurityConfigurerAdapter，因为它是SecurityConfigurerAdapter最基本的实现。

要让HttpSecurity能够接受SecurityConfigurer，我们可以添加插件（譬如OAuth），这用一种更复杂的方法更新了HttpSecurity对象。事实上，HttpSecurity.formLogin()方法也是用SecurityConfigurerAdapter来实现的。不同之处在于，HttpSecurity.formLogin()是Spring Security的核心模块，所以有更简便的方法来实现pply(SecurityConfigurerAdapter)。

我们不仅可以让SecurityConfigurer应用到HttpSecurity对象（译者注：configure(HttpSecurity http)方法），也可以让SecurityConfigurer应用到AuthenticationManagerBuilder（译者注：registerAuthentication(AuthenticationManagerBuilder auth)方法）。正是InMemoryClientDetailsServiceConfigurer对象起了作用。这意味着认证原理也很容易被扩展。

这对你来说意味着什么？假设你有个更复杂的Security配置，并且想分享给全公司。你可以选择让工程师们复制粘贴这些配置。另外你还可以创建你自己的SecurityConfigurerAdapter实现，让开发者们专注在你们公司自己的DSL。

### 更多OAuth实例

我们这里演示的例子非常的简单，而且这也不是一个真实的例子。在我们觉得已经完善OAuth支持之前，我们不会在博客上再更新相关内容的。下面还有些更多的OAuth实例，你可以试一试：

- Spring Security JavaConfig's [sparklr application](https://github.com/SpringSource/spring-security-javaconfig/blob/master/samples/oauth2-sparklr/src/main/java/org/springframework/security/oauth/examples/sparklr/config/OAuth2ServerConfig.java)
- [The Spring Rest Stack's](https://github.com/joshlong/the-spring-rest-stack) – oauth module

### 欢迎反馈

如果你发现了bug，或者觉得有什么地方值得改进，请你不要犹豫，给我们留言！我们希望听到你的想法，以便在大部分人获得代码之前，我们便确保代码的正确性。很早的尝试新功能是一种简单有效的回馈社区的方法，这样做的好处就是能帮助你获得你希望获得的功能。

请到“Java Config”目录下的[Spring Security JIRA](https://jira.springsource.org/browse/SEC)记录下任何问题。记录了一个JIRA之后，我们希望（当然并不是必须的）你在pull request中提交你的代码。你可以在[贡献者指引](https://github.com/SpringSource/spring-security/blob/master/CONTRIBUTING.md)中阅读更详细的步骤。如果你有任何不清楚的，请使用[Spring Security论坛](http://forum.springsource.org/forumdisplay.php?33-Security)或者[Stack Overflow](http://stackoverflow.com/questions/tagged/spring-security)，并使用"spring-security"标签(我会一直查看这个标签)。如果你针对这个博客有任何意见，也请留言。使用合适的工具对每个人来说都会带来便利。

### 结论

Spring Security Java配置只是用OAuth作为一个概念验证，说明在不远的将来会全面支持扩展。到那时，对OAuth的Java配置成熟后，将可以取代XML配置。我最后一篇文章将会讨论Spring Security Java配置的可读性。


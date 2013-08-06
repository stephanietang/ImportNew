## SPRING SECURITY JAVA配置：可读性

这篇文章，我会介绍如何让Spring Security Java配置的可读性更高。这篇文章详细阐述了"Spring Security Java Config Preview: Web Security"系列文章中的如下观点：

格式化我们的Java配置代码，可读性将更强。它和XML命名规则类似，"and()"等同于XML命名空间中的结束元素。

## 缩进

Spring Security的Java配置的缩进对可读性有很大的影响。我们应该更多的使用缩进。从更高的层次来看，应该看起来是这样：

<pre  class="brush: java; gutter: true" >
builder
  #1 I modify builder
      I modify #1
      I also modify #1
      I also modify #1
      and
  #2 I also modify builder
      I modify #2
      I also modify #2
      and
  #3 I also modify builder
      I modify #3
  ...
</pre>

具体的例子，请看以下的代码：

<pre  class="brush: java; gutter: true" >
http
    // #1
    .formLogin()
        // #2
        .loginUrl("/login")
        .failureUrl("/login?error")
        // #3
        .and()
    // #4
    .authorizeUrls()
        // #5
        .antMatchers("/signup","/about").permitAll()
        .antMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated();
</pre>

- #1 formLogin更改了http对象本身。formLogin相对于http进行了缩进(就像XML中<form-login>相对于<http>缩进一样)。
- #2 loginUrl和failureUrl更高了formLogin的配置。例如如果需要用户登陆的话，loginUrl决定了Spring Security跳转到哪个页面。所以loginUrl和failureUrl都是formLogin的子节点。
- #3 "and"表示我们已经配置完了父节点了（这里是formLogin），同时下一行将减少缩进。你可以将整体看作成是http由formLogin和authorizeUrls来进行配置。如果你没有其它要配置了，就不需要用“and”。
- #4 我们在这里减少的缩进是因为authorizeUrls和基于表单的登陆无关。它的作用是限制多个URL的访问。
- #5 每个antMatchers和anyRequest都更改了authorizeUrl的验证要求。所以它们都是authorizeUrls的子节点。

## IDE的格式化工具

缩进可能和IDE的格式化工具配合不好。许多IDE都能让开发者对选中的代码块禁用格式化功能。例如在STS/Eclipse中，你可以使用@formatter:off和@fommatter:on来关闭或者开启格式化功能。看下面的代码：

<pre  class="brush: java; gutter: true" >
// @formatter:off
http
    .formLogin()
        .loginUrl("/login")
        .failureUrl("/login?error")
        .and()
    .authorizeUrls()
        .antMatchers("/signup","/about").permitAll()
        .antMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated();
// @formatter:on
</pre>

要使上面的代码能够正确运作，你需要进行如下配置：

- Preferences -> Java -> Code Style -> Formatter
- 点击"Edit"
- 选中"Off/On Tags"栏
- 确保“Enable Off/On tags”被选中了
- 你也可以更改开启和关闭格式化功能的字符
- 然后点OK

## 对比XML命名空间

缩进将Java配置和XML命名空间配置联系起来。虽然有时候并不一定完全对等，但大多数情况下都是有效的。让我们来比较一下两者吧：

<pre  class="brush: java; gutter: true" >
http
    .formLogin()
        .loginPage("/login")
        .failureUrl("/login?error")
        .and()
    .authorizeUrls()
        .antMatchers("/signup","/about").permitAll()
        .antMatchers("/admin/**").hasRole("ADMIN")
        .anyRequest().authenticated();
</pre>

XML配置虽然不对等，但很类似，请看下面的XML配置代码。我们注意到，两者之间的差别主要是因为Java配置和XML配置的默认值不同。

<http use-expressions="true">
  <form-login
      login-page="/login"
      authentication-failure-url="/login?error"
  /> <!-- similar to and() -->
  <intercept-url pattern="/signup" access="permitAll"/>
  <intercept-url pattern="/about" access="permitAll"/>
  <intercept-url pattern="/admin/**" access="hasRole('ROLE_ADMIN')"/>
</http>

我们注意到http和<http>非常相似。唯一的差别是Java配置使用“authorizeUrls”来表示use-expressions="true"。而formLogin和<form-login>也很类似，formLogin的每一个子节点都是XML的<form-login>的属性。这种相似性也是非常合理的，因为XML也更改了XML元素，所以子节点更改了母节点。

formLogin下的and等同于XML的结束符。
authorizeUrls的子节点等同于<intercept-urls>。

## 总结

你应该已经知道如何缩进Spring Security的Java配置代码了。缩进将会帮助你写出可读性更高的代码，也更容易从XML配置转换成Java代码，反之亦然。
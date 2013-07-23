SPRING SECURITY JAVA CONFIG PREVIEW: READABILITY

In this post, I will discuss how to make your Spring Security Java configuration more readable. The post is intended to elaborate on a point from Spring Security Java Config Preview: Web Security where I stated:

By formatting our Java configuration code it is much easier to read. It can be read similar to the XML namespace equivalent where "and()" represents optionally closing an XML element.

Indentation

The indentation of Spring Security's Java configuration really impacts its readability. In general, indentation like a bullet list should be preferred. At a high level it looks like this:
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
For a more concrete example, take a look at the following code:
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
#1 formLogin updates the http object itself. The indentation of formLogin is incremented from that of http (much like they way the <form-login> is indented from <http>)
#2 loginUrl and failureUrl update the formLogin configuration. For example, loginUrl determines where Spring Security will redirect if log in is required. For this reason, each is a child of formLogin.
#3 and means we are done configuring the parent (in this case formLogin). This also implies that the next line will decrease indentation by one. When looking at the configuration you can read it as http is configured with formLogin and authorizeUrls. If we had nothing else to configure, the and is not necessary.
#4 We decrease the indentation with authorizeUrls since it is not related to form based log in. Instead, its intent is to restrict access to various URLs.
#5 each antMatchers and anyRequest modifies the authorization requirements for authorizeUrls. This is why each is a child of authorizeUrls
IDE Formatters
The indentation may cause problems with code formatters. Many IDE's will allow you to disable formatting for select blocks of code with comments. For example, in STS/Eclipse you can use the comments of @formatter:off and @formatter:on to turn off and on code formatting. An example is shown below:

<pre class="brush: java; gutter: true">
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

For this feature to work, make sure you have it enabled:
Navigate to Preferences -> Java -> Code Style -> Formatter
Click the Edit button
Select the Off/On Tags tab
Ensure Enable Off/On tags is selected
You can optionally change the strings used for disabling and enabling formatting here too.
Click OK
Comparison to XML Namespace
Our indentation also helps us relate the Java Configuration to the XML namespace configuration. This is not always true, but it does help. Let's compare our configuration to the relevant XML configuration below.

<pre class="brush: java; gutter: true">
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
The relevant, but not equivalent, XML configuration can be seen below. Note that the differences between how Spring Security will behave between these configurations is due to the different default values between Java Configuration and XML configuration.


<http use-expressions="true">
  <form-login
      login-page="/login"
      authentication-failure-url="/login?error"
  /> <!-- similar to and() -->
  <intercept-url pattern="/signup" access="permitAll"/>
  <intercept-url pattern="/about" access="permitAll"/>
  <intercept-url pattern="/**" access="hasRole('ROLE_USER')"/>
</http>

The first thing to notice is that the http and <http> are quite similar. One difference is that Java Configuration uses authorizeUrls to specify use-expressions="true"
formLogin and <form-login> are quite similar. Each child of formLogin is an XML attribute of <form-login>. Based upon our explanation of indentation, the similarities are logical since XML attributes modify XML elements.
The and under formLogin is very similar to ending an XML element.
Each child of authorizeUrls is similar to each <intercept-urls>, except that Java Configuration specifies requires-channel differently which helps reduce configuration in many circumstances.

总结

You should now know how to consistently indent your Spring Security Java Configuration. By doing so your code will be more readable and be easier to translate to and from the XML configuration equivalents.
# CSRF漏洞详解，一文看懂CSRF_aFa攻防实验室的博客-CSDN博客
**0x00：CSRF 简述**
----------------

[CSRF](https://so.csdn.net/so/search?q=CSRF&spm=1001.2101.3001.7020)（Cross Site Request Forgery，跨站请求伪造），字面理解意思就是在别的站点伪造了一个请求。专业术语来说就是在受害者访问一个网站时，其 Cookie 还没有过期的情况下，攻击者伪造一个链接地址发送受害者并欺骗让其点击，从而形成 CSRF 攻击。

0x01：CSRF 案例
------------

受害者 (A) 登录了某个银行给朋友 (B) 转账，其转账操作发送的请求 URL 如下：

```null
https:
```

account 代表受害者，money 是要转账的金额，touser 是被转入的账户。发送这个链接请求后，A 给 B 转账的操作完成。这时攻击者（C）伪造了一个链接，如下：

```null
https:
```

这个链接的请求是 A 用户给 C 用户转账一万元。当 A 没有登录不存在 Cookie 信息时，此链接是无法执行的。这时攻击者通过一系列手段让 A 执行此链接，当 A 登录银行没有退出的时候，点击此链接便会执行成功。

0x02：代码示例
---------

当程序对于类似此敏感信息操作提交时，没有进行相应的防护，便会产生 CSRF 攻击，例如一下代码：

```null
<form method="GET" action="/transferFunds">    转账金额：<input type="text" name="money">    转入账户：<input type="text" name="touser">    <input type="submit" name="action" value="提交">
```

0x03：如何测试
---------

在渗透测试中，可以先看下网页源代码对于敏感信息提交有无防护措施，初步判断是否存在 CSRF，随后通过抓包确定提交的完整 URL 链接，并伪造另一个链接进行测试。  
在代码审计中，可以先查看网页源代码，是否有防护措施。随后可查看 WEB 应用程序的配置文件中是否有相应的验证措施，最后查看后台的处理逻辑，对于发送过来的请求是否有过滤等措施。

0x04：防护方法
---------

1，二次验证，进行重要敏感操作时，要求用户进行二次验证。

2，验证码，进行重要敏感操作时，加入验证码。

3，验证 HTTP 的 Referer 字段。

4，请求地址中添加 Token 并验证。

5，HTTP 头中自定义属性并验证。

0x05：防护代码
---------

1，对于二次验证，可添加 JS，提交请求后询问客户是否提交，而不是直接发送请求给后台。

```null
if(confirm('确认进行转账操作？')){<form method="GET" action="/transferFunds">    转账金额：<input type="text" name="money">    转入账户：<input type="text" name="touser"><input type="submit" name="action" value="提交">
```

2，对于验证码，进行转账时，可输入图形验证码，也可以添加手机接收验证码等功能。

```null
window.onload=function createCode(){  var checkCode = document.getElementById("code");  var random = new Array(0,1,2,3,4,5,6,7,8,9,'A','B','C','D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R', 'S','T','U','V','W','X','Y','Z'); for(var i = 0; i < codeLength; i++) {var index = Math.floor(Math.random()*36);var inputCode = document.getElementById("yzm").value.toUpperCase();if(inputCode.length <= 0) {     }else if(inputCode != code ) {
```

  
以上代码，当验证码为空时或错误时则无法提交请求。

3，对于验证 HTTP Referer 字段，请求到后台时，判断下请求是否来自自己的站点，如果不是 Referer 的值不是以自己域名开头，则会请求失败。

```null
String referer = request.getHeader("Referer");if((referer!=null) && (referer.trim().startsWith("xxxx.com"))){    chain.doFilter(request,response);request.getRequestDispatcher("error.jsp").forward(request,response);
```

以上代码，使用 Java 的过滤器 Filter 来拦截请求，当获取请求的 Referer 的值不为空时并且是以自己站点的域名开头时，则放行。否则跳转到 error 页面。

4，对于请求地址中添加 Token 验证，也是用的最多的一个方法，在表单中添加一个 hidden 隐藏字段，发送请求时一起发送，并在服务器验证 Token 的值。Token 的值越复杂，则安全性越高。

```null
HttpServletRequest req = (HttpServletRequest)request;HttpSession s = req.getSession();String sToken = (String)s.getAttribute("token");    sToken = generateToken();    s.setAttribute("token",sToken);    chain.doFilter(request,response);String token = req.getParameter("token");if(sToken != null && sToken.equals(token)){        chain.doFilter(request,response);        request.getRequestDispatcher("error.jsp").forward(request,response);
```

对于代码中的 generateToken 生成 token 的方法，可以使用 Java 的 UUID，代码如下：

```null
    String uuid = UUID.randomUUID().toString().replaceAll("-", "")
```

5，对于 HTTP 头部自定义属性验证，和 token 机制类似。只不过是把 token 从表单放到了请求的头重，如下代码：

```null
dojo.xhr = function(method,args,hasBody) {         args.headers = args.header || {};         tokenValue = '<%=request.getSession(false).getAttribute("token")%>'; var token = dojo.getObject("tokenValue");     args.headers["token"] = (token) ? token : "  ";     return plainXhr(method,args,hasBody); 
```

dojo.xhr 是用 JS 写的一个工具包，重写其中的方法，把从 session 中获取的 token 值放到新添加的头部字段中，然后发送给后台。

0x06：CSRF 总结
------------

对于 CSRF 其危害性比较大，不易防护，建议在开发过程中结合以上的多条防护措施进行防护，不建议只用某一点或某一条，多层防护更有利于系统的安全。

* * *

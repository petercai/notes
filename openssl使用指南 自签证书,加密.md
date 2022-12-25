# openssl使用指南:自签证书,加密
*   生成自签证书：[https://goharbor.io/docs/2.3.0/install-config/configure-https/](https://goharbor.io/docs/2.3.0/install-config/configure-https/)
*   配置主机信任证书：[https://goharbor.io/docs/2.3.0/install-config/troubleshoot-installation/#https](https://goharbor.io/docs/2.3.0/install-config/troubleshoot-installation/#https)

| 类别 | 值 |
| --- | --- |
| 对称加解密 | openssl \[enc -\]des -k 123456 -in a.txt -out a.txt.crpy |
| 单向加密(文件特征码) | openssl dgst a.txt |
| 生成加密密码 | openssl passwd 123456 |
| 生成随机数 | openssl rand -hex 123456 |
| 生成私钥 | (umask 077 ; openssl genrsa -out id\_rsa; openssl rsa -in id\_rsa -pubout >id_rsa.pub) |

```
 ``wang@wang-pc:~$ openssl help
openssl:Error: 'help' is an invalid command.

Standard commands
asn1parse         ca                ciphers           cms               
crl               crl2pkcs7         dgst              dh                
dhparam           dsa               dsaparam          ec                
ecparam           enc               engine            errstr            
gendh             gendsa            genpkey           genrsa            
nseq              ocsp              passwd            pkcs12            
pkcs7             pkcs8             pkey              pkeyparam         
pkeyutl           prime             rand              req               
rsa               rsautl            s_client          s_server          
s_time            sess_id           smime             speed             
spkac             srp               ts                verify            
version           x509              

Message Digest commands (see the `dgst' command for more details)
md4               md5               rmd160            sha               
sha1              

Cipher commands (see the `enc' command for more details)
aes-128-cbc       aes-128-ecb       aes-192-cbc       aes-192-ecb       
aes-256-cbc       aes-256-ecb       base64            bf                
bf-cbc            bf-cfb            bf-ecb            bf-ofb            
camellia-128-cbc  camellia-128-ecb  camellia-192-cbc  camellia-192-ecb  
camellia-256-cbc  camellia-256-ecb  cast              cast-cbc          
cast5-cbc         cast5-cfb         cast5-ecb         cast5-ofb         
des               des-cbc           des-cfb           des-ecb           
des-ede           des-ede-cbc       des-ede-cfb       des-ede-ofb       
des-ede3          des-ede3-cbc      des-ede3-cfb      des-ede3-ofb      
des-ofb           des3              desx              rc2               
rc2-40-cbc        rc2-64-cbc        rc2-cbc           rc2-cfb           
rc2-ecb           rc2-ofb           rc4               rc4-40            
seed              seed-cbc          seed-cfb          seed-ecb          
seed-ofb          


wang@wang-pc:~$ whatis md5
md5 (1ssl)           - message digests
wang@wang-pc:~$ whatis ca
ca (1ssl)            - sample minimal CA application
wang@wang-pc:~$ whatis enc
enc (1ssl)           - symmetric cipher routines

wang@wang-pc:~$ man ca 
CA(1SSL)                                       OpenSSL                                      CA(1SSL)
NAME
       ca - sample minimal CA application
SYNOPSIS
       openssl ca [-days arg] [-key arg] [-cert file] [-in file] [-out file]`` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45
*   46
*   47
*   48
*   49
*   50
*   51
*   52


```

a， 对称加解密文件 : enc
----------------

```
`openssl enc ( man enc : 默认选项: -e, -salt, 甚至enc也可以省略)
openssl enc -ciphername [-in filename] [-out filename] [-e] [-d]  [-a/-base64] [-k password] [-kfile filename] [-salt]

[root@c7 ssl]
a.txt
[root@c7 ssl]
a b c 123 xxyz 

[root@c7 ssl]
[root@c7 ssl]
a.txt  a.txt.des-cbc
[root@c7 ssl]
Salted__�6T�����C0���C��y��9)PN[�� 
[root@c7 ssl]
[root@c7 ssl]
a b c 123 xxyz 

[root@c7 ssl]
[root@c7 ssl]
[root@c7 ssl]
a b c 123 xxyz` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23


```

b， 单向加密(文件特征码): dgst (默认选项： -md5, -hex)
---------------------------------------

```
 `openssl dgst [-sha|-sha1|-mdc2|-ripemd160|-sha224|-sha256|-sha384|-sha512|-md2|-md4|-md5|-dss1]
      [-hex] [-binary]  [-out filename] [-sign filename] [-keyform arg] [-verify filename]  [-signature filename] 
      [file...]

[root@c7 md5]
a b c 123 xxyz
[root@c7 md5]
8c382fd658224880cbb86c48a27ec215  a.txt
[root@c7 md5]
[root@c7 md5]
a.txt: OK

[root@c7 md5]
[root@c7 md5]
a.txt: FAILED
md5sum: WARNING: 1 computed checksum did NOT match

[root@c7 md5]
MD5(a.txt)= 2c4b50b6339567df47f67ba87e34cc99
[root@c7 md5]
[root@c7 md5]
a.txt: OK
[root@c7 md5]
[root@c7 md5]
a.txt: FAILED
md5sum: WARNING: 1 computed checksum did NOT match

[root@c7 sha]
[root@c7 sha]
a.txt: OK
[root@c7 sha]
[root@c7 sha]
[root@c7 sha]
a.txt: FAILED
sha512sum: WARNING: 1 computed checksum did NOT match` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37


```

c， 生成加密密码: passwd
-----------------

```
`[root@c7 ssl2]
sslpasswd (1ssl)     - compute password hashes
passwd (1)           - update user's authentication tokens
[root@c7 ssl2]
openssl passwd [-crypt] [-1] [-apr1] [-salt string]
		[-in file]  {password}

wang@wang-pc:~$ openssl passwd -1 -salt haohao 123456
$1$haohao$5REY/e6ZWowqez3MncIkV1
wang@wang-pc:~$ echo "123456" >a.txt
wang@wang-pc:~$ openssl passwd -1 -salt haohao -in a.txt 
$1$haohao$5REY/e6ZWowqez3MncIkV1` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16


```

d， 生成随机数: rand
--------------

```
`wang@wang-pc:~$ whatis rand
rand (3)             - pseudo-random number generator
rand (1ssl)          - generate pseudo-random bytes
wang@wang-pc:~$ man rand
openssl rand [-out file] [-base64] [-hex] num

[root@c7 ssl2]
03e0ad
[root@c7 ssl2]
09df1298
[root@c7 ssl2]
92dae01fb3b3b8b1c1b7
[root@c7 ssl2]
1Gg/IH1apoLp8A==
[root@c7 ssl2]
WwTKHmmxAwAIq/5hjGnye2ag24Q=` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18


```

![](https://img-blog.csdnimg.cn/2021022414284993.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2V5ZW9mZWFnbGU=,size_16,color_FFFFFF,t_70)

| 角色 | ip | 操作 | 持有的证书信息 |
| --- | --- | --- | --- |
| CA服务器 | 192.168.56.109, Centos6 | 自签证书，签发别人请求的证书 | subject= /C=CN/ST=beijing/L=beijing/O=my.com/OU=devops/CN=ca.my.com |
| web服务器 | 192.168.56.106, Centos7 | 给CA发送签证请求 | subject= /C=CN/ST=beijing/O=my.com/OU=devops/CN=www.my.com |

| CA操作条目 | 具体方法 |
| --- | --- |
| 生成私钥 | openssl genrsa -out cakey.pem 2048 |
| ####从私钥抽取公钥 | openssl rsa -in cakey.pem -pubout |
| ca自签证书 | openssl req -new -x509 -key cakey.pem -out cacert.pem -days 365 |
| ####从证书抽取公钥 | openssl x509 -in cacert.pem -noout -pubkey |

| 客户端操作条目 | 具体方法 |
| --- | --- |
| 生成私钥 | openssl genrsa -out httpkey.pem 2048 |
| 生成签证请求 | openssl req -new -key httpkey.pem -out httpcsr.pem -days 365 |

快速生成证书：在当前目录
------------

```
 `mkdir ssl; cd ssl
(umask 077; openssl genrsa -out cakey.pem 4096)

openssl req -new -x509 -key cakey.pem -out cacert.pem -days 36500 \
  -subj "/C=CN/ST=bj/L=bj/O=my.com/OU=devops/CN=my.cone/emailAddress=''"

touch /etc/pki/CA/{serial,index.txt}
[ ! -f /etc/pki/CA/index.txt ] && echo 01 > /etc/pki/CA/serial 

mkdir httpd; cd httpd
(umask 077; openssl genrsa -out httpd.key 2048)

openssl req -new -key httpd.key -out httpd.csr -days 36500 \
   -subj "/C=CN/ST=bj/L=bj/O=my.com/OU=devops/CN=my.cone/emailAddress=''" 


openssl ca -in httpd.csr -out httpd.crt -days 36500   -cert ../cacert.pem   -keyfile ../cakey.pem` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23


```

a, 第一步：搭建内网CA机构
---------------

```
`[root@c6 CA]
/etc/pki/CA
/etc/pki/CA/certs
/etc/pki/CA/private
/etc/pki/tls
...
/etc/pki/tls/openssl.cnf 
[root@c6 CA]

[ ca ]
default_ca      = CA_default            

[ CA_default ]
dir             = /etc/pki/CA           
certs           = $dir/certs            
crl_dir         = $dir/crl              
database        = $dir/index.txt        
new_certs_dir   = $dir/newcerts         
certificate     = $dir/cacert.pem       
serial          = $dir/serial           
crl             = $dir/crl.pem          
private_key     = $dir/private/cakey.pem
RANDFILE        = $dir/private/.rand    
default_days    = 365                   
default_crl_days= 30                    
default_md      = default               
preserve        = no                    

1, 私钥
[root@c6 ssl]
Generating RSA private key, 4096 bit long modulus
............................................................................................................................................................++
................++
e is 65537 (0x10001)
[root@c6 ssl]
cakey.pem
[root@c6 ssl]
total 4
-rw------- 1 root root 3243 1月   7 15:20 cakey.pem

2, 自签证书 , 免交换填充参数：-subj "/C=CN/ST=bj/L=bj/O=my.com/OU=devops/CN=my.cone/emailAddress=''" 
[root@c6 ssl]
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:beijing
Locality Name (eg, city) [Default City]:beijing
Organization Name (eg, company) [Default Company Ltd]:my.com
Organizational Unit Name (eg, section) []:devops
Common Name (eg, your name or your server's hostname) []:ca.my.com
Email Address []:
[root@c6 ssl]
cacert.pem  certs/  crl/  newcerts/  private/

[root@c6 ~]
subject= /C=CN/ST=beijing/L=beijing/O=my.com/OU=devops/CN=ca.my.com
serial=DEC3D30B63F260F0

3,创建序列号文件
[root@c6 ssl]
[root@c6 ssl]` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45
*   46
*   47
*   48
*   49
*   50
*   51
*   52
*   53
*   54
*   55
*   56
*   57
*   58
*   59
*   60
*   61
*   62
*   63
*   64
*   65
*   66
*   67
*   68
*   69
*   70
*   71
*   72
*   73
*   74


```

b, 第二步：内网web站点请求签证
------------------

```
`1, 生成请求
[root@c7 ~]
[root@c7 httpd]
conf  conf.d  conf.modules.d  http.groups  http.users  logs  modules  run
[root@c7 httpd]
[root@c7 httpd]
[root@c7 ssl]
Generating RSA private key, 2048 bit long modulus
.....................+++
.............................................+++
e is 65537 (0x10001)
[root@c7 ssl]
httpd.key
[root@c7 ssl]
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:beijing
Locality Name (eg, city) [Default City]:beijing
Organization Name (eg, company) [Default Company Ltd]:my.com
Organizational Unit Name (eg, section) []:devops
Common Name (eg, your name or your server's hostname) []:www.my.com
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:1234
An optional company name []:
[root@c7 ssl]# ls
httpd.csr  httpd.key

2, 发送请求文件
[root@c7 ssl]# scp httpd.csr 192.168.56.109:~/
The authenticity of host '192.168.56.109 (192.168.56.109)' can't be established.
RSA key fingerprint is SHA256:Tjq0JWgf+q2KLez78jN0Inoupl7xka+obehHvOhjXDI.
RSA key fingerprint is MD5:54:ff:be:3f:9e:1d:cd:72:7e:1a:96:11:7a:80:3d:42.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.56.109' (RSA) to the list of known hosts.
root@192.168.56.109's password: 
httpd.csr                                     100% 1066   551.3KB/s   00:00` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44
*   45


```

c, 第三步：CA签署证书,并发放给web站点
-----------------------

```
`[root@c6 ~]
httpd.csr 

[root@c6 ~]
Using configuration from /etc/pki/tls/openssl.cnf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: Jan  7 07:53:20 2020 GMT
            Not After : Jan  6 07:53:20 2021 GMT
        Subject:
            countryName               = CN
            stateOrProvinceName       = beijing
            organizationName          = my.com
            organizationalUnitName    = devops
            commonName                = www.my.com
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                E0:55:2F:BB:0A:FC:5B:77:D4:C2:F8:EA:3F:59:2B:15:AA:5F:A0:BB
            X509v3 Authority Key Identifier: 
                keyid:FC:CA:70:85:EB:0C:DA:EB:65:D4:72:A3:AA:17:A3:B1:77:23:52:3E

Certificate is to be certified until Jan  6 07:53:20 2021 GMT (365 days)
Sign the certificate? [y/n]:y

1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated

[root@c6 ~]
V	210106075320Z		01	unknown	/C=CN/ST=beijing/O=my.com/OU=devops/CN=www.my.com

[root@c6 ~]
root@192.168.56.106's password: 
httpd.crt                                     100% 5706     5.6KB/s   00:00` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31
*   32
*   33
*   34
*   35
*   36
*   37
*   38
*   39
*   40
*   41
*   42
*   43
*   44


```

d, 第四步：web站点查看证书
----------------

```bash
[root@c7 ~]
httpd.crt  httpd.csr  httpd.key


[root@c7 ~]
/etc/httpd/ssl
[root@c7 ssl]
serial=01
subject= /C=CN/ST=beijing/O=my.com/OU=devops/CN=www.my.com

```

a，配置私有证书
--------

```
`[root@test-c6 ~]# grep -Ev '^#|^$' /etc/httpd/conf.d/ssl.conf                
LoadModule ssl_module modules/mod_ssl.so                                     
Listen 443                                                                   
SSLPassPhraseDialog  builtin                                                 
SSLSessionCache         shmcb:/var/cache/mod_ssl/scache(512000)              
SSLSessionCacheTimeout  300                                                  
SSLMutex default                                                             
SSLRandomSeed startup file:/dev/urandom  256                                 
SSLRandomSeed connect builtin                                                
SSLCryptoDevice builtin                                                      
<VirtualHost _default_:443>                                                  
	ErrorLog logs/ssl_error_log                                                  
	TransferLog logs/ssl_access_log                                              
	LogLevel warn                                                                
	SSLEngine on                                                                 
	SSLProtocol all -SSLv2                                                       
	SSLCipherSuite DEFAULT:!EXP:!SSLv2:!DES:!IDEA:!SEED:+3DES                    
	SSLCertificateFile /etc/httpd/conf/web-crt/web/httpd.crt                     
	SSLCertificateKeyFile /etc/httpd/conf/web-crt/web/httpd.key                  
	<Files ~ "\.(cgi|shtml|phtml|php3?)$">                                       
	    SSLOptions +StdEnvVars                                                   
	</Files>                                                                     
	<Directory "/var/www/cgi-bin">                                               
	    SSLOptions +StdEnvVars                                                   
	</Directory>                                                                 
	SetEnvIf User-Agent ".*MSIE.*" \                                             
	         nokeepalive ssl-unclean-shutdown \                                  
	         downgrade-1.0 force-response-1.0                                    
	CustomLog logs/ssl_request_log \                                             
	          "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"                  
</VirtualHost>` 

![](https://csdnimg.cn/release/blogv2/dist/pc/img/newCodeMoreWhite.png)

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19
*   20
*   21
*   22
*   23
*   24
*   25
*   26
*   27
*   28
*   29
*   30
*   31


```

![](https://img-blog.csdnimg.cn/2020080416241236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2V5ZW9mZWFnbGU=,size_16,color_FFFFFF,t_70)
  
![](https://img-blog.csdnimg.cn/20200804162513163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2V5ZW9mZWFnbGU=,size_16,color_FFFFFF,t_70)

b，把http访问转发到https
-----------------

```
[root@test-c6 ~]# tail /etc/httpd/conf/httpd.conf                      
#    DocumentRoot /www/docs/dummy-host.example.com                     
#    ServerName dummy-host.example.com                                 
#    ErrorLog logs/dummy-host.example.com-error_log                    
#    CustomLog logs/dummy-host.example.com-access_log common           
#</VirtualHost>                                                        
                                                                       
#追加这三段                                                                       
RewriteEngine on                                                       
RewriteCond %{SERVER_PORT} !^443$                                      
RewriteRule ^(.*)?$ https://%{SERVER_NAME}%{REQUEST_URI} [L,R]         

```

![](https://img-blog.csdnimg.cn/2020080416403775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2V5ZW9mZWFnbGU=,size_16,color_FFFFFF,t_70)
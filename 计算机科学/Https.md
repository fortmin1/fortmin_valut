
| 后缀              | 通常表示什么                     |
| --------------- | -------------------------- |
| `.crt`          | 通常是证书                      |
| `.cer`          | 通常是证书                      |
| `.key`          | 通常是私钥                      |
| `.pem`          | 通常表示 PEM 文本编码，里面可以装很多东西    |
| `.der`          | DER 二进制编码                  |
| `.p12` / `.pfx` | PKCS#12 容器，可同时装私钥和证书       |
| `.jks`          | Java KeyStore 容器           |
| `.keystore`     | 只是常见文件名，实际可能是 JKS 或 PKCS12 |

## pem
一个 PEM 证书：
```
-----BEGIN CERTIFICATE-----
MIIF...
...
-----END CERTIFICATE-----
```
一个 PEM 私钥：
```
-----BEGIN PRIVATE KEY-----
MIIE...
...
-----END PRIVATE KEY-----
```
RSA 私钥也可能是：
```
-----BEGIN RSA PRIVATE KEY-----
MIIE...
...
-----END RSA PRIVATE KEY-----
```
公钥：
```
-----BEGIN PUBLIC KEY-----
MIIB...
...
-----END PUBLIC KEY-----
```
甚至一个 `.pem` 文件里可以同时放多个证书：
```
-----BEGIN CERTIFICATE-----
服务器证书
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
中间CA证书1
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
中间CA证书2
-----END CERTIFICATE-----
```
这就是很多 `fullchain.pem` 的内容。
所以：
```
.pem
```
更多是在说：
> 这是 Base64 + BEGIN/END 头尾包起来的文本格式。
## crt
`.crt` 通常表示：
```
certificate
```
也就是“这是一个证书文件”。
但 `.crt` 文件内部可能有两种编码。
一种是 PEM：
```
-----BEGIN CERTIFICATE-----
MIIF...
-----END CERTIFICATE-----
```
这种本质上：
```
内容：X.509 certificate
编码：PEM
扩展名：.crt
```
另一种是 DER 二进制：
```
一堆二进制字节
```
本质上：
```
内容：X.509 certificate
编码：DER
扩展名：.crt
```
所以：
> `.crt` 更多在表达“内容是证书”，而不严格表达编码方式。
`.cer` 也是类似的，通常也是证书。
Windows 世界比较常见：
```
.cer
```
Linux/Nginx 世界比较常见：
```
.crt
.pem
```
但从技术上来说，后缀没有强制标准。
## key
`.key` 一般表示：
```
私钥
```
例如：
```
server.key
```
内容可能是：
```
-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----
```
或者：
```
-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----
```
`.key` 也只是惯例。
它通常使用 PEM 编码，但也不绝对。
## p12,pfx
`.p12` 和 `.pfx` 又是另一类东西。
这两个通常表示：
```
PKCS#12
```
它不是单个证书，而是一个“密码保护的容器”。
里面可以放：
```
server.p12
│
├── Private Key
│
├── Server Certificate
│
├── Intermediate CA
└── Intermediate CA
```
所以 PKCS#12 特别适合 Tomcat、Java、Windows、IIS 这一类环境。
`.pfx` 和 `.p12` 基本可以认为是同一种格式：
```
PKCS#12
```
只是历史命名不同。
你可以把它理解成：
```
.zip 包
```
这个类比不完全精确，但思路类似：
> 不是单个证书，而是一个能装多个密钥和证书的容器。

## jks
`.jks` 是 Java 自己传统的 KeyStore 格式：
```
Java KeyStore
```
里面也可以装：
```
PrivateKeyEntry
TrustedCertificateEntry
```
比如：
```
tomcat.jks
│
├── PrivateKeyEntry
│   ├── private key
│   └── certificate chain
│
└── trustedCertEntry
```
不过现在 Java 生态已经更倾向 PKCS#12。
现在新版本 Java 的默认 keystore type 很多情况下就是：
```
PKCS12
```
而不是 JKS。

---
你现在那个：
```
tomcat.keystore
```
尤其要注意：
`.keystore` 并不是一个标准格式名字。
只是文件名。
里面实际可能是：
```
JKS
```
也可能是：
```
PKCS12
```
所以要真正判断：
```
keytool -list -v -keystore tomcat.keystore
```
看：
```
Keystore type: PKCS12
```
或者：
```
Keystore type: JKS
```
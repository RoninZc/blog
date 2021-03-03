---
title: "RSA(七) X.509 自签名证书"
date: 2021-03-03T13:47:25+08:00
draft: true
toc: false
images:
tags: 
  - RSA
---

# 前言

继前文[RSA(六) X.509 CA 证书](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/) 所述，*X.509 CA* 证书是由 𝐶𝐴CA 认证中心签名并颁发的；但是最后，作者留下了这么一个疑问，就是如果在企业内网，我同样需要对公钥进行认证，但是因为不需要连接外网，所以并不需要 𝐶𝐴CA 证书(因为，𝐶𝐴CA 证书毕竟开销不菲)；那么是否有这样的一种可替代的方案，在不使用 𝐶𝐴CA 证书的前提下，能否保证公钥的合法性？答案是`自签名证书`；

备注，本文是作者的原创作品，转载请注明出处。

> 转载至`https://www.shangyang.me/2017/05/27/encrypt-rsa-selfsigned-certificate/`

# 定义

看下 wikipedia https://en.wikipedia.org/wiki/Self-signed_certificate 上的最重要的一段解释

> In technical terms a self-signed certificate is one signed with its own private key.

自签名证书说穿了，就是一个由自己的私钥进行签名的证书；

# 与 CA 证书的区别

通过 [RSA(六) X.509 CA 证书](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/) 章节我们知道，要保证公钥的合法性，我们需要把自己的公钥交给第三方 𝐶𝐴CA 机构，通过它的私钥来进行签名，并生成一张 𝐶𝐴CA 证书并颁发给用户；而与 𝐶𝐴CA 证书相对应的就是自签名证书，也就是说，我自己的公钥不交给第三方的 𝐶𝐴CA 机构进行签名，而是直接由自己的私钥进行签名，并生成一张自签名的证书；

# 如何生成

这里我主要讲解一下，如果通过 *openssl* 来生成自签名的证书，

```
$ openssl req \
>        -newkey rsa:2048 -nodes -keyout private.key \
>        -x509 -days 365 -out self-signed.crt
```

然后同样需要输入 [CSR 相关的信息](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/#通过-CSR-申请-X509-CA-证书)来申请；最后，会在本地目录中生成一个私钥 *private.key* 和一张自签名的证书 *self-signed.crt*；注意，自签名证书文件的后缀为 *.crt*；下面来看看各个参数的简要说明，

`-x509`告诉 openssl 生成一张自签名的证书；
`-nodes`告诉 openssl 在生成私钥的时候忽略密码

参考
[How To Create a Self-Signed SSL Certificate for Apache in Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-16-04)

[OpenSSL Essentials: Working with SSL Certificates, Private Keys and CSRs](https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs)

# 内部结构

我们来查看一下刚才通过 openssl 生成的自签名证书 *self-signed.crt* 的内部结构；

```
$ openssl x509 -text -noout -in domain.crt
```

内容如下，

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            9f:56:fd:f5:9d:13:2e:d2
        Signature Algorithm: sha1WithRSAEncryption
        Issuer: C=CN, ST=ChengDu, L=ChengDu, O=HRX, OU=HRX, CN=HRX/emailAddress=comedshang@163.com
        Validity
            Not Before: May 27 13:42:23 2017 GMT
            Not After : May 27 13:42:23 2018 GMT
        Subject: C=CN, ST=ChengDu, L=ChengDu, O=HRX, OU=HRX, CN=HRX/emailAddress=comedshang@163.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
            RSA Public Key: (2048 bit)
                Modulus (2048 bit):
                    00:e5:06:e7:94:b4:ff:ad:ae:26:9c:2c:76:2e:d2:
                    c7:f6:b3:51:9a:15:1f:d6:6f:ee:f7:7b:13:61:b5:
                    d5:07:de:6f:e4:78:05:cc:b3:74:fc:c4:ec:7f:07:
                    c7:b3:1b:c3:b6:c5:e8:9a:00:48:5c:e3:7c:51:e2:
                    34:1d:0e:e0:2f:4f:3d:4a:68:e3:fd:b4:c2:79:7f:
                    f3:ac:24:6d:71:d6:44:7a:97:7a:10:e0:5b:2e:1c:
                    80:91:71:4c:45:e8:97:2c:5d:30:68:1c:2a:28:96:
                    24:1a:a2:40:ad:d8:aa:9b:d8:3b:89:e4:eb:a0:77:
                    a4:1f:ab:5f:7d:8e:82:37:1d:c5:f5:9d:d6:5a:19:
                    ea:5e:57:35:f9:ba:63:66:f0:4c:48:97:22:8f:2f:
                    bf:7f:51:fe:bf:20:01:3c:17:11:9d:82:01:7c:f5:
                    31:04:c7:33:10:75:5c:2a:b0:ae:d1:12:fe:6e:b9:
                    5b:cf:67:1e:78:b6:ae:87:70:65:f8:c6:88:c6:10:
                    7c:58:f5:7e:15:8a:47:97:9c:e1:68:7b:ed:7c:db:
                    e5:6a:de:c1:4b:a1:05:6d:da:1e:bf:44:f9:05:6b:
                    bb:c3:41:f3:f5:a8:39:7a:2b:eb:ac:d9:61:30:bf:
                    0d:56:54:f8:39:b9:fc:01:93:5a:1d:aa:bf:2f:c8:
                    30:97
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                CB:3C:17:D8:D4:F1:C2:7C:00:57:46:48:1F:9B:A2:4F:DA:9A:92:66
            X509v3 Authority Key Identifier: 
                keyid:CB:3C:17:D8:D4:F1:C2:7C:00:57:46:48:1F:9B:A2:4F:DA:9A:92:66
                DirName:/C=CN/ST=ChengDu/L=ChengDu/O=HRX/OU=HRX/CN=HRX/emailAddress=comedshang@163.com
                serial:9F:56:FD:F5:9D:13:2E:D2

            X509v3 Basic Constraints: 
                CA:TRUE
    Signature Algorithm: sha1WithRSAEncryption
        b1:6e:10:48:3c:4b:d1:4d:6e:5c:14:34:79:89:e0:95:3e:48:
        3d:53:6c:65:64:ce:90:e2:da:17:2f:e2:8e:13:6a:1c:e2:d8:
        b9:4c:f2:24:19:60:64:ae:66:cb:e6:82:de:a5:22:40:8e:50:
        94:4c:5f:87:6e:f6:c4:be:ff:3b:75:eb:3a:f5:eb:aa:47:c4:
        5c:14:d9:7d:38:ee:28:8c:96:8f:22:a1:85:63:a9:e3:23:d2:
        64:fe:50:dd:ab:4e:53:f6:f7:67:c1:ec:39:89:20:04:f1:3f:
        f1:18:5a:ab:77:eb:02:d3:93:34:ca:e8:81:6b:6f:60:5c:9d:
        b7:1f:e9:be:cb:9a:b2:73:47:52:d7:d6:89:ce:34:4c:46:3c:
        c3:73:9f:93:07:72:41:d4:64:f9:f1:52:56:78:ac:96:fe:da:
        b5:c0:b3:8f:e0:5e:8c:a3:bf:21:d7:99:27:ff:65:e4:62:8c:
        15:14:8f:bb:04:54:30:4e:5e:32:a8:8c:ab:70:27:14:99:5e:
        9b:11:dc:0a:e8:d4:59:8b:98:de:30:b3:5e:f2:8c:e4:b3:2b:
        62:07:9a:74:52:c0:e3:54:4c:86:4b:cd:88:f3:6b:1a:c8:66:
        d6:ab:1d:c5:12:e2:66:0a:01:a8:3d:0c:f8:d4:ac:1d:74:80:
        83:06:3f:6d
```



备注，要参看其它证书类型的内容，参考 https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs#view-certificates

一些特性，

* 首先，我们可以看到不像 𝐶𝐴CA 证书那样有多层证书结构，自签名证书只有一层证书结构；也就是没有 Certificate Chain 的概念；
* 再次，可以看到 *Issuer* 发布者和证书拥有者 *Subject* 是相同的，都是 *C=CN, ST=ChengDu, L=ChengDu, O=HRX, OU=HRX, [CN=HRX/emailAddress=comedshang@163.com](mailto:CN=HRX/emailAddress=comedshang@163.com)*，表明证书的签名方和证书自己是同一个机构(或者角色)，这也就是`自签名`命名的由来；

关键内容，

* Subject Public Key Info
  这段内容就是公钥的内容了；
* Signature Algorithm: sha1WithRSAEncryption
  这段内容就表示对公钥和身份信息一起的签名信息了，可以看到，采用的是 𝑆𝐻𝐴1SHA1 算法；

最后，比较重要的是，因为该证书只有一层，所以，自签名证书在通讯过程中扮演的也就是 Root Certificate，通常是需要被加载如客户端的 trust store 中；

# 如何保证内网通讯的安全性

比如内网中，Alice 想和 Bob 保持可靠的加密通讯；

* 首先，Alice 需要生成秘钥 𝐾𝑝Kp 以及自签名证书 𝐶𝑠𝑒𝑙𝑓Cself

* 然后，Bob 通过某种方式将 Alice 的 𝐶𝑠𝑒𝑙𝑓Cself 加入自己的 trust store 中；
  注意，如果 Bob 是通过浏览器访问的 Alice 站点，那么浏览器会提示，是否相信该证书，如果选择相信，这个时候，浏览器会自动将该 𝐶𝑠𝑒𝑙𝑓Cself 证书加入自己的 trust store 中；

* 之后，某个时刻，Bob 向 Alice 发起安全通讯的会话，双方握手过程中，

* Alice 将 𝐶′𝑠𝑒𝑙𝑓Cself′ (X.509 自签名)证书发送给 Bob

* Bob 通过 trust store 中之前预加载的 𝐶𝑠𝑒𝑙𝑓Cself 中所指明的签名算法对 𝐶′𝑠𝑒𝑙𝑓Cself′ 进行签名，得到签名 𝑆′S′

* 最后 Bob 使用 trust store 中 𝐶𝑠𝑒𝑙𝑓Cself 的签名 𝑆S 与 𝑆′S′ 进行比较，

* 若相等，则表示 Alice 当前发送过来的证书是可靠的，也就是说，其身份信息和公钥是合法的，没有被伪造过；

* 若不等，则表示 Alice 当前发送过来的证书是不可靠的，其身份信息或者是公钥是被篡改过的；应当立即停止与 Alice 的通讯；这种情况下，往往会提示握手失败，证书不可靠；

但切记，在公网中使用自签名证书的 SSL 加密通讯的方式是不可靠的，其根本原因是，因为没有公钥基础设施的支持，其自签名证书很容易被伪造；这部分内容在之前的 [RSA(五) PKI (Public Key Infrastructure) 公钥基础设施](https://www.shangyang.me/2017/05/25/encrypt-rsa-pki/) 等系列文章中有过详细的阐述，有兴趣的读者可以从这里开始；

# 如何加入 trust store

参考如下的几篇文章，

openssl with Apache:

[How To Create a Self-Signed SSL Certificate for Apache in Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-16-04)

[How to create a self-signed SSL Certificate for Apache](https://www.akadia.com/services/ssh_test_certificate.html)

java keytool series:

Tomcat SSL Installation Instructions
: https://www.sslshopper.com/tomcat-ssl-installation-instructions.html

java keytool - Key and Certificate Management Tool: http://docs.oracle.com/javase/1.5.0/docs/tooldocs/solaris/keytool.html

Import a private key into a Java Key Store: http://commandlinefanatic.com/cgi-bin/showarticle.cgi?article=art049
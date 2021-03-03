---
title: "RSA(六) X.509 CA 证书"
date: 2021-03-03T13:46:25+08:00
draft: true
toc: false
images:
tags: 
  - RSA
---

# 前言

由前文 [RSA(五) PKI (Public Key Infrastructure) 公钥基础设施](https://www.shangyang.me/2017/05/25/encrypt-rsa-pki/)可知，CA 证书授权中心颁发给用户的是一张 *X.509 证书*；本篇文章，博主将带领大家一探 *X.509 证书*的究竟；

重要：本文为作者的原创作品，转载需注明出处；

> 转载至`https://www.shangyang.me/2017/05/27/encrypt-rsa-selfsigned-certificate/`

# 通过 CSR 申请 X509 CA 证书

这里大致讲解如何申请 *X.509 CA* 证书的流程，

1. 用户先在本地通过 RSA 生成一对公钥 𝐾𝑝Kp 和密钥 𝐾𝑠Ks，
2. 然后，用户在本地生成一张 𝐶𝑆𝑅CSR 证书，既 [Certificate Signing Request](https://en.wikipedia.org/wiki/Certificate_signing_request)
   * 需要填写诸如你的域名，公司名，部门名称，城市名，地区名，国家名，电子邮件等等证明你身份的信息，
   * 添加用户公钥 𝐾𝑝
   * 将上述信息通过 𝐾𝑠 进行签名得到 𝑆𝑐𝑠𝑟
   * 将上述的签名 𝑆𝑐𝑠𝑟、𝐾𝑝 以及身份信息合并成为一张 𝐶𝑆𝑅 证书 ;
     TODO: 需要合并 𝑆𝑐𝑠𝑟 吗？这个还需要进一步求证……
3. 申请者通过向 CA 中心或者 RA 中心提交 𝐶𝑆𝑅 证书，申请 *X.509 证书*；
4. 𝐶𝐴 中心用它的密钥 𝐾𝑠(𝑐𝑎)Ks(ca) 对用户提交的 𝐶𝑆𝑅 证书进行签名，将签名和 𝐶𝑆𝑅 合并生成一张 *X.509 证书*；详细的签名过程，
   * 𝐶𝐴 中心通过签名算法(比如 MD5)对 𝐶𝑆𝑅 证书进行签名；
   * 然后通过 𝐾𝑠(𝑐𝑎)) 对上述的签名进行加密，得到加密后的签名；
   * 最后，将加密后的签名和用户提交的 𝐶𝑆𝑅合并成为一张 *X.509 证书*；
5. 最后将该 *X.509 证书* 颁发给用户；

备注，如何生成 CSR 证书可以参考 https://www.sslshopper.com/article-most-common-openssl-commands.html

# 证书链 Certificate Chain

## 什么是证书链

*X.509 证书* 中往往不止一张证书，而是由一系列的证书所组成的证书链，通常包含这样三层证书所构成的证书链，

![img](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/certificate-chain.png)

1. root certificate，根证书
2. intermediates certificates，一系列中间证书
3. end-user certificate，终端用户证书

相信大部分读者会和我一样，迷惑，为什么一张 *X.509* 证书要搞得这么复杂？

这样做其实是有历史原因的，那是因为随系统和浏览器预安装的根证书毕竟是有限的，往往就那么些，一旦系统已经发布，大部分用户也已经安装好了，Root Certificate 既根证书就没有办法再通过预安装的方式来进行扩充了；但是我们知道，会有日新月异的新的 CA 公司成立，并被允许授权颁发 *X.509* 证书，毕竟，我们要允许市场的充分竞争，那么它们的公钥如何预安装到客户端的浏览器和操作系统上呢？如果不能预安装，由前文[如何保证公钥不被篡改的](https://www.shangyang.me/2017/05/25/encrypt-rsa-pki/#如何保证公钥不被篡改的)分析可知，它们所颁发的证书是不可靠的，是可以被篡改的；那么如何调和这个矛盾呢？该怎么办呢？

于是，intermediates certificates 中间证书就出现了，这些证书就是表示由 Root Certificate 证书所签名认证的证书，这样，新的 CA 公司所颁发的证书可以由 Root Certificate 进行认证，保证其合法性和可靠性，这样就充分允许了新的 CA 公司成立并参与 CA 这块市场的竞争了；

下面，就用这么一个形象的例子来描述，比如，有一天，中国成立了一家 ShangYang 公司，该公司的主营业务就是为广大客户进行证书授权，参与 CA 领域的商业竞争；那么这个时候呢，我需要向掌管 Root Certificate 的机构提交申请，请它签名我的 CA 公钥 (说好的市场充分竞争呢？Root Certificate 机构不就是赤裸裸的垄断机构？谁知道呢..)，生成一张 intermediates certificate 中间证书；这样，由 ShangYang 公司所签名的证书就得到了 Root Certificate 的认证，那么由该公司所颁发的 *X.509* 证书就是合法，正规的了；

由此，我们也能够大致知道整个证书链的验证过程了，证书链的验证过程将会在后续[证书链的验证过程](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/#证书链的验证过程)进行详细的描述；

总结一下，

根证书是被预装到客户端电脑或者用户其它终端设备上的(比如手机)，它的作用主要是验证 CA 证书签名的合法性，也就是保证 CA 的证书(含 CA 的公钥)的合法性；最后，end-user certificate 终端用户证书，该证书由 CA 的证书保证其合法性；所以，可以看到，各个证书的验证过程是一环扣一环的，根证书验证 CA 证书的合法性，CA 证书验证用户证书的合法性；

## 证书链中每个证书所包含的内容

我们来看看 wikipedia.org 的 *X.509* 证书的情况，他是由 GlobalSign 机构颁发的，

### End-user(或 End-entity) certificate 既终端用户证书

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            10:e6:fc:62:b7:41:8a:d5:00:5e:45:b6
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=BE, O=GlobalSign nv-sa, CN=GlobalSign Organization Validation CA - SHA256 - G2
        Validity
            Not Before: Nov 21 08:00:00 2016 GMT
            Not After : Nov 22 07:59:59 2017 GMT
        Subject: C=US, ST=California, L=San Francisco, O=Wikimedia Foundation, Inc., CN=*.wikipedia.org
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub: 
                    04:c9:22:69:31:8a:d6:6c:ea:da:c3:7f:2c:ac:a5:
                    af:c0:02:ea:81:cb:65:b9:fd:0c:6d:46:5b:c9:1e:
                    ed:b2:ac:2a:1b:4a:ec:80:7b:e7:1a:51:e0:df:f7:
                    c7:4a:20:7b:91:4b:20:07:21:ce:cf:68:65:8c:c6:
                    9d:3b:ef:d5:c1
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Agreement
            Authority Information Access: 
                CA Issuers - URI:http://secure.globalsign.com/cacert/gsorganizationvalsha2g2r1.crt
                OCSP - URI:http://ocsp2.globalsign.com/gsorganizationvalsha2g2

            X509v3 Certificate Policies: 
                Policy: 1.3.6.1.4.1.4146.1.20
                  CPS: https://www.globalsign.com/repository/
                Policy: 2.23.140.1.2.2

            X509v3 Basic Constraints: 
                CA:FALSE
            X509v3 CRL Distribution Points: 

                Full Name:
                  URI:http://crl.globalsign.com/gs/gsorganizationvalsha2g2.crl

            X509v3 Subject Alternative Name: 
                DNS:*.wikipedia.org, DNS:*.m.mediawiki.org, DNS:*.m.wikibooks.org, DNS:*.m.wikidata.org, DNS:*.m.wikimedia.org, DNS:*.m.wikimediafoundation.org, DNS:*.m.wikinews.org, DNS:*.m.wikipedia.org, DNS:*.m.wikiquote.org, DNS:*.m.wikisource.org, DNS:*.m.wikiversity.org, DNS:*.m.wikivoyage.org, DNS:*.m.wiktionary.org, DNS:*.mediawiki.org, DNS:*.planet.wikimedia.org, DNS:*.wikibooks.org, DNS:*.wikidata.org, DNS:*.wikimedia.org, DNS:*.wikimediafoundation.org, DNS:*.wikinews.org, DNS:*.wikiquote.org, DNS:*.wikisource.org, DNS:*.wikiversity.org, DNS:*.wikivoyage.org, DNS:*.wiktionary.org, DNS:*.wmfusercontent.org, DNS:*.zero.wikipedia.org, DNS:mediawiki.org, DNS:w.wiki, DNS:wikibooks.org, DNS:wikidata.org, DNS:wikimedia.org, DNS:wikimediafoundation.org, DNS:wikinews.org, DNS:wikiquote.org, DNS:wikisource.org, DNS:wikiversity.org, DNS:wikivoyage.org, DNS:wiktionary.org, DNS:wmfusercontent.org, DNS:wikipedia.org
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Subject Key Identifier: 
                28:2A:26:2A:57:8B:3B:CE:B4:D6:AB:54:EF:D7:38:21:2C:49:5C:36
            X509v3 Authority Key Identifier: 
                keyid:96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C

    Signature Algorithm: sha256WithRSAEncryption
         8b:c3:ed:d1:9d:39:6f:af:40:72:bd:1e:18:5e:30:54:23:35:
         ...
```

上面这张证书就是 wikipedia.org 的终端用户证书了，看看几个关键的属性

1. Subject

   ```
   Subject: C=US, ST=California, L=San Francisco, O=Wikimedia Foundation, Inc., CN=*.wikipedia.org
   ```

   可以看到该证书的主体结构，表示该证书的主体机构是 wikipedia.org，以及相关的一些附属信息，比如国家，地域，公司名称等等；

2. Subject Alternative Name
   表述了该证书还可以用在哪些域名上，这里定义了好些其它的可用域名的验证上，

   ```
   X509v3 Subject Alternative Name: 
     DNS:*.wikipedia.org, DNS:*.m.mediawiki.org, DNS:*.m.wikibooks.org, DNS:*.m.wikidata.org, DNS:*.m.wikimedia.org, DNS:*.m.wikimediafoundation.org, DNS:*.m.wikinews.org, DNS:*.m.wikipedia.org, DNS:*.m.wikiquote.org, DNS:*.m.wikisource.org, DNS:*.m.wikiversity.org, DNS:*.m.wikivoyage.org, DNS:*.m.wiktionary.org, DNS:*.mediawiki.org, DNS:*.planet.wikimedia.org, DNS:*.wikibooks.org, DNS:*.wikidata.org, DNS:*.wikimedia.org, DNS:*.wikimediafoundation.org, DNS:*.wikinews.org, DNS:*.wikiquote.org, DNS:*.wikisource.org, DNS:*.wikiversity.org, DNS:*.wikivoyage.org, DNS:*.wiktionary.org, DNS:*.wmfusercontent.org, DNS:*.zero.wikipedia.org, DNS:mediawiki.org, DNS:w.wiki, DNS:wikibooks.org, DNS:wikidata.org, DNS:wikimedia.org, DNS:wikimediafoundation.org, DNS:wikinews.org, DNS:wikiquote.org, DNS:wikisource.org, DNS:wikiversity.org, DNS:wikivoyage.org, DNS:wiktionary.org, DNS:wmfusercontent.org, DNS:wikipedia.org
   ```

3. Subject Public Key

   ```
   Subject Public Key Info:
       Public Key Algorithm: id-ecPublicKey
           Public-Key: (256 bit)
           pub: 
               04:c9:22:69:31:8a:d6:6c:ea:da:c3:7f:2c:ac:a5:
               af:c0:02:ea:81:cb:65:b9:fd:0c:6d:46:5b:c9:1e:
               ed:b2:ac:2a:1b:4a:ec:80:7b:e7:1a:51:e0:df:f7:
               c7:4a:20:7b:91:4b:20:07:21:ce:cf:68:65:8c:c6:
               9d:3b:ef:d5:c1
           ASN1 OID: prime256v1
           NIST CURVE: P-256
   ```

   这就是终端用户的公钥信息了，注意，这里的公钥并不是采用的 RSA 算法，而是采用的 DSA 算法，具体可参考 [ECDSA](https://en.wikipedia.org/wiki/ECDSA)；

4. Signature Algorithm
   这部分内容表示所用的签名算法以及被 CA 中心签名后的内容，

   ```
   Signature Algorithm: sha256WithRSAEncryption
       8b:c3:ed:d1:9d:39:6f:af:40:72:bd:1e:18:5e:30:54:23:35:
   ```

   可以看到，该签名算法是采用的 SHA256 算法，并通过 RSA 加密(这里是通过 CA 的私钥加密)，所以这里得到的其实是被 CA 私钥加密后的 SHA 签名；后面对应 *8b:c3:ed:d1:9d:39:6f:af:40:72:bd:1e:18:5e:30:54:23:35:* 就是该加密后的签名；

从前面的系列文章 [RSA(五) PKI (Public Key Infrastructure) 公钥基础设施](https://www.shangyang.me/2017/05/25/encrypt-rsa-pki/)我们知道，终端用户公钥签名是需要被相关的 CA 证书验证的(既是通过 CA 的公钥进行解密验证)；那么它是如何找到对应的 CA 证书并进行验证的呢？通过终端用户证书中的两个字段；

1. Issuer

   ```
   Issuer: C=BE, O=GlobalSign nv-sa, CN=GlobalSign Organization Validation CA - SHA256 - G2
   ```

   可以明显的看到，该用户终端证书是由 **GlobalSign** 𝐶𝐴CA 中心进行验证并签发的。

2. Authority Key Identifier

   ```
   X509v3 Authority Key Identifier: 
              keyid:96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C
   ```

   𝐶𝐴中心的身份 𝐼𝐷

那顾名思义，终端用户证书将会找到与 *Issuer* 和 *Authority Key Identifier* 两者都匹配的 𝐶𝐴 证书来对它进行验证；好了，接下来，我们看看 𝐶𝐴 证书长什么样，也就是 *Intermediate certificate*；

### Intermediate certificate 既 CA 证书

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            04:00:00:00:00:01:44:4e:f0:42:47
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=BE, O=GlobalSign nv-sa, OU=Root CA, CN=GlobalSign Root CA
        Validity
            Not Before: Feb 20 10:00:00 2014 GMT
            Not After : Feb 20 10:00:00 2024 GMT
        Subject: C=BE, O=GlobalSign nv-sa, CN=GlobalSign Organization Validation CA - SHA256 - G2
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:c7:0e:6c:3f:23:93:7f:cc:70:a5:9d:20:c3:0e:
                    ...
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Certificate Sign, CRL Sign
            X509v3 Basic Constraints: critical
                CA:TRUE, pathlen:0
            X509v3 Subject Key Identifier:
                96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C
            X509v3 Certificate Policies:
                Policy: X509v3 Any Policy
                  CPS: https://www.globalsign.com/repository/

            X509v3 CRL Distribution Points:

                Full Name:
                  URI:http://crl.globalsign.net/root.crl

            Authority Information Access:
                OCSP - URI:http://ocsp.globalsign.com/rootr1

            X509v3 Authority Key Identifier:
                keyid:60:7B:66:1A:45:0D:97:CA:89:50:2F:7D:04:CD:34:A8:FF:FC:FD:4B

    Signature Algorithm: sha256WithRSAEncryption
         46:2a:ee:5e:bd:ae:01:60:37:31:11:86:71:74:b6:46:49:c8:
         ...
```

*Intermidate certificate* 既是 𝐶𝐴 证书，是用来验证用户终端证书(既 *End-entity certificate*)的；通过两个字段来匹配用户终端证书，

* 𝐶𝐴 证书中的 *Issuer* 和用户终端证书中的 *Issuer* 相匹配

* 

  𝐶𝐴

   

  证书中的

   

  Subject Key Identifier

   

  与用户终端证书中的

   

  Authority Key Identifier

   

  相匹配；

  ```
  X509v3 Subject Key Identifier:
                96:DE:61:F1:BD:1C:16:29:53:1C:C0:CC:7D:3B:83:00:40:E6:1A:7C
  ```

与 𝐶𝐴 证书相匹配的用户终端证书将会被该 𝐶𝐴 证书所验证；

### Root Certificate 既根证书

```
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            04:00:00:00:00:01:15:4b:5a:c3:94
    Signature Algorithm: sha1WithRSAEncryption
        Issuer: C=BE, O=GlobalSign nv-sa, OU=Root CA, CN=GlobalSign Root CA
        Validity
            Not Before: Sep  1 12:00:00 1998 GMT
            Not After : Jan 28 12:00:00 2028 GMT
        Subject: C=BE, O=GlobalSign nv-sa, OU=Root CA, CN=GlobalSign Root CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:da:0e:e6:99:8d:ce:a3:e3:4f:8a:7e:fb:f1:8b:
                    ...
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Certificate Sign, CRL Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier: 
                60:7B:66:1A:45:0D:97:CA:89:50:2F:7D:04:CD:34:A8:FF:FC:FD:4B
    Signature Algorithm: sha1WithRSAEncryption
         d6:73:e7:7c:4f:76:d0:8d:bf:ec:ba:a2:be:34:c5:28:32:b5:
         ...
```

*Intermediate certificate* 既是 𝐶𝐴 证书将会被 *Root certificate* 进行验证，并且 *Root certificate* 是证书验证链中的最后一环，所以的验证将会到此为止；那么 𝐶𝐴 证书又是如何找到对应的 *Root certificate* 进行验证的呢？主要通过如下的规则进行匹配

1. 𝐶𝐴 证书中的 *Issuer* 需要与 *Root certificate* 中的 *Issuer* 匹配
2. 𝐶𝐴 证书中的 *Authority Key Identifier* 字段需要与 *Root certificate* 证书中的 *Subject Key Identifier* 字段相匹配

这样，与之匹配的 𝐶𝐴 证书将会由该 *Root certificate* 证书进行验证；

最后，需要强调的是，𝐶𝐴 证书并不会随浏览器和系统的安装而预安装到用户的设备上，`被预安装到用户设备上的只有 Root certificate`；这样呢，就保证了终端用户的公钥的可靠性和安全性；另外，通过 *Root certificate* 会被安装到系统的 [trust store](https://en.wikipedia.org/wiki/Public_key_certificate#Root_programs)中，主流的有

* [Microsoft Root Program](https://technet.microsoft.com/en-us/library/cc751157.aspx)
* [Apple Root Program](https://www.apple.com/certificateauthority/ca_program.html)
* [Mozilla Root Program](https://www.mozilla.org/en-US/about/governance/policies/security-group/certs/policy/)
* [Oracle Java root program](http://www.oracle.com/technetwork/java/javase/javasecarootcertsprogram-1876540.html)
* Adobe [AATL](https://helpx.adobe.com/acrobat/kb/approved-trust-list2.html) and [EUTL](https://blogs.adobe.com/documentcloud/eu-trusted-list-now-available-in-adobe-acrobat/) root programs (used for document signing)

后续有时间的话，准备对 Java 的 trust store 进行一下梳理和介绍；

## 证书链的验证过程

有了上述分析以后，证书链的验证过程就显而易见了

* End-user certificate 通过 𝐶𝐴 证书进行验证
* 𝐶𝐴 证书经过 Root certificate 证书进行验证
* 完毕

# 实践

那么这里作者带领大家通过 CSR 的方式申请一个免费的 CA 证书，现在通过阿里云可以免费申请一个固定域名的 Symantec 的 CA 证书；

1. 登录阿里云产品中心，选择安全 -> CA 证书![img](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/apply-ca-how-step-1.png)

2. 然后选择免费的 DV 类型的证书![img](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/apply-ca-how-step-2.png)

3. 购买好了以后，进入管理后台管理，进入安全（云盾）-> 证书服务，然后对证书进行补全![img](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/apply-ca-how-step-3.png)补全既是需要用户自己提交 CSR 申请证书；

4. 填写域名信息，注意，这里只能绑定一个唯一的域名，且不能写任何的通配符![img](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/apply-ca-how-step-4.png)

5. 下一步将会填写一些个人信息，比如姓名、手机号码、地址等![img](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/apply-ca-how-step-5.png)这里要强调的是，阿里云需要验证域名的归属，既验证该域名的确归你所有；可以通过 DNS 和 文件 的两种方式进行验证；这里呢，我选择的是通过 DNS 验证，验证的大致过程是，阿里会向你的申请邮箱中发送一份邮件，邮件的内容中包含了如何验证域名的方法，里面包含一条用于验证的 TXT 记录，这个需要到个人域名管理中心去配置 TXT 转发规则即可；

6. 好了，这一步是关键了，这里有两个选项，由系统生成 CSR 或者自己生成 CSR，这里为了演示自提交 CSR 证书申请的方式，所以，我们选择自己生成 CSR![img](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/apply-ca-how-step-6.png)

7. 使用 openssl 命令生成 CSR

   ```
   $ openssl req -out CSR.csr -new -newkey rsa:2048 -nodes -keyout private.key
   ```

   然后提示输入国家、地址、姓名、域名、邮箱等等信息；如果是为公司申请邮箱，那么这里填写公司相关信息即可；这里`尤其`要`注意`的是，在输入域名的时候，必须与第 4 步的域名相匹配，否则第 9 步审核不能通过，输入域名的时候，提示输入 *Common Name (e.g. server FQDN or YOUR name) []* 的信息；最后会在本地生成两个文件，CSR.csr 和 private.key

8. 用文本编辑器打开 CSR.csr，可以看到大致内容如下

   ```
   -----BEGIN CERTIFICATE REQUEST-----
   MIIC3DCCAcQCAQAwgZYxCzAJBgNVBAYTAkNOMRAwDgYDVQQIEwdTaWNodWFuMRAw
   DgYDVQQHEwdDaGVuZ2R1MRQwEgYDVQQKEwtIdWlSb25nWGluZzEUMBIGA1UECxML
   SHVpUm9uZ1hpbmcxFDASBgNVBAMTC0h1aVJvbmdYaW5nMSEwHwYJKoZIhvcNAQkB
   FhJodWlyb25neGluZ0BocnguYWkwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
   AoIBAQC7MY7hTjbv7DkGoDTmvQ2toAe8nMTlQGPh+r4VvD+zzEiEudFPEI1cIFLr
   BLTfSyn9Awv7lgIjhJ4ghDkjAGwHnrTxzvldjfZkpKuBK9H8Vy2t7sorgoxEBF7j
   VbiiTBtSG6+ZNw8esqt5EECT19aP/RyJp65f8lysHwcHmZmVGaDq/VwQcbyuI6Vs
   ko/7sSchOAgUWn66oS+xw7mGYR212mQv6Bz0g0L1NVep8Doz8O2pWPHT1ZOdpBDU
   rzJJxHXzUgKVgYsAgMBAAGgADANBgkqhkiG9w0BAQUFAAOCAQEA
   V6rSXeGO4Z0q2sZ9gbdUBmpQ8AAdloByhd1BcwHuHt/nfPj59L3CT3EnTEez7cDt
   RUCbI2FbThBFfjngfTNE3PjsTsheCdAxoV6yRPo7Fpb5AKkhXDra1jjVjsY0maFl
   N23okpDCMzmUD2peKqumYhdHBw8wB3Y5HZQxxq688DwlHn0bLnylUPk/hDfuMzIs
   5vLIyDSGQiCwq9sU8wjhQOXqzZ37FgJcZ8GyvaJ3kUWDlLDPIGMiXQ0p4T39/ZaE
   my/C0JxSLiAKJs3L2f7HfKwUoRZDDnCS0WMdQunvxC4Dd7hyddCij6E1ExnT7EzC
   kiPq8xiGl2HRGW/JfWC3XA==
   -----END CERTIFICATE REQUEST-----
   ```

   可以看到，是一个经过 Base64 编码的 PEM 格式的内容；

9. 将上述内容复制，粘贴到阿里证书服务页面，点击保存![img](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/apply-ca-how-step-7.png)注意第 7 步的描述，必须保证 CSR 中的域名地址与第 4 步申请的时候填写的域名地址相匹配才行；

10. 验证域名的归属
    由于第 5 步中，作者是选择的 DNS 验证的方式，所以，第 9 步完成以后，阿里会发送一封邮件包含需要验证的 TXT 记录值，这里只需要到域名管理中心配置一下 TXT 值，即可验证通过；域名管理中心配置好了以后，大致内容如下，![img](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/apply-ca-how-step-9.png)不过，有时候，邮件迟迟不发送，这个时候，你可以直接点击`进度`按钮，也可以显示相关的 TXT 记录；![img](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/apply-ca-how-step-8.png)

11. 申请成功，当验证通过以后，状态便会变为`已签发`![img](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/apply-ca-how-step-10.png)这个时候，你就可以在证书管理后台中去下载该 CA 证书了..

All done…

后记于 2018-01-30 1:21 pm
阿里云上验证域名归属的步骤有变化，也就是上面的第 10 步，在添加主机记录的时候前面要加上 _dnsauth 前缀才行，如图
![apply-ca-how-step-10-2.png](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/apply-ca-how-step-10-2.png)
并且不再发邮件提示配置知己记录 TXT 值了；对应的域名配置如下，
![apply-ca-how-step-10-3.png](https://www.shangyang.me/2017/05/27/encrypt-rsa-x509/apply-ca-how-step-10-3.png)

更多详情参考 https://bbs.aliyun.com/read/573056.html?spm=a2c4e.11155515.0.0.kL3FVf

# 后续

这里主要讲解的是通过 𝐶𝐴 签名的证书，那如果，只是内网服务器，不需要连接外网，那么，实际上，我也就用不到这种公共的基础设施来保证我的证书的合法性；这个时候，X.509 证书还提供了一种自签名证书，什么意思呢？就是说，你可以自己生成 *Root certificate*，然后将它加入你本机的 trust store 中，用它来验证你的证书的合法性，这就是自签名证书了；详情参考
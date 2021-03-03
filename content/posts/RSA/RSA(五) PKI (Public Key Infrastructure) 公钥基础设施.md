---
title: "RSA(五) PKI (Public Key Infrastructure) 公钥基础设施"
date: 2021-03-03T13:45:25+08:00
draft: true
toc: false
images:
tags: 
  - RSA
---

# 前言

在上一篇《[RSA(四) 签名 Signature](https://www.shangyang.me/2017/05/24/encrypt-rsa-signature/)》章节[签名 Signature](https://www.shangyang.me/2017/05/24/encrypt-rsa-signature/#签名-Signature) 中提到，要使用签名(Signature)机制来保证信息再传递过程中没有被第三方所篡改，有一个前提，就是必须保证，在公钥的传递过程中不被他人篡改，否则，整个签名机制就形同一张白纸，毫无用处；那么又该如何保证公钥的合法性，保证公钥本身没有被篡改过呢？这就是 PKI 公钥基础设施要完成的工作和达到的目的；

备注，本文是作者的原创作品，转载请注明出处。

> 转载至`https://www.shangyang.me/2017/05/25/encrypt-rsa-pki/`

# 公钥被篡改

首先，我们来看看，在通讯过程中，公钥是如何被他人所篡改的？也就是 Man in Middle Attack 是怎么做到的？

先来看一个正常的，通过公钥和私钥加密通讯的场景，

![img](https://www.shangyang.me/2017/05/25/encrypt-rsa-pki/security-communicate-with-rsa.png)

服务器端生成公钥和私钥，并将公钥通过网络发送给客户端，客户端使用公钥加密 DES 对称加密密钥，然后将 DES 密钥发送给服务器端，之后，双方便可以进行加密通讯了；备注，这也是基于 RSA 加密通讯的基础；但是，这样做，并不可靠，下面我们再来看一个公钥被挟持并被篡改的场景；

![img](https://www.shangyang.me/2017/05/25/encrypt-rsa-pki/security-communicate-with-rsa-tamper.png)

这次通讯的时候，不幸的是，客户端与服务器通讯的时候，正好经过了一个黑客的路由器，通过网络嗅探，它截获了服务器所发送的公钥，并利用自己的私钥，生成了一个新的公钥，并替代原有的公钥并将这个新的公钥发送给了客户端，这个过程就是公钥被截获，被篡改的过程；而后，客户端将使用被篡改过后的公钥进行加密通讯，所以，凡是经过客户端加密的信息，全部可以被黑客的私钥解密，也就导致了，加密通讯`彻底失效`；正是在这种背景之下，PKI (Public Key Infrasture) 公钥基础设施孕育而生；

后记，发现还漏了一环，首先要知道的是，当通过 RSA 建立好 SSL 通讯通道以后，实际上，为了效率，该通道上数据实际上是通过对称加密密钥 AES 进行加密传输的，所以，能否保证数据通讯的安全性的重中之重的环节就是保证 AES 密钥在传输过程中的安全性，正如上图所描述的那样，当黑客截获到加密后的 AES 密钥以后，首先需要通过黑客自己的私钥对其解密，然后再通过黑客所截获的服务器的公钥对该 AES 密钥进行加密，然后再传输给服务器端，如果不这样做的话，服务器是无法通过它自己的私钥解密出 AES 密钥的；（后记于 2018-01-30 10:45AM）

# PKI

PKI 的核心职责就是通过一些列的措施保证公钥的合法性，那么它是如何保证公钥不被篡改，是合法的呢？

## 如何保证公钥不被篡改的

比如 Alice 通过 RSA 生成了公钥 𝐾𝑝Kp 和密钥 𝐾𝑠Ks，她需要与 Bob 进行网络通讯，而且她必须通过网络将公钥 𝐾𝑝Kp 发送给 Bob；那么 PKI 是如何保证此公钥 𝐾𝑝Kp 在传输过程中不被篡改的呢？

简而言之，PKI 的策略就是通过一个第三方机构 𝑀M，该机构 𝑀M 也有自己的公钥 𝐾𝑝(𝑚)Kp(m) 和密钥 𝐾𝑠(𝑚)Ks(m)；

**首先通过该机构 𝑀M 进行如下的操作，**

1. Alice 将自己的公钥 𝐾𝑝Kp 提交给 𝑀M；
2. 𝑀M 使用签名算法，比如 MD5，对 Alice 的原生公钥 𝐾𝑝Kp 进行签名得到 𝑆𝑝Sp；
3. 然后再通过 𝐾𝑠(𝑚)Ks(m) 对 𝑆𝑝Sp 进行加密得到加密后的签名 𝐸𝑠𝑝Esp；
4. 最后将 𝐸𝑠𝑝Esp 颁发给 Alice；

**Alice 如何将公钥发送给 Bob？**

* Alice 将加密后的签名 𝐸𝑠𝑝Esp 和公钥 𝐾𝑝Kp 一起发送给 Bob；
  备注，其实签名 𝐸𝑠𝑝Esp 和公钥 𝐾𝑝Kp 被纳入一张`证书`(Certificate)中，Alice 发送给 Bob 的其实就是这么一张证书；但是，整整 X.509 证书包含的内容比这个复杂许多，不过概念上是等价的；这里，也就是证书的由来了。不仅要知其然，更要知其所以然..

**Bob 如何保证接收到的 Alice 的公钥没有被篡改过？**

Bob 拿到 Alice 的证书以后，做如下操作，(为了区分，这里将传输过程中 Alice 的公钥命名为 𝐾′𝑝Kp′)

1. 通过 𝑀M 的公钥 𝐾𝑝(𝑚)Kp(m) 对 𝐸𝑠𝑝Esp 进行解密，得到 𝑆𝑝Sp；
   注意，如果 𝐸𝑠𝑝Esp 被篡改了，这里是无法解密得到 𝑆𝑝Sp，也就是说，要保证解密成功，𝐸𝑠𝑝Esp 一定是没有被篡改过的；
2. 然后使用双方所约定好的签名算法，比如 MD5，对 𝐾′𝑝Kp′ 进行签名，得到 𝑆′𝑝Sp′
3. 最后比较 𝑆𝑝Sp 与 𝑆′𝑝Sp′ 是否相等，若相等，则表示发送过程中的公钥 𝐾′𝑝Kp′ 没有被篡改过，否则，则可以断言，𝐾′𝑝Kp′ 被篡改过，于是整个通讯不安全、不可靠；

（备注，这里的机构 𝑀M 有个行业用语叫做 𝐶𝐴CA 既是 *Certification Authority*；）

等等，第三方机构 CA，在 Bob 端使用 CA 的公钥 𝐾𝑝(𝑐𝑎)Kp(ca) 对 𝐸𝑠𝑝Esp 进行解密，得到签名 𝑆𝑝Sp，那如果 CA 的公钥 𝐾𝑝(𝑐𝑎)Kp(ca) 在传输过程中也被篡改了呢？好问题，第三方机构 CA 自身如何保证其公钥的合法性？如果不能保证 CA 公钥的合法性，上述基于签名保证 Alice 公钥合法性的措施也就是是一纸空谈。

**那么，CA 又是如何保证自己的公钥的合法性的呢？**

这里将谈到的就是 KPI 的又一大特征，基础设施；

为了保证 CA 公钥的合法性，通常，CA 机构的公钥是随系统、浏览器等`预安装`到客户电脑上的，也就是说，你在装系统的时候，或者在安装浏览器的时候，CA 机构的公钥(一般是包含在 CA 证书中的) 被预装到了你的系统或者是浏览器中了；这样，黑客就没有办法通过网络拦截的办法去篡改 CA 机构的公钥了；这也就是`基础设施`命名的由来，既 *Infrastructure*；这样，也就保证了通过 CA 机构签名的证书即使是在不安全的网络环境中传播，依然是`可靠并且是有效的`；

## 概念流程图

真实过程中，用户是通过 𝐶𝑆𝑅CSR 证书向 𝐶𝐴CA 机构申请 𝑋.509X.509 证书，申请过程中会包含申请者的地址，单位，邮箱等信息，比下图要复杂；这里，主要是化繁为简，通过如下的概念图，将其核心逻辑阐述清楚，下面的逻辑流程图一一对应[如何保证公钥不被篡改的](https://www.shangyang.me/2017/05/25/encrypt-rsa-pki/#如何保证公钥不被篡改的)中所描述的步骤；

![img](https://www.shangyang.me/2017/05/25/encrypt-rsa-pki/sequence.png)

注意两个关键步骤

* *Step 6.1.1*: 断定签名 𝐸𝑠𝑝Esp 被篡改过；
  要能够断定这种情况，保证 𝐸𝑠𝑝Esp 一定没有被篡改过，需要公钥基础设施(既 PKI )的保证，保证签名颁发机构 𝑀M 的公钥是无法被篡改的
* *Step 8.1*: 𝑆𝑝!=𝑆′𝑝Sp!=Sp′，则可以断定公钥是被篡改过得；
  这里面更深层次的原因是，通过公钥认证机构得到的签名 𝑆𝑝Sp 与 Bob 自己得到的公钥签名 𝑆𝑝Sp 不匹配，则可断定，Alice 的公钥 𝐾𝑝Kp 被篡改过；

## KPI 包含哪些元素

在章节[如何保证公钥不被篡改的](https://www.shangyang.me/2017/05/25/encrypt-rsa-pki/#如何保证公钥不被篡改的)中，作者用最通俗易懂的言语描述了 KPI 公钥基础设施是如何保证公钥的合法性的，以及其重要性；不过，在官方定义中，严格的定义了如下的角色，

1. CA (Certificate Authority)，证书认证机构
   对公钥、用户身份信息、域名等信息进行签名，生成相关的电子证书；并将电子证书颁发给申请用户；
   另外还会通过其专有的协议来判断证书是否有效（是否超过使用有效期）,如果证书失效，将会生成证书回收列表既是 certificate revocation list ，该部分内容涉及到 Authority revocation lists 的相关内容，从 https://en.wikipedia.org/wiki/Certificate_authority 摘要其核心内容如下，

   > An authority revocation list (ARL) is a form of certificate revocation list (CRL) containing certificates issued to certificate authorities, contrary to CRLs which contain revoked end-entity certificates.

   https://tools.ietf.org/html/rfc5280#section-4.1.2.6 标注中也描述了有关 CRL 协议和标准；

2. RA (Registration Authority)，注册认证机构
   看下 https://en.wikipedia.org/wiki/Public_key_infrastructure 中的描述，

   > A registration authority which verifies the identity of entities requesting their digital certificates to be stored at the CA

   是用来验证证书申请机构的身份的；不过现在 CA 和 RA 并没有完全区分，往往两者表示同一个角色；
   用户需要像该机构提交一张 CSR 格式的证书(该证书的后缀名为 *.csr*)，既是 [certificate signing request](https://en.wikipedia.org/wiki/Certificate_signing_request) 来申请，里面需要填写诸如你的域名，公司名，部门名称，城市名，地区名，国家名，电子邮件等等证明你身份的信息；

从 #1 中，我们知道，CA 不但会对用户的身份信息以及公钥进行签名，而且会生成相应的电子证书来保存这些签名信息，最终将该证书颁发给用户；而事实上，该电子证书经过数个版本的变化，现在已经形成了一个事实标准，那就是 *X.509 证书*；作者会在后续的文章中对其进行详细的介绍；

### CA 机构

这里描述一下国际知名的 CA 机构有哪些，

如下内容可以从 https://en.wikipedia.org/wiki/Certificate_authority 获得

![img](https://www.shangyang.me/2017/05/25/encrypt-rsa-pki/famous-ca-list.png)
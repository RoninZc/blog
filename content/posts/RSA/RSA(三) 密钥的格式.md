---
title: "RSA(三) 密钥的格式"
date: 2021-03-03T13:43:25+08:00
draft: true
toc: false
images:
tags: 
  - RSA
---

# 前言

本篇博文重点描述密钥的种种格式；

备注，本文是作者的原创作品，转载请注明出处。

> 转载至`https://www.shangyang.me/2017/05/24/encrypt-rsa-keyformat/`

# PEM 和 DER

首先我们来看看什么是 PEM 和 DER

## 什么是 DER 格式

DER 是密钥的二进制表述格式；

http://fileformats.archiveteam.org/wiki/DER

> `Distinguished Encoding Rules (DER)` is a *binary serialization* of ASN.1 format. It is often used for cryptographic data such as certificates, but has other uses.

很明显，DER 就是 ASN.1 的二进制格式；

## 什么是 PEM 格式

PEM 格式既是对 DER 编码转码为 Base64 字符格式；通过解码，将会还原为 DER 格式；

http://fileformats.archiveteam.org/wiki/PEM

> A PEM file is plain text. It contain one or more objects, such as certificates or keys, which may not all be the same type. Each object is delimited by lines similar to “—–BEGIN …—–” and “—–END …—–”. Data that is not between such lines is ignored, and is sometimes used for comments, or for a human-readable dump of the encoded data.

Following the “BEGIN” and “END” keywords is a name (such as “CERTIFICATE”) that can be used as an identifier for the type of object.

> The data between the delimiter lines starts with an optional email-like header section, followed by base64-encoded payload data. After decoding, the payload data is in DER format.

总体而言，PEM 是明文格式，可以包含证书或者是密钥；其内容通常是以类似 “—–BEGIN …—–” 开头 “—–END …—–” 为结尾的这样的格式进行展示的；后续内容也描述到，PEM 格式的内容是 Base64 格式；通过解码，转换为 DER 格式，也就是说，PEM 是建立在 DER 编码之上的；

## 总结

DER 实际上就是密钥的最原始的二进制格式；而 PEM 是对 DER 的 Base64 的编码，PEM 解码后得到的就是 DER 编码格式；

# 格式

由于 DER 是二进制格式，不便于阅读和理解，一般而言，密钥都是通过 PEM 的格式进行存储的，所以，这部分内容主要是梳理出公钥和密钥以 PEM 编码存储的格式；

## 公钥 PEM 格式

### PKCS #1

PKCS #1 标准是专门为 RSA 密钥进行定义的，其对应的 PEM 文件格式如下，

```
-----BEGIN RSA PUBLIC KEY-----
BASE64 ENCODED DATA
-----END RSA PUBLIC KEY-----
```

上面的内容 *BASE64 ENCODED DATA* 指的就是 ANS.1 的 DER 的 Base64 编码，其内容类似于

```
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQChHmaw+WUhWrStdxWBcAR39i2e  
3yz+vfLiDALeTpWIH1jKiYtvw4nMg6453pXAJSvPn7mKaiGiC3USIt8qTL4eCPi9  
yNRDpZ1JRHI8M87VYB4c9KMk6IuVFiYyZ4MBTP87t89yeL9EOrAD0eFgi5fPx3g8  
b9QrmnyPhMVjP7ct+wIDAQAB
```



上述内容翻译成 ASN.1 的格式为，此部分内容参考 [RSA public key syntax 公钥语法](https://www.shangyang.me/2017/05/21/encrypt-rsa-pkcs/#RSA-public-key-syntax-公钥语法) 小节内容；

```
RSAPublicKey ::= SEQUENCE {
    modulus           INTEGER,  -- n
    publicExponent    INTEGER   -- e
}
```



### PKCS #8

有前面的分析我们可以知道，PKCS#8 标准定义了一个密钥格式的通用方案，它不仅仅为 RSA 所使用，同样也可以被其它密钥所使用；具体分析参考 [RFC5208 Private-Key Information Syntax Specification](https://www.shangyang.me/2017/05/21/encrypt-rsa-pkcs/#RFC5208-Private-Key-Information-Syntax-Specification)

其所对应的 PEM 格式定义如下，

```
-----BEGIN PUBLIC KEY-----
BASE64 ENCODED DATA
-----END PUBLIC KEY-----
```



注意，这里就没有 RSA 字样了，因为 PKCS#8 是一个通用型的秘钥格式方案；其中的 *BASE64 ENCODED DATA* 所标注的内容为 PEM 格式中对 DER 原始二进制进行的 BASE64 编码；

所对应的 DER 原始二进制所表述的内容为

```
PublicKeyInfo ::= SEQUENCE {
  algorithm       AlgorithmIdentifier,
  PublicKey       BIT STRING
}

AlgorithmIdentifier ::= SEQUENCE {
  algorithm       OBJECT IDENTIFIER,
  parameters      ANY DEFINED BY algorithm OPTIONAL
}
```

**重要补充**

从这里可以看到，PKCS#8 虽然名字叫做 Private-Key Information Syntax Specification，但是实际上，可以看到，它同样可以用作 Public Key 的格式定义；而 PKCS#8 是站在 PKCS#7 CMS 的基础之上进行编码格式定义的；

## 私钥 PEM 格式

### PKCS #1

PKCS#1 是专门为 RSA 所涉及的，其对应的 PEM 格式如下

```
-----BEGIN RSA PRIVATE KEY-----
BASE64 ENCODED DATA
-----END RSA PRIVATE KEY-----
```

其中的 *BASE64 ENCODED DATA* 所标注的内容为 PEM 格式中对 DER 原始二进制进行的 BASE64 编码；

原始的 DER 格式结构，既是 ASN.1 的数据结构，此部分内容参考 [RSA private key syntax 私钥语法](https://www.shangyang.me/2017/05/21/encrypt-rsa-pkcs/#RSA-private-key-syntax-私钥语法) 小节内容；

```
RSAPrivateKey ::= SEQUENCE {
  version           Version,
  modulus           INTEGER,  -- n
  publicExponent    INTEGER,  -- e
  privateExponent   INTEGER,  -- d
  prime1            INTEGER,  -- p
  prime2            INTEGER,  -- q
  exponent1         INTEGER,  -- d mod (p-1)
  exponent2         INTEGER,  -- d mod (q-1)
  coefficient       INTEGER,  -- (inverse of q) mod p
  otherPrimeInfos   OtherPrimeInfos OPTIONAL
}
```



### PKCS #8

#### 未加密

其所对应的 PEM 格式定义如下，

```
-----BEGIN PRIVATE KEY-----
BASE64 ENCODED DATA
-----END PRIVATE KEY-----
```



注意，这里就没有 RSA 字样了，因为 PKCS#8 是一个通用型的秘钥格式方案；其中的 *BASE64 ENCODED DATA* 所标注的内容为 PEM 格式中对 DER 原始二进制进行的 BASE64 编码；

所对应的 DER 原始二进制所表述的内容为

```
PrivateKeyInfo ::= SEQUENCE {
  version         Version,
  algorithm       AlgorithmIdentifier,
  PrivateKey      BIT STRING
}

AlgorithmIdentifier ::= SEQUENCE {
  algorithm       OBJECT IDENTIFIER,
  parameters      ANY DEFINED BY algorithm OPTIONAL
}
```



#### 加密

由于私钥是非常私密的，所以在存储到时候往往需要对私钥的内容也进行加密，

PEM 格式

```
-----BEGIN ENCRYPTED PRIVATE KEY-----
BASE64 ENCODED DATA
-----END ENCRYPTED PRIVATE KEY-----
```



DER 格式，既是根据 ASN.1 标准所定义的格式

```
EncryptedPrivateKeyInfo ::= SEQUENCE {
  encryptionAlgorithm  EncryptionAlgorithmIdentifier,
  encryptedData        EncryptedData
}

EncryptionAlgorithmIdentifier ::= AlgorithmIdentifier

EncryptedData ::= OCTET STRING
```
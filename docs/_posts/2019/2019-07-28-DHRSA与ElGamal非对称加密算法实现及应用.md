---
layout: post
title: DH、RSA与ElGamal非对称加密算法实现及应用
date: 2019-07-28 21:52:00
author: 薛师兄
tags: 密码学
---
## 1.对称加密与非对称加密概述

关于对称加密与非对称加密的概念这里不再多说，感兴趣可以看下我之前的几篇文章，下面说一说两者的主要区别。

对称加密算法数据安全，密钥管理复杂，密钥传递过程复杂，存在密钥泄露问题。

非对称加密算法强度复杂、安全性依赖于算法与密钥。但是由于算法复杂，使得非对称算法加解密速度没有对称算法加解密的速度快。

对称密钥体制中只有一种密钥，并且是非公开的。如果要解密就得让对方知道密钥。所以保证其安全性就是保证密钥的安全。

非对称密钥体制有两种密钥，其中一个是公开的，这样就可以不需要像对称密码那样向对方传输密钥了。因此安全性就大了很多。

|            | 对称加密                                      | 非对称加密                              |
| ---------- | --------------------------------------------- | --------------------------------------- |
| 算法复杂度 | 弱                                            | 强                                      |
| 加解密速度 | 快                                            | 慢                                      |
| 安全性     | 低                                            | 高                                      |
| 常见算法   | DES、3DES、Blowfish、IDEA、RC4、RC5、RC6、AES | RSA、DSA、ECC、Diffie-Hellman、El Gamal |

## 2.DH算法实现过程及相关类详解

Diffie-Hellman算法(D-H算法)，密钥一致协议。是由公开密钥密码体制的奠基人Diffie和Hellman所提出的一种思想。简单的说就是允许两名用户在公开媒体上交换信息以生成"一致"的、可以共享的密钥。换句话说，就是由甲方产出一对密钥（公钥、私钥），乙方依照甲方公钥产生乙方密钥对（公钥、私钥）。以此为基线，作为数据传输保密基础，同时双方使用同一种对称加密算法构建本地密钥（SecretKey）对数据加密。这样，在互通了本地密钥（SecretKey）算法后，甲乙双方公开自己的公钥，使用对方的公钥和刚才产生的私钥加密数据，同时可以使用对方的公钥和自己的私钥对数据解密。不单单是甲乙双方两方，可以扩展为多方共享数据通讯，这样就完成了网络交互数据的安全通讯！该算法源于中国的同余定理——中国馀数定理。

**流程分析：** 

1.甲方构建密钥对儿，将公钥公布给乙方，将私钥保留；双方约定数据加密算法；乙方通过甲方公钥构建密钥对儿，将公钥公布给甲方，将私钥保留。 

2.甲方使用私钥、乙方公钥、约定数据加密算法构建本地密钥，然后通过本地密钥加密数据，发送给乙方加密后的数据；乙方使用私钥、甲方公钥、约定数据加密算法构建本地密钥，然后通过本地密钥对数据解密。 

3.乙方使用私钥、甲方公钥、约定数据加密算法构建本地密钥，然后通过本地密钥加密数据，发送给甲方加密后的数据；甲方使用私钥、乙方公钥、约定数据加密算法构建本地密钥，然后通过本地密钥对数据解密。 

Java代码：

```java
import javax.crypto.Cipher;
import javax.crypto.KeyAgreement;
import javax.crypto.SecretKey;
import javax.crypto.interfaces.DHPublicKey;
import javax.crypto.spec.DHParameterSpec;
import java.security.*;
import java.security.spec.X509EncodedKeySpec;
import java.util.Base64;
import java.util.Objects;

public class DH {
    private static final String src = "dh test";

    public static void main(String[] args) {
        jdkDH();
    }

    // jdk实现：
    public static void jdkDH() {
        try {
            // 1.发送方初始化密钥，将公钥给接收方
            KeyPairGenerator senderKeyPairGenerator = KeyPairGenerator.getInstance("DH");
            senderKeyPairGenerator.initialize(512);
            KeyPair senderKeyPair = senderKeyPairGenerator.generateKeyPair();
            // 发送方公钥，发送给接收方（通过网络或文件的形式）
            PublicKey senderPublicKey = senderKeyPair.getPublic(); // 公钥
            PrivateKey senderPrivateKey = senderKeyPair.getPrivate(); // 私钥
            
            // 接收方还原发送方公钥
            KeyFactory receiverKeyFactory = KeyFactory.getInstance("DH");
            X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(senderPublicKey.getEncoded());
            senderPublicKey = receiverKeyFactory.generatePublic(x509EncodedKeySpec);
            
            // 2.接收方通过发送方的公钥构建密钥，将公钥给发送方
            DHParameterSpec dhParameterSpec = ((DHPublicKey) senderPublicKey).getParams();
            KeyPairGenerator receiverKeyPairGenerator = KeyPairGenerator.getInstance("DH");
            receiverKeyPairGenerator.initialize(dhParameterSpec);
            KeyPair receiverKeypair = receiverKeyPairGenerator.generateKeyPair();
            PrivateKey receiverPrivateKey = receiverKeypair.getPrivate(); // 私钥
            PublicKey receiverPublicKey = receiverKeypair.getPublic(); // 公钥

            // 3.接收方使用自己的私钥和发送方的公钥构建本地密钥
            KeyAgreement receiverKeyAgreement = KeyAgreement.getInstance("DH");
            receiverKeyAgreement.init(receiverPrivateKey);
            receiverKeyAgreement.doPhase(senderPublicKey, true);
            SecretKey receiverDesKey = receiverKeyAgreement.generateSecret("DES");
            
            // 发送方还原接收方公钥
            KeyFactory senderKeyFactory = KeyFactory.getInstance("DH");
            x509EncodedKeySpec = new X509EncodedKeySpec(receiverPublicKey.getEncoded());
            receiverPublicKey = senderKeyFactory.generatePublic(x509EncodedKeySpec);
            
            // 4.发送方使用自己的私钥和接收方的公钥构建本地密钥
            KeyAgreement senderKeyAgreement = KeyAgreement.getInstance("DH");
            senderKeyAgreement.init(senderPrivateKey);
            senderKeyAgreement.doPhase(receiverPublicKey, true);
            SecretKey senderDesKey = senderKeyAgreement.generateSecret("DES");

            if (Objects.equals(receiverDesKey, senderDesKey)) {
                System.out.println("双方密钥相同");
            }

            // 5.发送方使用本地密钥加密
            Cipher cipher = Cipher.getInstance("DES");
            cipher.init(Cipher.ENCRYPT_MODE, senderDesKey);
            byte[] result = cipher.doFinal(src.getBytes());
            System.out.println("bc dh encrypt:" + Base64.getEncoder().encodeToString(result));

            // 6.接收方使用本地密钥解密
            cipher.init(Cipher.DECRYPT_MODE, receiverDesKey);
            result = cipher.doFinal(result);
            System.out.println("bc dh decrypt:" + new String(result));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

注意：因为JDK的版本问题，如果遇到异常`java.security.NoSuchAlgorithmException: Unsupported secret key algorithm: DES`，可以在运行的时候追加JVM参数`-Djdk.crypto.KeyAgreement.legacyKDF=true`

## 3.RSA算法实现及应用

RSA是目前最有影响力的公钥加密算法，它能够抵抗到目前为止已知的绝大多数密码攻击，已被ISO推荐为公钥数据加密标准。

RSA算法支持公钥加密、私钥解密以及私钥加密、公钥解密。

RSA算法既可以用于加密也可用于数字签名。

![](./20190728DHRSA与ElGamal非对称加密算法实现及应用/19153274.png)

Java代码：

```java
import javax.crypto.Cipher;
import java.security.*;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.Base64;

public class RSA {
    public static final String src = "rsa test";

    public static void main(String[] args) {
        jdkRSA();
    }

    // jdk实现：
    public static void jdkRSA() {
        try {
            // 1.生成公钥和私钥
            KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
            keyPairGenerator.initialize(512);
            KeyPair keyPair = keyPairGenerator.generateKeyPair();
            RSAPublicKey rsaPublicKey = (RSAPublicKey) keyPair.getPublic();
            RSAPrivateKey rsaPrivateKey = (RSAPrivateKey) keyPair.getPrivate();
            System.out.println("Public Key:" + Base64.getEncoder().encodeToString(rsaPublicKey.getEncoded()));
            System.out.println("Private Key:" + Base64.getEncoder().encodeToString(rsaPrivateKey.getEncoded()));

            // 2.私钥加密、公钥解密 ---- 加密
            //PKCS8EncodedKeySpec类表示私钥的ASN.1编码。
            PKCS8EncodedKeySpec pkcs8EncodedKeySpec = new PKCS8EncodedKeySpec(rsaPrivateKey.getEncoded());
            KeyFactory keyFactory = KeyFactory.getInstance("RSA");
            PrivateKey privateKey = keyFactory.generatePrivate(pkcs8EncodedKeySpec);

            Cipher cipher = Cipher.getInstance("RSA");
            cipher.init(Cipher.ENCRYPT_MODE, privateKey);
            byte[] result = cipher.doFinal(src.getBytes());
            System.out.println("私钥加密、公钥解密 ---- 加密:" + Base64.getEncoder().encodeToString(result));

            // 3.私钥加密、公钥解密 ---- 解密
            //X509EncodedKeySpec类表示根据ASN.1类型SubjectPublicKeyInfo编码的公钥的ASN.1编码。
            X509EncodedKeySpec x509EncodedKeySpec = new X509EncodedKeySpec(rsaPublicKey.getEncoded());
            keyFactory = KeyFactory.getInstance("RSA");
            PublicKey publicKey = keyFactory.generatePublic(x509EncodedKeySpec);

            cipher = Cipher.getInstance("RSA");
            cipher.init(Cipher.DECRYPT_MODE, publicKey);
            result = cipher.doFinal(result);
            System.out.println("私钥加密、公钥解密 ---- 解密:" + new String(result));

            // 4.公钥加密、私钥解密 ---- 加密
            //X509EncodedKeySpec类表示根据ASN.1类型SubjectPublicKeyInfo编码的公钥的ASN.1编码。
            X509EncodedKeySpec x509EncodedKeySpec2 = new X509EncodedKeySpec(rsaPublicKey.getEncoded());
            KeyFactory keyFactory2 = KeyFactory.getInstance("RSA");
            PublicKey publicKey2 = keyFactory2.generatePublic(x509EncodedKeySpec2);

            Cipher cipher2 = Cipher.getInstance("RSA");
            cipher2.init(Cipher.ENCRYPT_MODE, publicKey2);
            byte[] result2 = cipher2.doFinal(src.getBytes());
            System.out.println("公钥加密、私钥解密 ---- 加密:" + Base64.getEncoder().encodeToString(result2));

            // 5.私钥解密、公钥加密 ---- 解密
            //PKCS8EncodedKeySpec类表示私钥的ASN.1编码。
            PKCS8EncodedKeySpec pkcs8EncodedKeySpec5 = new PKCS8EncodedKeySpec(rsaPrivateKey.getEncoded());
            KeyFactory keyFactory5 = KeyFactory.getInstance("RSA");
            PrivateKey privateKey5 = keyFactory5.generatePrivate(pkcs8EncodedKeySpec5);
            Cipher cipher5 = Cipher.getInstance("RSA");
            cipher5.init(Cipher.DECRYPT_MODE, privateKey5);
            byte[] result5 = cipher5.doFinal(result2);
            System.out.println("公钥加密、私钥解密 ---- 解密:" + new String(result5));
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```

图解流程：

![](./20190728DHRSA与ElGamal非对称加密算法实现及应用/90462684.png)

## 4.非对称加密算法ElGamal

在密码学中，ElGamal加密算法是一个基于迪菲-赫尔曼密钥交换的非对称加密算法。它在1985年由塔希尔·盖莫尔提出。GnuPG和PGP等很多密码学系统中都应用到了ElGamal算法。

ElGamal加密系统通常应用在混合加密系统中。例如：用对称加密体制来加密消息，然后利用ElGamal加密算法传递密钥。这是因为在同等安全等级下，ElGamal加密算法作为一种非对称密码学系统，通常比对称加密体制要慢。对称加密算法的密钥和要传递的消息相比通常要短得多，所以相比之下使用ElGamal加密密钥然后用对称加密来加密任意长度的消息，这样要更快一些。

ElGamal算法只提供了公钥加密，私钥解密形式，Jdk中没有实现，Bouncy Castle中对其进行了实现。

导入Bouncy Castle依赖：

```xml
<dependency>
     <groupId>org.bouncycastle</groupId>
     <artifactId>bcprov-jdk15</artifactId>
     <version>1.46</version>
</dependency>
```

Java代码：

```java
import org.bouncycastle.jce.provider.BouncyCastleProvider;

import javax.crypto.Cipher;
import javax.crypto.spec.DHParameterSpec;
import java.security.*;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.Base64;

public class ElGamal {
    //非对称密钥算法
    public static final String EL_GAMAL = "ElGamal";
    /**
     * 密钥长度，DH算法的默认密钥长度是1024
     * 密钥长度必须是8的倍数，在160到16384位之间
     */
    private static final int KEY_SIZE = 256;

    public static void main(String[] args) throws Exception {
        System.out.println("=============接收方构建密钥对=============");
        // 加入对BouncyCastle支持
        Security.addProvider(new BouncyCastleProvider());
        AlgorithmParameterGenerator apg = AlgorithmParameterGenerator.getInstance(EL_GAMAL);
        //初始化参数生成器
        apg.init(KEY_SIZE);
        //生成算法参数
        AlgorithmParameters params = apg.generateParameters();
        //构建参数材料
        DHParameterSpec elParams = params.getParameterSpec(DHParameterSpec.class);
        //实例化密钥生成器
        KeyPairGenerator kpg = KeyPairGenerator.getInstance(EL_GAMAL);
        //初始化密钥对生成器
        kpg.initialize(elParams, new SecureRandom());
        KeyPair keyPair = kpg.generateKeyPair();
        //公钥
        PublicKey publicKey = keyPair.getPublic();
        //私钥
        PrivateKey privateKey = keyPair.getPrivate();
        System.out.println("公钥：" + Base64.getEncoder().encodeToString(publicKey.getEncoded()));
        System.out.println("私钥：" + Base64.getEncoder().encodeToString(privateKey.getEncoded()));
        System.out.println("=============密钥对构造完毕，接收方将公钥公布给发送方=============");
        String str = "ElGamal密码交换算法";
        System.out.println("原文：" + str);
        System.out.println("=============发送方还原接收方公钥，并使用公钥对数据进行加密=============");
        //还原公钥
        KeyFactory keyFactory = KeyFactory.getInstance(EL_GAMAL);
        X509EncodedKeySpec x509KeySpec = new X509EncodedKeySpec(publicKey.getEncoded());
        publicKey = keyFactory.generatePublic(x509KeySpec);
        System.out.println("公钥：" + Base64.getEncoder().encodeToString(publicKey.getEncoded()));
        //数据加密
        Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
        cipher.init(Cipher.ENCRYPT_MODE, publicKey);
        byte[] bytes = cipher.doFinal(str.getBytes());
        System.out.println("加密后的数据：" + Base64.getEncoder().encodeToString(bytes));
        System.out.println("=============接收方使用私钥对数据进行解密===========");
        //还原私钥
        PKCS8EncodedKeySpec pkcs8KeySpec = new PKCS8EncodedKeySpec(privateKey.getEncoded());
        keyFactory = KeyFactory.getInstance(EL_GAMAL);
        privateKey = keyFactory.generatePrivate(pkcs8KeySpec);
        //数据解密
        cipher = Cipher.getInstance(keyFactory.getAlgorithm());
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
        byte[] bytes1 = cipher.doFinal(bytes);
        System.out.println("解密后的数据：" + new String(bytes1));
    }
}
```

图解流程：

![](./20190728DHRSA与ElGamal非对称加密算法实现及应用/77964127.png)
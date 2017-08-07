---
title: Android数据加密
date: 2017-05-02 09:47:03
categories: Android
tags: 加密
---
在Android开发中，客户端与服务器进行数据交互时，无可避免的会涉及到诸如用户信息、密码等敏感信息。因此，对重要数据的加密就显得非常重要。

<!-- more -->

## 常用加密方式

### MD5

MD5是一种被广泛使用的密码散列函数。可以将不定长度的信息，加密成固定长度的散列值（128位，一般表示为32位十六进制数字）。

MD5是**不可逆的加密算法**，常用于只需加密无需解密的数据上，比如用户密码。也常用来保证数据的完整性，因为数据更改后，其加密的MD5也会随着改变，对比前后的MD5值可确定数据是否完整或被更改。

MD5已经被证实无法防止碰撞，可以被加以破解。可以通过多次加密、加盐的方式增大破解难度。但对于高度安全性的数据，一般建议改用其他算法，比如SHA-2、SHA-3。因其稳定快速的特点，MD5仍广泛应用于普通数据的错误检查领域。

### RSA

RSA加密算法是以算法的三位发明者名字首字母命名，是一种**非对称加密算法**。到目前为止，世界上还没有任何可靠的攻击RSA算法的方式。

> 非对称加密算法：该算法需要两个密钥，公钥和私钥。公钥和私钥是一对，如果用公钥对数据进行加密，只有用对应的私钥才能解密；反之，如果用私钥对数据进行加密，只有用对应的公钥才能加密。简单来说，即“公钥加密，私钥解密；私钥加密，公钥解密”。

### AES

AES全称为Advanced Encryption Standard，即高级加密标准，是一种**区块加密**标准。这个算法已经替代原来的DES（Data Encryption Standard），被多方分析并广为使用。

> 区块加密是一种**对称加密算法**。这类算法在加密和解密时使用相同的密钥。

## 加密使用

### 使用RSA

首先，需要生成一对公钥和私钥，生成方法很多，可以网上搜索，下面给出一种示例。

``` java
//生成密钥对  
private KeyPair generateKeyPair() throws Exception {
    try {
        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA",
                new org.bouncycastle.jce.provider.BouncyCastleProvider());
        // 这个值关系到块加密的大小，可以更改，但是不要太大，否则效率会低
        final int KEY_SIZE = 1024; //1024-bit密钥----目前较流行
        keyPairGen.initialize(KEY_SIZE, new SecureRandom());
        KeyPair keyPair = keyPairGen.genKeyPair();
        return keyPair;
    } catch (Exception e) {
        throw new Exception(e.getMessage());
    }

}
 //生成公钥
private RSAPublicKey generateRSAPublicKey(byte[] modulus,
                                          byte[] publicExponent) throws Exception {

    KeyFactory keyFac = null;
    try {
        keyFac = KeyFactory.getInstance("RSA",
                new org.bouncycastle.jce.provider.BouncyCastleProvider());
    } catch (NoSuchAlgorithmException ex) {
        throw new Exception(ex.getMessage());
    }
    RSAPublicKeySpec pubKeySpec = new RSAPublicKeySpec(new BigInteger(
            modulus), new BigInteger(publicExponent));
    try {
        return (RSAPublicKey) keyFac.generatePublic(pubKeySpec);
    } catch (InvalidKeySpecException ex) {
        throw new Exception(ex.getMessage());
    }

}
 //生成私钥
private RSAPrivateKey generateRSAPrivateKey(byte[] modulus,
                                            byte[] privateExponent) throws Exception {
    KeyFactory keyFac = null;
    try {
        keyFac = KeyFactory.getInstance("RSA",
                new org.bouncycastle.jce.provider.BouncyCastleProvider());
    } catch (NoSuchAlgorithmException ex) {
        throw new Exception(ex.getMessage());
    }
    RSAPrivateKeySpec priKeySpec = new RSAPrivateKeySpec(new BigInteger(
            modulus), new BigInteger(privateExponent));
    try {
        return (RSAPrivateKey) keyFac.generatePrivate(priKeySpec);
    } catch (InvalidKeySpecException ex) {
        throw new Exception(ex.getMessage());
    }
}
```

生成之后，公钥存放在客户端，可以被别人知道，而私钥则要放在服务端，不能泄露。如果公司有人事变动，避免离职人员泄露私钥，可以重新生成密钥。

单独使用RSA加密，只适用于**客户端传输给服务器的数据重要，服务器返回的数据不重要**的情况。比如传递用户的手机号等个人信息，需要在客户端使用公钥对数据进行加密，传递到服务器后使用私钥进行解密。而服务器返回给客户端的数据，主要是一些状态码以及提示信息，不需要加密，再直接返回即可。

### 使用RSA+AES

然而还有一些情况，是属于**客户端传输给服务器的数据重要，服务器返回的数据也重要**。比如用户进行登录时，传递的用户名密码需要加密，返回的令牌Token也需要加密。由于公钥是公开的，只使用RSA加密没办法对返回数据进行加密。这时候，就需要再结合AES加密了。

使用这种加密，主要有以下几个步骤（如果返回数据不重要，可以省略6，7步骤）。

1. 客户端随机产生AES密钥。

2. 客户端对要传输的重要数据进行AES加密。

3. 客户端对AES的密钥，使用RSA的公钥加密。

4. 服务端对拿到的AES密钥用RSA的私钥进行解密，拿到AES密钥原文。

5. 服务端对加密后的重要信息进行AES解密，得到原始内容。

6. 对返回数据，使用得到的AES密钥加密，返回给客户端。

7. 客户端用之前随机产生的AES密钥，对返回结果进行解密。

在这种加密方式中，AES的密钥是在传输过程中才随机生成的，而且传输时通过RSA加密，所以无法被获取到，能有效地保障数据的安全。


title: Java加解密工具
author: 赵旭东
tags:
  - 3des
  - ''
categories:
  - 技术
date: 2018-06-22 16:23:00
---
先写一种加解密工具。后续继续添加其他方式！

<!--more-->

# 3DES加解密

```java

package com.github.pig.common.util;


import org.apache.commons.lang.StringUtils;

import javax.crypto.Cipher;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESedeKeySpec;
import javax.crypto.spec.IvParameterSpec;
import java.security.InvalidKeyException;
import java.security.Key;
import java.util.Base64;


public class EnDecryptUtil {

    // 密钥
    private final static String secretKey = "zzzz";
    // 向量
    private final static String iv = "xxxx";
    // 加解密统一使用的编码方式
    private final static String encoding = "utf-8";

    /**
     * 3DES加密
     *
     * @param plainText 普通文本
     * @return
     * @throws Exception
     */
    public static String d3esEncode(String plainText) throws Exception {
        if(plainText == null){
            return plainText;
        }
        Key deskey = null;
        byte[] encryptData = null;
        DESedeKeySpec spec = new DESedeKeySpec(secretKey.getBytes());
        SecretKeyFactory keyfactory = SecretKeyFactory.getInstance("desede");
        deskey = keyfactory.generateSecret(spec);

        Cipher cipher = Cipher.getInstance("desede/CBC/PKCS5Padding");
        IvParameterSpec ips = new IvParameterSpec(iv.getBytes());
        cipher.init(Cipher.ENCRYPT_MODE, deskey, ips);
        encryptData = cipher.doFinal(plainText.getBytes(encoding));
        return Base64.getEncoder().encodeToString(encryptData);

    }

    /**
     * 3DES解密
     *
     * @param encryptText 加密文本
     * @return
     * @throws Exception
     */
    public static String d3esDecode(String encryptText) throws Exception {
        if(StringUtils.isBlank(encryptText)){
            return encryptText;
        }
        Key deskey = null;
        byte[] decryptData = null;
        String result = "";
        if (null != encryptText && !"".equals(encryptText)) {
            DESedeKeySpec spec = new DESedeKeySpec(secretKey.getBytes());
            SecretKeyFactory keyfactory = SecretKeyFactory.getInstance("desede");
            deskey = keyfactory.generateSecret(spec);
            Cipher cipher = Cipher.getInstance("desede/CBC/PKCS5Padding");
            IvParameterSpec ips = new IvParameterSpec(iv.getBytes());
            cipher.init(Cipher.DECRYPT_MODE, deskey, ips);

            decryptData = cipher.doFinal(Base64.getDecoder().decode(encryptText));
            result = new String(decryptData, encoding);
        }
        return result;
    }

    public static void main(String[] args) throws Exception {
        String s = new String("123456");

        System.out.println(EnDecryptUtil.d3esEncode(s));
        System.out.println(EnDecryptUtil.d3esDecode(EnDecryptUtil.d3esEncode(s)));
    }
}


```
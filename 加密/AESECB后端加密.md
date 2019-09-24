```java
package com.example.service.webhook.util;

import org.apache.commons.codec.binary.Base64;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.security.Provider;
import java.security.SecureRandom;
import java.security.Security;
import java.util.Arrays;

/**
 * Created by jy on 2017/8/15.
 */
public class AESECB {
    
    /***
     * 默认向量常量
     **/
    public static final String IV = "2015030120123456";
    private final static Logger logger = LoggerFactory.getLogger(AESECB.class);
    
    /**
     * 加密 128位
     *
     * @param content 需要加密的内容
     * @param pkey    加密密码
     * @return
     */
    public static byte[] aesEncrypt(String content, String pkey) throws Exception {
        SecretKeySpec key = new SecretKeySpec(pkey.getBytes(), "AES");
        Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");// "算法/模式/补码方式"
        //        IvParameterSpec iv = new IvParameterSpec(IV.getBytes());
        cipher.init(Cipher.ENCRYPT_MODE, key);
        byte[] encrypted = cipher.doFinal(content.getBytes("UTF-8"));
        return encrypted; // 加密
    }
    
    /**
     * 获得密钥
     *
     * @param secretKey
     * @return
     * @throws Exception
     */
    private static SecretKey generateKey(String secretKey) throws Exception {
        //防止linux下 随机生成key
        Provider p = Security.getProvider("SUN");
        SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG", p);
        secureRandom.setSeed(secretKey.getBytes());
        KeyGenerator kg = KeyGenerator.getInstance("AES");
        kg.init(secureRandom);
        // 生成密钥
        return kg.generateKey();
    }
    
    /**
     * @param content
     * @param pkey    长度为16个字符,128位
     * @param @return 设定文件
     * @return String    返回类型
     * @throws
     * @Title: aesEncryptStr
     * @Description: aes对称加密
     */
    public static String aesEncryptStr(String content, String pkey) throws Exception {
        byte[] aesEncrypt = aesEncrypt(content, pkey);
        logger.debug("aesEncrypt String:" + Arrays.toString(aesEncrypt));
        String base64EncodeStr = Base64.encodeBase64String(aesEncrypt);
        logger.debug("aesEncrypt base64EncodeStr:" + base64EncodeStr);
        return base64EncodeStr;
    }
    
    /**
     * @param content base64处理过的字符串
     * @param pkey
     * @return String    返回类型
     * @throws Exception
     * @throws
     * @Title: aesDecodeStr
     * @Description: 解密 失败将返回NULL
     */
    public static String aesDecodeStr(String content, String pkey) throws Exception {
        
        byte[] base64DecodeStr = Base64.decodeBase64(content);
        logger.debug("base64DecodeStr String:" + Arrays.toString(base64DecodeStr));
        byte[] aesDecode = aesDecode(base64DecodeStr, pkey);
        if (aesDecode == null) {
            return null;
        }
        String result = new String(aesDecode, "UTF-8");
        logger.debug("aesDecode result:" + result);
        return result;
    }
    
    /**
     * 解密 128位
     *
     * @param content 待解密内容
     * @param pkey    解密密钥
     * @return
     */
    public static byte[] aesDecode(byte[] content, String pkey) throws Exception {
        SecretKeySpec key = new SecretKeySpec(pkey.getBytes(), "AES");
        IvParameterSpec iv = new IvParameterSpec(IV.getBytes("UTF-8"));
        Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");// 创建密码器
        cipher.init(Cipher.DECRYPT_MODE, key, iv);// 初始化
        byte[] result = cipher.doFinal(content);
        return result; // 解密
    }
    
    public static void main(String[] args) {
        try {
            System.out.println(AESECB.aesEncryptStr("{\"phone\":\"13500000000\",""\"name\":\"张三\",\"idcard\":\"330501198900000000\"}","00a4be26195d4856965c5293629b84b2"));
            //            System.out.println(AESECB.aesDecodeStr("lcbadiprucg1c1Fje0gGO88uZXW2vOmZyuxU8sZGAUJIJ2YoJwNnFBJ0DpYlwz2W/JVzFPhvkeRDSqZC2jnHOBEx/dNUHqdx+wh66GnZI40=","85685d2b6d6e4600a817953113a98268"));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
}


```


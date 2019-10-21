## **一、工具类**

### md5加密工具类

```java
public class MD5Utils {

    private static final String hexDigIts[] = {"0","1","2","3","4","5","6","7","8","9","a","b","c","d","e","f"};

    /**
     * MD5加密
     * @param origin 字符
     * @param charsetname 编码
     * @return
     */
    public static String MD5Encode(String origin, String charsetname){
        String resultString = null;
        try{
            resultString = new String(origin);
            MessageDigest md = MessageDigest.getInstance("MD5");
            if(null == charsetname || "".equals(charsetname)){
                resultString = byteArrayToHexString(md.digest(resultString.getBytes()));
            }else{
                resultString = byteArrayToHexString(md.digest(resultString.getBytes(charsetname)));
            }
        }catch (Exception e){
        }
        return resultString;
    }


    public static String byteArrayToHexString(byte b[]){
        StringBuffer resultSb = new StringBuffer();
        for(int i = 0; i < b.length; i++){
            resultSb.append(byteToHexString(b[i]));
        }
        return resultSb.toString();
    }

    public static String byteToHexString(byte b){
        int n = b;
        if(n < 0){
            n += 256;
        }
        int d1 = n / 16;
        int d2 = n % 16;
        return hexDigIts[d1] + hexDigIts[d2];
    }

}
```

### base64加密工具类

```java
public class Base64Util {

    // 字符串编码
    private static final String UTF_8 = "UTF-8";

    /**
     * 加密字符串
     * @param inputData
     * @return
     */
    public static String decodeData(String inputData) {
        try {
            if (null == inputData) {
                return null;
            }
            return new String(Base64.decodeBase64(inputData.getBytes(UTF_8)), UTF_8);
        } catch (UnsupportedEncodingException e) {
        }
        return null;
    }

    /**
     * 解密加密后的字符串
     * @param inputData
     * @return
     */
    public static String encodeData(String inputData) {
        try {
            if (null == inputData) {
                return null;
            }
            return new String(Base64.encodeBase64(inputData.getBytes(UTF_8)), UTF_8);
        } catch (UnsupportedEncodingException e) {
        }
        return null;
    }

    public static void main(String[] args) {
        System.out.println(Base64Util.encodeData("我是中文"));
        String enStr = Base64Util.encodeData("我是中文");
        System.out.println(Base64Util.decodeData(enStr));
    }
}
```

### Bcrypt工具类

```java
public class BcryptCipher {
  // generate salt seed
  private static final int SALT_SEED = 12;
  // the head fo salt
  private static final String SALT_STARTSWITH = "$2a$12";
  
  public static final String SALT_KEY = "salt";
  
  public static final String CIPHER_KEY = "cipher";
  
  /**
   * Bcrypt encryption algorithm method
   * @param encryptSource
   * need to encrypt the string
   * @return Map , two values in Map , salt and cipher
   */
  public static Map<String, String> Bcrypt(final String encryptSource) {
    String salt = BCrypt.gensalt(SALT_SEED);
    Map<String, String> bcryptResult = Bcrypt(salt, encryptSource);
    return bcryptResult;
  }
  /**
   *
   * @param salt encrypt salt, Must conform to the rules
   * @param encryptSource
   * @return
   */
  public static Map<String, String> Bcrypt(final String salt, final String encryptSource) {
    if (StringUtils.isBlank(encryptSource)) {
      throw new RuntimeException("Bcrypt encrypt input params can not be empty");
    }
    
    if (StringUtils.isBlank(salt) || salt.length() != 29) {
      throw new RuntimeException("Salt can't be empty and length must be to 29");
    }
    if (!salt.startsWith(SALT_STARTSWITH)) {
      throw new RuntimeException("Invalid salt version, salt version is $2a$12");
    }
    
    String cipher = BCrypt.hashpw(encryptSource, salt);
    Map<String, String> bcryptResult = new HashMap<String, String>();
    bcryptResult.put(SALT_KEY, salt);
    bcryptResult.put(CIPHER_KEY, cipher);
    return bcryptResult;
  }
  
}
```

## **二、加密测试**

### MD5加密测试

```java
/**
 * MD5加密
 */
public class MD5Test {
  public static void main(String[] args) {
    String string = "我是一句话";
    String byteArrayToHexString = MD5Utils.byteArrayToHexString(string.getBytes());
    System.out.println(byteArrayToHexString);//e68891e698afe4b880e58fa5e8af9d

  }
}
```

### base64加密测试

```java
/**
 * base64加密
 */
public class Bast64Tester {
  
  public static void main(String[] args) {
    String string = "我是一个字符串";
    String encodeData = Base64Util.encodeData(string); //加密
    String decodeData = Base64Util.decodeData(encodeData); //解密
    System.out.println(encodeData);//5oiR5piv5LiA5Liq5a2X56ym5Liy
    System.out.println(decodeData);//我是一个字符串

  }
}
```

### SHA加密测试

```java
/**
 * SHA加密
 */
public class ShaTest {
  
  public static void main(String[] args) {
    String string = "我是一句话";
    
    String sha256Crypt = Sha2Crypt.sha256Crypt(string.getBytes());
    System.out.println(sha256Crypt);//$5$AFoQTeyt$TiqmobvcQXjXaAQMYosAAO4KI8LfigZMGHzq.Dlp4NC

  }
}
```

### BCrypt加密测试

```java
/**
 * BCrypt加密
 */
public class BCryptTest {

  public static void main(String[] args) {
    
    String string = "我是一句话";
    Map<String, String> bcrypt = BcryptCipher.Bcrypt(string);
    System.out.println(bcrypt.keySet()); //[cipher, salt]
    
    System.out.println(bcrypt.get("cipher")); //$2a$12$ylb92Z84gqlrSfzIztlCV.dK0xNbw.pOv3UwXXA76llOsNRTJsE/.
    System.out.println(bcrypt.get("salt")); //$2a$12$ylb92Z84gqlrSfzIztlCV.
    
    Map<String, String> bcrypt2 = BcryptCipher.Bcrypt(bcrypt.get("salt"),string);
    System.out.println(bcrypt2.get("SALT_KEY")); //null
    System.out.println(bcrypt2.get("CIPHER_KEY")); //null
  }
}
```
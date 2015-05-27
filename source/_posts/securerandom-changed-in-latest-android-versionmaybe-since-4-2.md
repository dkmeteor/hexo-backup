title: SecureRandom changed in latest Android version(maybe since 4.2)
id: 133
categories:
  - android
date: 2014-11-11 02:13:15
tags:
---

<div style="color: #000000;"></div>

<div style="color: #000000;">

使用以前的自己写的一个AES加密工具类的时候发现跑不通了,调用会直接crash,并抛出
<pre class="lang:java decode:true ">11-11 09:05:16.747: W/System.err(13832): java.security.NoSuchAlgorithmException: SecureRandom AES implementation not found</pre>
&nbsp;

源代码如下:
<pre class="lang:java decode:true ">import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.security.Provider;
import java.security.SecureRandom;
import java.security.Security;
import java.util.Locale;

import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.KeyGenerator;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;

/**
* 
* @author Peter.Ding
* 
*/public class AESTools {
    private static AESTools instance = new AESTools();
    private String password = "peter.ding@augmentum.com";
    private KeyGenerator kgen;

    private AESTools() {
    }

    public static AESTools getInstance() {
        return instance;
    }

    public String encrypt(String content) {
        try {
            kgen = KeyGenerator.getInstance("AES");
            Provider p = Security.getProvider("BC");
            SecureRandom s = SecureRandom.getInstance("AES", p);

            //The right code 
            //Provider p = Security.getProvider("Crypto"); 
            //SecureRandom s = SecureRandom.getInstance("SHA1PRNG", p);

            s.setSeed(password.getBytes());
            kgen.init(128, s);
            SecretKey secretKey = kgen.generateKey();
            byte[] enCodeFormat = secretKey.getEncoded();
            SecretKeySpec key = new SecretKeySpec(enCodeFormat, "AES");
            Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
            byte[] byteContent = content.getBytes("utf-8");
            cipher.init(Cipher.ENCRYPT_MODE, key);
            byte[] result = cipher.doFinal(byteContent);
            return bytesToHexString(result);
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        } catch (IllegalBlockSizeException e) {
            e.printStackTrace();
        } catch (BadPaddingException e) {
            e.printStackTrace();
        } catch (InvalidKeyException e) {
            e.printStackTrace();
        }
        return null;
    }

    public String decrypt(String str) {
        byte[] content = hexStringToByte(str);
        try {
            kgen = KeyGenerator.getInstance("AES");
            Provider p = Security.getProvider("BC");
            SecureRandom s = SecureRandom.getInstance("AES", p);

            //The right code 
            //Provider p = Security.getProvider("Crypto"); 
            //SecureRandom s = SecureRandom.getInstance("SHA1PRNG", p);

            s.setSeed(password.getBytes());
            kgen.init(128, s);
            SecretKey secretKey = kgen.generateKey();
            byte[] enCodeFormat = secretKey.getEncoded();
            SecretKeySpec key = new SecretKeySpec(enCodeFormat, "AES");
            Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
            cipher.init(Cipher.DECRYPT_MODE, key);
            byte[] result = cipher.doFinal(content);
            System.out.println(bytesToHexString(result));
            return new String(result, "utf-8");
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
        } catch (InvalidKeyException e) {
            e.printStackTrace();
        } catch (IllegalBlockSizeException e) {
            e.printStackTrace();
        } catch (BadPaddingException e) {
            e.printStackTrace();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        return null;
    }

    public String bytesToHexString(byte[] src) {
        StringBuilder stringBuilder = new StringBuilder("");
        if (src == null || src.length &lt;= 0) {
            return null;
        }
        for (int i = 0; i &lt; src.length; i++) {
            int v = src[i] &amp; 0xFF;
            String hv = Integer.toHexString(v);
            if (hv.length() &lt; 2) {
                stringBuilder.append(0);
            }
            stringBuilder.append(hv);
        }
        return stringBuilder.toString().toUpperCase(Locale.ENGLISH);
    }

    public static byte[] hexStringToByte(String hex) {
        int len = (hex.length() / 2);
        byte[] result = new byte[len];
        char[] achar = hex.toCharArray();
        for (int i = 0; i &lt; len; i++) {
            int pos = i * 2;
            result[i] = (byte) (toByte(achar[pos]) &lt;&lt; 4 | toByte(achar[pos + 1]));
        }
        return result;
    }

    private static byte toByte(char c) {
        byte b = (byte) "0123456789ABCDEF".indexOf(c);
        return b;
    }

}</pre>
&nbsp;

以上代码是我12年的时候为 加密数据库写的,我马上意识到肯定是Android又修改了Security Provider,当时2.1 2.3适配把我坑了一次,4.0适配又把我坑了一次,没想到还要坑我第三次.....无语...
<pre class="lang:java decode:true ">Provider p = Security.getProvider("BC");
SecureRandom s = SecureRandom.getInstance("AES", p);</pre>
&nbsp;

报错的就是上面这行代码,果断复制粘贴百度

结果搜出来结果就是这个 [http://dkmeteor.iteye.com/blog/1394887](http://dkmeteor.iteye.com/blog/1394887)

仔细一看还是我2012年写的blog.... 算了....不看也知道肯定是 Provider和SecureRandom又改动了...

直接拉Provider列表

    Android's OpenSSL-backed security provider
    1ASN.1, DER, PkiPath, PKCS7
    BouncyCastle Security Provider v1.49
    HARMONY (SHA1 digest; SecureRandom; SHA1withDSA signature)
    Harmony JSSE Provider
    Android KeyStore security provider

[http://www.bouncycastle.org/](http://www.bouncycastle.org/) 翻了一下文档 `BouncyCastle` 是支持AES的,不知为何直接调用会抛NoSuchAlgorithmException 没看到关于Android内Provider支持的Algorithm具体列表.

试了下使用 Android OpenSSL (2.3以下版本不支持) , 一样会抛出该错误,最后想了想改成原始版本的默认值 SHA1PRNG和Crypto. Done. (2.3+版本默认Default Provider修改成了 Android OpenSSL)

* * *

后来又找到一篇 Security Enhancements in Jelly Bean 的文档

[http://android-developers.blogspot.com/2013/02/security-enhancements-in-jelly-bean.html](http://android-developers.blogspot.com/2013/02/security-enhancements-in-jelly-bean.html)

原文在墙外,关键部分我复制保存以下
> New implementation of SecureRandom
> 
> Android 4.2 includes a new default implementation of SecureRandom based on OpenSSL. In the older Bouncy Castle-based implementation, given a known seed, SecureRandom could technically (albeit incorrectly) be treated as a source of deterministic data. With the new OpenSSL-based implementation, this is no longer possible.
> 
> In general, the switch to the new SecureRandom implementation should be transparent to apps. However, if your app is relying on SecureRandom to generate deterministic data, such as keys for encrypting data, you may need to modify this area of your app. For example, if you have been using SecureRandom to retrieve keys for encrypting/decrypting content, you will need to find another means of doing that.
> 
> A recommended approach is to generate a truly random AES key upon first launch and store that key in internal storage. For more information, see the post "Using Cryptography to Store Credentials Safely".
还有另一篇关于 最佳实践的

[http://android-developers.blogspot.com/2013/02/using-cryptography-to-store-credentials.html](http://android-developers.blogspot.com/2013/02/using-cryptography-to-store-credentials.html)

&nbsp;
> The right way
> 
> A more reasonable approach is simply to generate a truly random AES key when an application is first launched:
> 
> public static SecretKey generateKey() throws NoSuchAlgorithmException {
> // Generate a 256-bit key
> final int outputKeyLength = 256;
> 
> SecureRandom secureRandom = new SecureRandom();
> // Do *not* seed secureRandom! Automatically seeded from system entropy.
> KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
> keyGenerator.init(outputKeyLength, secureRandom);
> SecretKey key = keyGenerator.generateKey();
> return key;
> }
> Note that the security of this approach relies on safeguarding the generated key, which is is predicated on the security of the internal storage. Leaving the target file unencrypted (but set to MODE_PRIVATE) would provide similar security.
> 
> Even more security
> If your app needs additional encryption, a recommended approach is to require a passphase or PIN to access your application. This passphrase could be fed into PBKDF2 to generate the encryption key. (PBKDF2 is a commonly used algorithm for deriving key material from a passphrase, using a technique known as "key stretching".) Android provides an implementation of this algorithm inside SecretKeyFactory as PBKDF2WithHmacSHA1:
> 
> public static SecretKey generateKey(char[] passphraseOrPin, byte[] salt) throws NoSuchAlgorithmException, InvalidKeySpecException {
> // Number of PBKDF2 hardening rounds to use. Larger values increase
> // computation time. You should select a value that causes computation
> // to take &gt;100ms.
> final int iterations = 1000;
> 
> // Generate a 256-bit key
> final int outputKeyLength = 256;
> 
> SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");
> KeySpec keySpec = new PBEKeySpec(passphraseOrPin, salt, iterations, outputKeyLength);
> SecretKey secretKey = secretKeyFactory.generateSecret(keySpec);
> return secretKey;
> }
> The salt should be a random string, again generated using SecureRandom and persisted on internal storage alongside any encrypted data. This is important to mitigate the risk of attackers using a rainbow table to precompute password hashes.
因为使用了固定的seed作为随机种子生成器(RNG)的参数,导致生成的随机数其实是固定的,所以官方不推荐这种用法,当然我类库里的seed其实就是 加密密码,如果不需要指定密码的话, 倒是可以用官方的推荐的这种方式.

</div>

&nbsp;
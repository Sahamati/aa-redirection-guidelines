# Java

Sample Java code for XOR and encryption of the **ecreq** and **ecres** fields.

```text
import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.PBEKeySpec;
import javax.crypto.spec.SecretKeySpec;
import java.security.SecureRandom;
import java.security.spec.KeySpec;
import java.util.Base64;

import static java.nio.charset.StandardCharsets.UTF_8;
import static java.util.Base64.getUrlDecoder;
import static java.util.Base64.getUrlEncoder;
import static javax.crypto.Cipher.DECRYPT_MODE;
import static javax.crypto.Cipher.ENCRYPT_MODE;

public class WebRedirection {
    private static final String AES_ALGO = "AES";
    private static final String KEY_ALGO = "PBKDF2WithHmacSHA256";
    private static final String AES_CBC_PKCS5 = "AES/CBC/PKCS5Padding";
    private static final String secretKey = "ac12ghd75kf75r";
    private static final String fi = "FIUID";

    public static String encrypt(String strToEncrypt, String salt) {
        final byte[] iv = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0};

        IvParameterSpec ivspec = new IvParameterSpec(iv);

        try {
            SecretKeyFactory factory = SecretKeyFactory.getInstance(KEY_ALGO);
            KeySpec spec = new PBEKeySpec(secretKey.toCharArray(), salt.getBytes(), 65536, 256);
            SecretKey tmp = factory.generateSecret(spec);
            SecretKeySpec secretKey = new SecretKeySpec(tmp.getEncoded(), AES_ALGO);
            Cipher cipher = Cipher.getInstance(AES_CBC_PKCS5);
            cipher.init(ENCRYPT_MODE, secretKey, ivspec);
            return getUrlEncoder().encodeToString(cipher.doFinal(strToEncrypt.getBytes(UTF_8)));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }


    public static String decrypt(String strToDecrypt, String salt) {
        byte[] iv = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0};
        IvParameterSpec ivspec = new IvParameterSpec(iv);
        try {
            SecretKeyFactory factory = SecretKeyFactory.getInstance(KEY_ALGO);
            KeySpec spec = new PBEKeySpec(secretKey.toCharArray(), salt.getBytes(), 65536, 256);
            SecretKey tmp = factory.generateSecret(spec);
            SecretKeySpec secretKey = new SecretKeySpec(tmp.getEncoded(), AES_ALGO);
            Cipher cipher = Cipher.getInstance(AES_CBC_PKCS5);
            cipher.init(DECRYPT_MODE, secretKey, ivspec);
            return new String(cipher.doFinal(getUrlDecoder().decode(strToDecrypt)));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public static void printUsage() {
        System.out.println("Usage: java EncryptUtil <salt> <payload>");
        System.out.println("       where, 'reqdate' is to be used as salt & payload is as per the webview URL redirection spec doc.");
        System.out.println("  e.g. java EncryptUtil 031120201803460 \"txnid=T1234&sessionid=S1234&srcref=Srcref1234&userid=acme@perfios-aa&redirect=https://example.com\"");
        System.out.println("");
    }

    public static void main(String[] args) {
        String salt = null;
        String payload = null;


        if (args.length < 2) {
            printUsage();
            return;
        }
        salt = args[0];
        payload = args[1];
        if (salt == null || salt.length() == 0) {
            System.out.println("Error: Salt must not be empty.");
            printUsage();
            return;
        } else {
            System.out.println("Using salt: " + salt);
        }
        if (payload == null || payload.length() == 0) {
            System.out.println("Error: Payload must not be empty.");
            printUsage();
            return;
        }
        System.out.println("Using payload: " + payload);
        String encData = encrypt(payload, salt);

        if (encData != null) {
            System.out.println("Encrypted & encoded data: " + encData);
            System.out.println("Decrypted data (to verify): " + decrypt(encData, salt));

            String xoredFI = encryptValueToXor(fi, salt);
            System.out.println("Xored FI: " + xoredFI);
            System.out.println("Xored FI (reversed): " + decryptXoredValue(xoredFI, salt));
        }
    }

    public static byte[] getRandomNonce(int numBytes) {
        byte[] nonce = new byte[numBytes];
        new SecureRandom().nextBytes(nonce);
        return nonce;
    }


    /**
     * Generate xored output of a using key.
     *
     * @param a
     * @param key
     * @return
     */
    static byte[] xor(byte[] a, byte[] key) {
        byte[] out = new byte[a.length];
        for (int i = 0; i < a.length; i++) {
            out[i] = (byte) (a[i] ^ key[i % key.length]);
        }
        return out;
    }

    static String decryptXoredValue(String xoredValue, String key) {
        return new String(xor(Base64.getDecoder().decode(xoredValue.getBytes()), key.getBytes()));
    }

    static String encryptValueToXor(String value, String key) {
        return new String(Base64.getEncoder().encode(xor(value.getBytes(), key.getBytes())));
    }
}

```


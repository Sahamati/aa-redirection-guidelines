# Java

Sample Java code for XOR and encryption of the **ecreq** and **ecres** fields.

```text
import static java.nio.charset.StandardCharsets.UTF_8;
import static java.util.Base64.getUrlDecoder;
import static java.util.Base64.getUrlEncoder;
import static javax.crypto.Cipher.DECRYPT_MODE;
import static javax.crypto.Cipher.ENCRYPT_MODE;

import java.security.SecureRandom;
import java.util.Base64;

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;

public class WebRedirection {
	private static final int DEFAULT_PASSWORD_LENGTH = 32;
	private static final int IV_LEN = 16;
	private static final String ALLOWED_CHARS = "0123456789abcdefghijklmnopqrstuvwxyz-_ABCDEFGHIJKLMNOPQRSTUVWXYZ!#$%&*?@^_~";
    private static final String AES_ALGO = "AES";
    private static final String AES_CBC_PKCS5 = "AES/CBC/PKCS5Padding";
    private static final String fi = "FIUID";
    private static final SecureRandom RANDOM = new SecureRandom();

    /**
     * Use this to generate a random password and share with AA/FIU which will be used as a symmetric key between FIU and AA.
     * Password needs to be generated only once during the initial configuration of redirection between FIU and AA and has to be done at the same between AA and FIU.
     * Password needs to be shared out of band. (i.e. secure e-mail or any other secure channel)
     * 
     * @return
     */
	public static String generateRamdomPassword() {
		return RANDOM.ints(DEFAULT_PASSWORD_LENGTH, 0, ALLOWED_CHARS.length()).mapToObj(i -> ALLOWED_CHARS.charAt(i))
				.collect(StringBuilder::new, StringBuilder::append, StringBuilder::append).toString();
	}
	
	/**
	 * Generate random iv of 16 bytes and returns base 64 encoded String.
	 * Use this for every request/response encryption/decyption. Pass the iv value to the FIU/AA in the iv url parameter.
	 * @return
	 */
	public static String generateIV() {
		byte[] iv = new byte[IV_LEN];
		RANDOM.nextBytes(iv);
		return Base64.getEncoder().encodeToString(iv);
	}
    
	/**
	 * Encrypt the ecreq/ecres using this method.
	 * 
	 * @param strToEncrypt
	 * @param password
	 * @param salt
	 * @param ivString
	 * @return
	 */
    public static String encrypt(String strToEncrypt, String password, String salt, String ivString) {
        final byte[] iv = Base64.getDecoder().decode(ivString);
        IvParameterSpec ivspec = new IvParameterSpec(iv);

        try {
        	HKDF hkdf = new HKDF();
        	byte[] key = hkdf.deriveSecrets(password.getBytes(), salt.getBytes(), null, 32);
            SecretKeySpec secretKey = new SecretKeySpec(key, AES_ALGO);
            Cipher cipher = Cipher.getInstance(AES_CBC_PKCS5);
            cipher.init(ENCRYPT_MODE, secretKey, ivspec);
            return getUrlEncoder().encodeToString(cipher.doFinal(strToEncrypt.getBytes(UTF_8)));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * Decrypt the ecreq/ecres using this method.
     * 
     * @param strToDecrypt
     * @param password
     * @param salt
     * @param ivString
     * @return
     */
    public static String decrypt(String strToDecrypt, String password, String salt, String ivString) {
        final byte[] iv = Base64.getDecoder().decode(ivString);
        IvParameterSpec ivspec = new IvParameterSpec(iv);
        try {
        	HKDF hkdf = new HKDF();
        	byte[] key = hkdf.deriveSecrets(password.getBytes(), salt.getBytes(), null, 32);
            SecretKeySpec secretKey = new SecretKeySpec(key, AES_ALGO);
            Cipher cipher = Cipher.getInstance(AES_CBC_PKCS5);
            cipher.init(DECRYPT_MODE, secretKey, ivspec);
            return new String(cipher.doFinal(getUrlDecoder().decode(strToDecrypt)));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public static void printUsage() {
        System.out.println("Usage: java EncryptUtil <password> <salt> <payload>");
        System.out.println("       where, 'reqdate' is to be used as salt & payload is as per the webview URL redirection spec doc.");
        System.out.println("  e.g. java EncryptUtil nsrkGE&-p1Qh*Gn$3P0&Qt0s9!uz1U_I 031120201803460 \"txnid=T1234&sessionid=S1234&srcref=Srcref1234&userid=acme@perfios-aa&redirect=https://example.com\"");
        System.out.println("");
    }

    public static void main(String[] args) {

        if (args.length < 3) {
            printUsage();
            return;
        }

        String password = args[0]; 
        String salt = args[1];
        String payload = args[2];

        if (password == null || password.length() != DEFAULT_PASSWORD_LENGTH) {
            System.out.println("Error: password length must be " + DEFAULT_PASSWORD_LENGTH);
            printUsage();
            return;
        } else {
            System.out.println("Using password: " + password);
        }
        
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
        
        // Generate IV for every redirection and pass in the iv url parameter
        String ivString = generateIV();
        
        String encData = encrypt(payload, password, salt, ivString);

        if (encData != null) {
            System.out.println("Encrypted & encoded data: " + encData);
            String decrypted = decrypt(encData, password, salt, ivString);
            System.out.println("Decrypted data (to verify): " + decrypted);
            System.out.println("Is decrypted data same as input?: " + (decrypted.equals(payload)));

            String xoredFI = encryptValueToXor(fi, salt);
            System.out.println("Xored FI: " + xoredFI);
            System.out.println("Xored FI (reversed): " + decryptXoredValue(xoredFI, salt));
        }
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

import java.io.ByteArrayOutputStream;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;


/**
 * This is a copy of the HKDF file from the libsignal-protocol-java
 * which is a v2 implementation as per RFC 5869
 * All credits go to the original author.
 * For licensing, see the the original github location.
 * 
 * https://github.com/signalapp/libsignal-protocol-java/blob/master/java/src/main/java/org/whispersystems/libsignal/kdf/HKDF.java
 */
public class HKDF {

  private static final int HASH_OUTPUT_SIZE  = 32;
  
  public byte[] deriveSecrets(byte[] inputKeyMaterial, byte[] info, int outputLength) {
    byte[] salt = new byte[HASH_OUTPUT_SIZE];
    return deriveSecrets(inputKeyMaterial, salt, info, outputLength);
  }

  public byte[] deriveSecrets(byte[] inputKeyMaterial, byte[] salt, byte[] info, int outputLength) {
    byte[] prk = extract(salt, inputKeyMaterial);
    return expand(prk, info, outputLength);
  }

  private byte[] extract(byte[] salt, byte[] inputKeyMaterial) {
    try {
      Mac mac = Mac.getInstance("HmacSHA256");
      mac.init(new SecretKeySpec(salt, "HmacSHA256"));
      return mac.doFinal(inputKeyMaterial);
    } catch (NoSuchAlgorithmException | InvalidKeyException e) {
      throw new AssertionError(e);
    }
  }

  private byte[] expand(byte[] prk, byte[] info, int outputSize) {
    try {
      int                   iterations     = (int) Math.ceil((double) outputSize / (double) HASH_OUTPUT_SIZE);
      byte[]                mixin          = new byte[0];
      ByteArrayOutputStream results        = new ByteArrayOutputStream();
      int                   remainingBytes = outputSize;

      for (int i= getIterationStartOffset();i<iterations + getIterationStartOffset();i++) {
        Mac mac = Mac.getInstance("HmacSHA256");
        mac.init(new SecretKeySpec(prk, "HmacSHA256"));

        mac.update(mixin);
        if (info != null) {
          mac.update(info);
        }
        mac.update((byte)i);

        byte[] stepResult = mac.doFinal();
        int    stepSize   = Math.min(remainingBytes, stepResult.length);

        results.write(stepResult, 0, stepSize);

        mixin          = stepResult;
        remainingBytes -= stepSize;
      }

      return results.toByteArray();
    } catch (NoSuchAlgorithmException | InvalidKeyException e) {
      throw new AssertionError(e);
    }
  }

  protected int getIterationStartOffset(){
	  return 1;
  }

}
```

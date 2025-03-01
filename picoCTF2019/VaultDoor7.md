# Vault Door 7

## Introduction
In this post, I'll walk you through my experience solving a picoCTF challenge called "Vault Door 7". This challenge involves analyzing Java source code to uncover a hidden password. It's a great exercise for beginners interested in reverse engineering and source code analysis. I'll share the steps I took to crack the code and the thought process behind reversing the logic.

## My Experience
This forth third Challenge So if i miss something sorry about that Let's See the Questions

<hr/>
This vault uses bit shifts to convert a password string into an array of integers. Hurry, agent, we are running out of time to stop Dr. Evil's nefarious plans! The source code for this vault is here: VaultDoor7.java
<hr/>

so anther java code-base and like before i download it and open that in visual studio and here it is code

```Java
import java.util.*;
import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import java.security.*;

class VaultDoor7 {
    public static void main(String args[]) {
        VaultDoor7 vaultDoor = new VaultDoor7();
        Scanner scanner = new Scanner(System.in);
        System.out.print("Enter vault password: ");
        String userInput = scanner.next();
	String input = userInput.substring("picoCTF{".length(),userInput.length()-1);
	if (vaultDoor.checkPassword(input)) {
	    System.out.println("Access granted.");
	} else {
	    System.out.println("Access denied!");
        }
    }

    // Each character can be represented as a byte value using its
    // ASCII encoding. Each byte contains 8 bits, and an int contains
    // 32 bits, so we can "pack" 4 bytes into a single int. Here's an
    // example: if the hex string is "01ab", then those can be
    // represented as the bytes {0x30, 0x31, 0x61, 0x62}. When those
    // bytes are represented as binary, they are:
    //
    // 0x30: 00110000
    // 0x31: 00110001
    // 0x61: 01100001
    // 0x62: 01100010
    //
    // If we put those 4 binary numbers end to end, we end up with 32
    // bits that can be interpreted as an int.
    //
    // 00110000001100010110000101100010 -> 808542562
    //
    // Since 4 chars can be represented as 1 int, the 32 character password can
    // be represented as an array of 8 ints.
    //
    // - Minion #7816
    public int[] passwordToIntArray(String hex) {
        int[] x = new int[8];
        byte[] hexBytes = hex.getBytes();
        for (int i=0; i<8; i++) {
            x[i] = hexBytes[i*4]   << 24
                 | hexBytes[i*4+1] << 16
                 | hexBytes[i*4+2] << 8
                 | hexBytes[i*4+3];
        }
        return x;
    }

    public boolean checkPassword(String password) {
        if (password.length() != 32) {
            return false;
        }
        int[] x = passwordToIntArray(password);
        return x[0] == 1096770097
            && x[1] == 1952395366
            && x[2] == 1600270708
            && x[3] == 1601398833
            && x[4] == 1716808014
            && x[5] == 1734304867
            && x[6] == 942695730
            && x[7] == 942748212;
    }
}
```


```Java
import java.nio.charset.Charset;
import java.util.Arrays;

public class App {
    public static void main(String[] args) throws Exception {
        String password = genPassword();
        System.out.println(password);
    }

    public static byte[] passwordToHexArray(int[] x) {
        byte[] hexBytes = new byte[32];
        for (int i = 0; i < 8; i++) {
            hexBytes[i * 4] = (byte) (x[i] >> 24);
            hexBytes[i * 4 + 1] = (byte) (x[i] >> 16);
            hexBytes[i * 4 + 2] = (byte) (x[i] >> 8);
            hexBytes[i * 4 + 3] = (byte) x[i];
        }
        return hexBytes;
    }

    public static String genPassword() {
        int[] password = {
                1096770097, 1952395366, 1600270708, 1601398833,
                1716808014, 1734304867, 942695730, 942748212
        };

        byte[] pass = passwordToHexArray(password);
        return new String(pass);
    }
}
```

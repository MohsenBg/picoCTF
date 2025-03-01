# Vault Door 7

## Introduction
In this challenge, we analyze a Java program that verifies a password. Instead of brute-forcing, we reverse-engineer its logic to find the correct input.
The checkPassword function applies a bitwise XOR to each character. By reversing this operation, we can reconstruct the valid password.
This write-up walks through the code analysis and password generation step by step

## My Experience
Let's See The Question and analyze it:
<hr/>
This vault uses bit shifts to convert a password string into an array of integers. Hurry, agent, we are running out of time to stop Dr. Evil's nefarious plans! The source code for this vault is here: VaultDoor7.java
<hr/>
In this challenge, we will work on cracking XOR encryption as mentioned in the question. Let's open the Java file provided by the challenge.
 
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

Let's analyze the code and figure out how the password is validated so we can generate a valid password to bypass this level.

If you solved the Vault Door Challenge, you should already know that the main part of the code we need to check is the `checkPassword` function.

``` Java
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
```
The `checkPassword` function first checks if the password length is 32. Then, it converts the password into a byte array and loops over each byte, XORing it with 0x55 and subtracting a corresponding byte from myBytes. If the result is not zero, the password is incorrect.

We can simplify the if statement like this:
```Java
if ((passBytes[i] ^ 0x55) !=  myBytes[i]) {
    return false;
}
```
If you look at it closely, you can see that the result of XORing the password with 0x55 must equal myBytes[i].

So, we can reverse that by XORing myBytes with 0x55. Let's write a Java function to do that:

```Java
public static byte[] passwordToHexArray(int[] x) {
    byte[] hexBytes = new byte[32];
    for (int i = 0; i < 8; i++) {
        hexBytes[i*4]   = (byte) (x[i] >> 24);
        hexBytes[i*4+1] = (byte) (x[i] >> 16);
        hexBytes[i*4+2] = (byte) (x[i] >> 8);
        hexBytes[i*4+3] = (byte) x[i];
    }
    return hexBytes;
}
   
public static String genPassword() {
    int [] password = {
        1096770097,1952395366,1600270708,1601398833,
        1716808014,1734304867,942695730,942748212
    };

    byte[] pass = passwordToHexArray(password);
    return new String(pass);
}

```

Now, running `genPassword()` will generate the correct password to bypass this level.


The password is:
```
A_b1t_0f_b1t_sh1fTiNg_dc80e28124
```

Therefore, the flag is:
```
picoCTF{A_b1t_0f_b1t_sh1fTiNg_dc80e28124}
```

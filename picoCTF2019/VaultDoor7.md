# Vault Door 7

## Introduction
In this post, we solve the picoCTF "Vault Door 7" Reverse Engineering challenge, which uses bitwise shifts to validate the password, making it harder to analyze. This is a great exercise for improving your reverse engineering skills! 🚀

## My Experience
Let's See The Question and analyze it:
<hr/>
This vault uses bit shifts to convert a password string into an array of integers. Hurry, agent, we are running out of time to stop Dr. Evil's nefarious plans! The source code for this vault is here: VaultDoor7.java
<hr/>
This Vault Door 7 challenge uses bit shifts to convert a password into an array of integers, making reverse engineering more complex. The provided source code is in Java, and our task is to reconstruct the correct password.

Since this is a Vault Door challenge, I downloaded the `VaultDoor7.java` file and opened it in VS Code to analyze its logic.

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

The code first removes the picoCTF{} prefix, then converts the password into an array of integers using the passwordToIntArray function. The challenge checks if this array matches predefined integer values.

Now, let’s break it down and extract the password! 


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

When the checkPassword function is called with the user’s password as an argument, it first checks if the password length is 32. Then, it calls passwordToIntArray, which converts the string into an array of 32-bit integers.

This function processes the password in groups of four characters, combining them into a single 32-bit integer. Since each character is 1 byte (8 bits) and there are four characters in each group, the total size becomes 4 × 8 = 32 bits.

### How checkPassword Converts a String to an Array of 32-Bit Integers
Let's analyze this using an example. Suppose our password contains the four-character sequence "01ab".
The most significant bits of the 32-bit integer are determined by the first character, '0'. Its ASCII value is 0x30 (00110000 in binary). To position it as the most significant byte, we shift it left by 24 bits:

``` bash
00110000 << 24  →  00110000 00000000 00000000 00000000
```

The next character, '1', has an ASCII value of 0x31 (00110001 in binary). It fills the next 8 bits (23–16), so we shift it left by 16 bits:

```bash 
00110001 << 16  →  00000000 00110001 00000000 00000000
```

The third character, 'a', has an ASCII value of 0x61 (01100001 in binary). It fills bits 15–8, so we shift it left by 8 bits

``` bash
01100001 << 8  →  00000000 00000000 01100001 00000000
```

The last character, 'b', has an ASCII value of 0x62 (01100010 in binary). It occupies the least significant byte, so no shifting is needed:

```bash 
01100010  →  00000000 00000000 00000000 01100010
```

Now, we use the bitwise OR operation to combine all four parts into a single 32-bit integer:

```bash
"0":        00110000 00000000 00000000 00000000  |
"1":        00000000 00110001 00000000 00000000  |
"a":        00000000 00000000 01100001 00000000  |
"b":        00000000 00000000 00000000 01100010  
---------------------------------------------------
"result":   00110000 00110001 01100001 01100010  
```

Let's write a Java function to Reverse that:

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

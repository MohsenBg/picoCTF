# Vault Door 6

## Introduction
In this challenge, we analyze a Java program that verifies a password. Instead of brute-forcing, we reverse-engineer its logic to find the correct input.
The checkPassword function applies a bitwise XOR to each character. By reversing this operation, we can reconstruct the valid password.
This write-up walks through the code analysis and password generation step by step

## My Experience
Let's See What is the question of This challenge.
<hr/>
This vault uses an XOR encryption scheme. The source code for this vault is here: VaultDoor6.java
<hr/>
In this challenge, we will work on cracking XOR encryption as mentioned in the question. Let's open the Java file provided by the challenge.

```Java
import java.util.*;

class VaultDoor6 {
    public static void main(String args[]) {
        VaultDoor6 vaultDoor = new VaultDoor6();
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

    // Dr. Evil gave me a book called Applied Cryptography by Bruce Schneier,
    // and I learned this really cool encryption system. This will be the
    // strongest vault door in Dr. Evil's entire evil volcano compound for sure!
    // Well, I didn't exactly read the *whole* book, but I'm sure there's
    // nothing important in the last 750 pages.
    //
    // -Minion #3091
    public boolean checkPassword(String password) {
        if (password.length() != 32) {
            return false;
        }
        byte[] passBytes = password.getBytes();
        byte[] myBytes = {
            0x3b, 0x65, 0x21, 0xa , 0x38, 0x0 , 0x36, 0x1d,
            0xa , 0x3d, 0x61, 0x27, 0x11, 0x66, 0x27, 0xa ,
            0x21, 0x1d, 0x61, 0x3b, 0xa , 0x2d, 0x65, 0x27,
            0xa , 0x66, 0x36, 0x30, 0x67, 0x6c, 0x64, 0x6c,
        };
        for (int i=0; i<32; i++) {
            if (((passBytes[i] ^ 0x55) - myBytes[i]) != 0) {
                return false;
            }
        }
        return true;
    }
}
```

Let's analyze the code and figure out how the password is validated so we can generate a valid password to bypass this level.

If you solved the Vault Door Challenge, you should already know that the main part of the code we need to check is the `checkPassword` function.

``` Java
public boolean checkPassword(String password) {
    if (password.length() != 32) {
        return false;
    }
    byte[] passBytes = password.getBytes();
    byte[] myBytes = {
        0x3b, 0x65, 0x21, 0xa , 0x38, 0x0 , 0x36, 0x1d,
        0xa , 0x3d, 0x61, 0x27, 0x11, 0x66, 0x27, 0xa ,
        0x21, 0x1d, 0x61, 0x3b, 0xa , 0x2d, 0x65, 0x27,
        0xa , 0x66, 0x36, 0x30, 0x67, 0x6c, 0x64, 0x6c,
    };
    for (int i=0; i<32; i++) {
        if (((passBytes[i] ^ 0x55) - myBytes[i]) != 0) {
            return false;
        }
    }
    return true;
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
public static String genPassword() {
    byte[] passBytes = new byte[32];
    byte[] myBytes = {
        0x3b, 0x65, 0x21, 0xa , 0x38, 0x0 , 0x36, 0x1d,
        0xa , 0x3d, 0x61, 0x27, 0x11, 0x66, 0x27, 0xa ,
        0x21, 0x1d, 0x61, 0x3b, 0xa , 0x2d, 0x65, 0x27,
        0xa , 0x66, 0x36, 0x30, 0x67, 0x6c, 0x64, 0x6c,
    };     
    for (int i=0; i<32; i++) {
        passBytes[i] = (byte) (myBytes[i] ^ 0x55); 
    }
       
    return new String(passBytes);
}
```

Now, running `genPassword()` will generate the correct password to bypass this level.


The password is:
```
n0t_mUcH_h4rD3r_tH4n_x0r_3ce2919
```

Therefore, the flag is:
```
picoCTF{n0t_mUcH_h4rD3r_tH4n_x0r_3ce2919}
```

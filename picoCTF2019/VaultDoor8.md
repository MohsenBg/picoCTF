# Vault Door 8

## Introduction
In this post, we solve the picoCTF "Vault Door 8" Reverse Engineering challenge, which uses bitwise shifts to validate the password, making it harder to analyze. This is a great exercise for improving your reverse engineering skills! 🚀

## My Experience
Let's See The Question and analyze it:

<hr/>
Apparently Dr. Evil's minions knew that our agency was making copies of their source code, because they intentionally sabotaged this source code in order to make it harder for our agents to analyze and crack into! The result is a quite mess, but I trust that my best special agent will find a way to solve it. The source code for this vault is here: VaultDoor8.java
<hr/>

As usual in the Vault Door challenge, we need to analyze the Java source code. The question tells us that the minions have changed the previous source code, so let's download it and open it in VS Code to take a look!

```Java
// These pesky special agents keep reverse engineering our source code and then
// breaking into our secret vaults. THIS will teach those sneaky sneaks a
// lesson.
//
// -Minion #0891
import java.util.*;
import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import java.security.*;

class VaultDoor8 {
    public static void main(String args[]) {
        Scanner b = new Scanner(System.in);
        System.out.print("Enter vault password: ");
        String c = b.next();
        String f = c.substring(8, c.length() - 1);
        VaultDoor8 a = new VaultDoor8();
        if (a.checkPassword(f)) {
            System.out.println("Access granted.");
        } else {
            System.out.println("Access denied!");
        }
    }

    public char[] scramble(String password) {
        /* Scramble a password by transposing pairs of bits. */
        char[] a = password.toCharArray();
        for (int b = 0; b < a.length; b++) {
            char c = a[b];
            c = switchBits(c, 1, 2);
            c = switchBits(c, 0, 3);

            c = switchBits(c, 5, 6);
            c = switchBits(c, 4, 7);
            c = switchBits(c, 0, 1);

            c = switchBits(c, 3, 4);
            c = switchBits(c, 2, 5);
            c = switchBits(c, 6, 7);
            a[b] = c;
        }
        return a;
    }

    public char switchBits(char c, int p1, int p2) {
        /*
         * Move the bit in position p1 to position p2, and move the bit
         * that was in position p2 to position p1. Precondition: p1 < p2
         */ 
        char mask1 = (char) (1 << p1);
        char mask2 = (char) (1 << p2);

        char bit1 = (char) (c & mask1);
        char bit2 = (char) (c & mask2);
     
        char rest = (char) (c & ~(mask1 | mask2));
        char shift = (char) (p2 - p1);
        char result = (char) ((bit1 << shift) | (bit2 >> shift) | rest);
        return result;
    }

    public boolean checkPassword(String password) {
        char[] scrambled = scramble(password);
        char[] expected = {
                0xF4, 0xC0, 0x97, 0xF0, 0x77, 0x97, 0xC0, 0xE4, 0xF0, 0x77, 0xA4, 0xD0, 0xC5, 0x77, 0xF4, 0x86, 0xD0,
                0xA5, 0x45, 0x96, 0x27, 0xB5, 0x77, 0xD2, 0xD0, 0xB4, 0xE1, 0xC1, 0xE0, 0xD0, 0xD0, 0xE0 };
        return Arrays.equals(scrambled, expected);
    }
}
```

Like before, the `picoCTF{}` part is removed from the password input, and later, the `checkPassword` function is called with the modified password. As you can figure out, just like in previous Vault Door challenges, the main part of the code is the `checkPassword` function.

The `checkPassword` function calls another function named `scramble`, which swaps bits and then converts the result into an array of characters. Later, it compares each character with another character array called `expected`. If all characters match in both arrays, it returns `true`; otherwise, it returns `false`.

So, to reverse that operation, we need to pass `expected` as an argument to `scramble` and then swap the bits in reverse order to get the real password. Let's write a Java code to do that!

```Java
public static char[] scramble(String password) {
    char[] a = password.toCharArray();
    for (int b = 0; b < a.length; b++) {
        char c = a[b];
       
        c = switchBits(c, 6, 7);
        c = switchBits(c, 2, 5);
        c = switchBits(c, 3, 4);
        c = switchBits(c, 0, 1);
        c = switchBits(c, 4, 7);
        c = switchBits(c, 5, 6);
        c = switchBits(c, 0, 3);
        c = switchBits(c, 1, 2);

        a[b] = c;
    }
    return a;
}

public static char switchBits(char c, int p1, int p2) {
    char mask1 = (char) (1 << p1); 
    char mask2 = (char) (1 << p2); 
       
    char bit1 = (char) (c & mask1); 
    char bit2 = (char) (c & mask2); 
            
    char rest = (char) (c & ~(mask1 | mask2));
    char shift = (char) (p2 - p1);
    char result = (char) ((bit1 << shift) | (bit2 >> shift) | rest);
    return result;
}

public static String genPassword() {
    char[] expected = {
            0xF4, 0xC0, 0x97, 0xF0, 0x77, 0x97, 0xC0, 0xE4, 0xF0, 0x77, 0xA4, 0xD0, 0xC5, 0x77, 0xF4, 0x86, 0xD0,
            0xA5, 0x45, 0x96, 0x27, 0xB5, 0x77, 0xD2, 0xD0, 0xB4, 0xE1, 0xC1, 0xE0, 0xD0, 0xD0, 0xE0 };
    String scramble = new String(expected);
    char[] password = scramble(scramble)

    return new String(password);
}
```

Now, running `genPassword()` will generate the correct password to bypass this level.


The password is:
```
s0m3_m0r3_b1t_sh1fTiNg_91c642112
```

Therefore, the flag is:
```
picoCTF{s0m3_m0r3_b1t_sh1fTiNg_91c642112}
```

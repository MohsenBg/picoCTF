# Vault Door 3

## Introduction

In this post, I'll walk you through my experience solving a picoCTF challenge called "Vault Door 3." This challenge involves analyzing Java source code to uncover a hidden password. It's a great exercise for beginners interested in reverse engineering and source code analysis. I'll share the steps I took to crack the code and the thought process behind reversing the logic.

## My Experience

Today I start anther CTF challenge to solve, and here it is:

<hr/>
vault uses for-loops and byte arrays. The source code for this vault is here: VaultDoor3.java
<hr/>

After solving the previous challenges, I was curious to see if this one would be more challenging. Like the previous Vault Door challenges, this one is based on Java source code. I downloaded the file and opened it in Visual Studio Code.

and here it is the code:

``` java
import java.util.*;

class VaultDoor3 {
    public static void main(String args[]) {
        VaultDoor3 vaultDoor = new VaultDoor3();
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

    public boolean checkPassword(String password) {
        if (password.length() != 32) {
            return false;
        }
        char[] buffer = new char[32];
        int i;
        for (i=0; i<8; i++) {
            buffer[i] = password.charAt(i);
        }
        for (; i<16; i++) {
            buffer[i] = password.charAt(23-i);
        }
        for (; i<32; i+=2) {
            buffer[i] = password.charAt(46-i);
        }
        for (i=31; i>=17; i-=2) {
            buffer[i] = password.charAt(i);
        }
        String s = new String(buffer);
        return s.equals("jU5t_a_sna_3lpm12g94c_u_4_m7ra41");
    }
}
```

The `checkPassword` function is responsible for verifying the password. It manipulates characters from the input password using four for-loops to create a buffer array. The buffer is then checked against the hardcoded string `"jU5t_a_sna_3lpm12g94c_u_4_m7ra41"`.

To solve this challenge, I needed to reverse the logic of each loop to retrieve the original password.
Before showing the code, I want to mention that I added some comments to help me understand the program's flow and loops. In this challenge, we need to reverse the logic of the code to get the password. If you prefer to see the code without my comments, you can check the original version at the top.

here is the `checkPassword` function :
``` java
    public boolean checkPassword(String password) {
        if (password.length() != 32) {
            return false;
        }
        char[] buffer = new char[32];
        int i;
        
        // buffer[1] = password[1]
        // buffer[2] = password[2]
        // ...
        // buffer[7] = password[7]
        for (i=0; i<8; i++) {
            buffer[i] = password.charAt(i);
        }


        // buffer[8]  = password[14] 
        // buffer[9]  = password[13]
        // ...
        // buffer[15] = password[8]
        for (; i<16; i++) {
            buffer[i] = password.charAt(23-i); 
        }


        // buffer[16]  = password[30] 
        // buffer[18]  = password[32]
        // ...
        // buffer[30]  = password[16]
        for (; i<32; i+=2) {
            buffer[i] = password.charAt(46-i);
        }

        
        // buffer[31]  = password[31] 
        // buffer[30]  = password[30]
        // ...
        // buffer[17]  = password[17]
        for (i=31; i>=17; i-=2) {
            buffer[i] = password.charAt(i);
        }
        String s = new String(buffer);
        return s.equals("jU5t_a_sna_3lpm12g94c_u_4_m7ra41");
    }
```

As you can see, there are four for-loops in the code. In each loop, some characters are moved to the buffer array—some in the exact positions of the password and others in different positions. Our goal is to reverse the logic of this function so that we can retrieve the password from the buffer values.

You could manually trace the loops and logic in reverse order, but since we have computers, we can use them to crack the password more efficiently.

But how do we reverse the logic?
Good question! We need to move through the function from bottom to top.

Here’s the function that converts the buffer to the password. By the way, I used Java because the challenge itself was written in Java.java too:

``` java
public class App {
    public static void main(String[] args) {
        String password = genPassword();
        System.out.println(password);
    }

    public static String genPassword() {
        char[] buffer = {
                'j', 'U', '5', 't', '_', 'a', '_', 's', 'n', 'a', '_',
                '3', 'l', 'p', 'm', '1', '2', 'g', '9', '4', 'c', '_',
                'u', '_', '4', '_', 'm', '7', 'r', 'a', '4', '1'
        };

        char[] password = new char[32];

        // Reverse the last loop first
        for (int i=31; i>=17; i-=2) {
            password[i] = buffer[i];
        }

        // Reverse the third loop
        for (int i = 16; i<32; i+=2) {
            password[46-i] = buffer[i];
        }

        // Reverse the second loop
        for (int i = 8; i<16; i++) {
            password[23-i] = buffer[i];
        }

        // Reverse the first loop
        for (int i=0; i<8; i++) {
            password[i] = buffer[i];
        }

        return new String(password);
    }
}

```
if you’re looking at the code and feeling confused, don’t worry! Let's break it down step by step.

First, we need a function that takes the buffer and converts it into the password. We already have the buffer:
```
jU5t_a_sna_3lpm12g94c_u_4_m7ra41
```
I stored each character in a `char[] buffer` array. To store the generated password, I used a `char[] password` instead of a `String` because it's easier to manipulate individual characters in an array, especially when working with multiple loops.

As mentioned before, we’re reversing the logic by going from the bottom to the top. Let's analyze each loop one by one:


### First Loop
```java
// buffer[31] = password[31] 
// buffer[30] = password[30]
// ...
// buffer[17] = password[17]
for (int i = 31; i >= 17; i -= 2) {
    buffer[i] = password.charAt(i);
}
```
To reverse the logic, notice that `buffer[i] = password[i].` This means the character at position i is simply copied from password to buffer without any changes.
So, to reverse it, we just swap the assignment:

```java
for (int i = 31; i >= 17; i -= 2) {
    password[i] = buffer[i];
}
```

### Second Loop
```java
// buffer[16] = password[30] 
// buffer[18] = password[28]
// ...
// buffer[30] = password[16]
for (; i < 32; i += 2) {
    buffer[i] = password.charAt(46 - i);
}

```
To reverse this logic, we first check the initial value of i, which is 16 after the previous loop.
We then swap the positions of buffer and password, like this

```java
for (int i = 16; i < 32; i += 2) {
    password[46 - i] = buffer[i];
}
```

### Third Loop

```java
// buffer[8] = password[14] 
// buffer[9] = password[13]
// ...
// buffer[15] = password[8]
for (; i < 16; i++) {
    buffer[i] = password.charAt(23 - i); 
}
```
From the previous loop, i is now 8. We just need to swap the positions of buffer and password:

```java
for (int i = 8; i < 16; i++) {
    password[23 - i] = buffer[i];
}
```

### Fourth (Last) Loop
```java
// buffer[1] = password[1]
// buffer[2] = password[2]
// ...
// buffer[7] = password[7]
for (i=0; i<8; i++) {
    buffer[i] = password.charAt(i);
}
```
The last loop is similar to the first one, where the characters are directly copied:
```java
for (int i = 0; i < 8; i++) {
    password[i] = buffer[i];
}
```
## Final Output

Now, Let's run the code to get the password:

The password is:
```
jU5t_a_s1mpl3_an4gr4m_4_u_c79a21
```

Therefore, the flag is:
```
picoCTF{jU5t_a_s1mpl3_an4gr4m_4_u_c79a21}
```

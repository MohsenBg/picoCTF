# Vault Door 1

## Introduction
In this post, we tackle the Java-based code challenge called "Vault Door 1." Our goal is to reverse engineer the source code to understand how the password is validated and create the correct password to bypass this level.

## My Experience
New Days New Challenge to Solve so  I tried to solve the question, and here it is:

<hr/>
This vault uses some complicated arrays! I hope you can make sense of it, special agent. The source code for this vault is here: VaultDoor1.java
<hr/>

I had solved a similar challenge named vault-door-training, so I expected to encounter another Java code-based challenge. After reading the question, I discovered my guess was correct. I opened the code in Visual Studio Code and saw the following:

``` java
import java.util.*;

class VaultDoor1 {
    public static void main(String args[]) {
        VaultDoor1 vaultDoor = new VaultDoor1();
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

    // I came up with a more secure way to check the password without putting
    // the password itself in the source code. I think this is going to be
    // UNHACKABLE!! I hope Dr. Evil agrees...
    //
    // -Minion #8728
    public boolean checkPassword(String password) {
        return password.length() == 32 &&
               password.charAt(0)  == 'd' &&
               password.charAt(29) == '9' &&
               password.charAt(4)  == 'r' &&
               password.charAt(2)  == '5' &&
               password.charAt(23) == 'r' &&
               password.charAt(3)  == 'c' &&
               password.charAt(17) == '4' &&
               password.charAt(1)  == '3' &&
               password.charAt(7)  == 'b' &&
               password.charAt(10) == '_' &&
               password.charAt(5)  == '4' &&
               password.charAt(9)  == '3' &&
               password.charAt(11) == 't' &&
               password.charAt(15) == 'c' &&
               password.charAt(8)  == 'l' &&
               password.charAt(12) == 'H' &&
               password.charAt(20) == 'c' &&
               password.charAt(14) == '_' &&
               password.charAt(6)  == 'm' &&
               password.charAt(24) == '5' &&
               password.charAt(18) == 'r' &&
               password.charAt(13) == '3' &&
               password.charAt(19) == '4' &&
               password.charAt(21) == 'T' &&
               password.charAt(16) == 'H' &&
               password.charAt(27) == '5' &&
               password.charAt(30) == '2' &&
               password.charAt(25) == '_' &&
               password.charAt(22) == '3' &&
               password.charAt(28) == '0' &&
               password.charAt(26) == '7' &&
               password.charAt(31) == 'e';
    }
}
```
As I mentioned in the previous challenge, if you know a bit of Java, you can figure out how the password is constructed. If you're not familiar with it, don’t worry! Just like the previous challenge, the main part is the `checkPassword` function, so it’s worth inspecting closely.

``` java
    public boolean checkPassword(String password) {
        return password.length() == 32 &&
               password.charAt(0)  == 'd' &&
               password.charAt(29) == '9' &&
               password.charAt(4)  == 'r' &&
               password.charAt(2)  == '5' &&
               password.charAt(23) == 'r' &&
               password.charAt(3)  == 'c' &&
               password.charAt(17) == '4' &&
               password.charAt(1)  == '3' &&
               password.charAt(7)  == 'b' &&
               password.charAt(10) == '_' &&
               password.charAt(5)  == '4' &&
               password.charAt(9)  == '3' &&
               password.charAt(11) == 't' &&
               password.charAt(15) == 'c' &&
               password.charAt(8)  == 'l' &&
               password.charAt(12) == 'H' &&
               password.charAt(20) == 'c' &&
               password.charAt(14) == '_' &&
               password.charAt(6)  == 'm' &&
               password.charAt(24) == '5' &&
               password.charAt(18) == 'r' &&
               password.charAt(13) == '3' &&
               password.charAt(19) == '4' &&
               password.charAt(21) == 'T' &&
               password.charAt(16) == 'H' &&
               password.charAt(27) == '5' &&
               password.charAt(30) == '2' &&
               password.charAt(25) == '_' &&
               password.charAt(22) == '3' &&
               password.charAt(28) == '0' &&
               password.charAt(26) == '7' &&
               password.charAt(31) == 'e';
    }
}
```

The checkPassword function does exactly what its name suggests – it checks if the user-entered password is correct. The password is generated when the code runs. You may ask how the password is generated, and the answer is simple: each character is placed in its correct position within the password string.

As I’m a bit lazy, I wrote a short piece of code to generate the password. You can use any language to do this, but for simplicity, I decided to use Java, as per the question.

Here’s my Java code:

``` java
import java.util.Arrays;

public class App {
    public static void main(String[] args) throws Exception {
        String password = genPassword();
        System.out.println(password);
    }
    public static String genPassword() {
        char[] password = new char[32];
        Arrays.fill(password, '*');
        password[0] = 'd'; // password.charAt(0) == 'd'
        password[29] = '9'; // password.charAt(29) == '9'
        password[4] = 'r'; // password.charAt(4) == 'r'
        password[2] = '5'; // password.charAt(2) == '5'
        password[23] = 'r'; // password.charAt(23) == 'r'
        password[3] = 'c'; // password.charAt(3) == 'c'
        password[17] = '4'; // password.charAt(17) == '4'
        password[1] = '3'; // password.charAt(1) == '3'
        password[7] = 'b'; // password.charAt(7) == 'b'
        password[10] = '_'; // password.charAt(10) == 'b'
        password[5] = '4'; // password.charAt(5) == '4'
        password[9] = '3'; // password.charAt(9) == '3'
        password[11] = 't'; // password.charAt(11) == 't'
        password[15] = 'c'; // password.charAt(15) == 'c'
        password[8] = 'l'; // password.charAt(8) == 'l'
        password[12] = 'H'; // password.charAt(12) == 'H'
        password[20] = 'c'; // password.charAt(20) == 'c'
        password[14] = '_'; // password.charAt(14) == '_'
        password[6] = 'm'; // password.charAt(6) == 'm'
        password[24] = '5'; // password.charAt(24) == '5'
        password[18] = 'r'; // password.charAt(18) == 'r'
        password[13] = '3'; // password.charAt(13) == '3'
        password[19] = '4'; // password.charAt(19) == '4'
        password[21] = 'T'; // password.charAt(21) == 'T'
        password[16] = 'H'; // password.charAt(16) == 'H'
        password[27] = '5'; // password.charAt(27) == '5'
        password[30] = '2'; // password.charAt(30) == '2'
        password[25] = '_'; // password.charAt(25) == '_'
        password[22] = '3'; // password.charAt(22) == '3'
        password[28] = '0'; // password.charAt(28) == '0'
        password[26] = '7'; // password.charAt(26) == '7'
        password[31] = 'e'; // password.charAt(31) == 'e'
    
        return new String(password);
    }
}

```

The main part, as you know, is the `genPassword` function, which creates the password simply by placing each character in its correct position. After running the code, I got the following password:

The password is:
```
d35cr4mbl3_tH3_cH4r4cT3r5_75092e
```

Therefore, the flag is:
```
picoCTF{d35cr4mbl3_tH3_cH4r4cT3r5_75092e}
```

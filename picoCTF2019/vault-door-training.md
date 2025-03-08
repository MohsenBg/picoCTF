# 🔒 Vault Door Training  

- **📛 Name:** Vault Door Training  
- **⚡ Level:** Easy  
- **🔗 Link:** [Challenge Link](https://play.picoctf.org/practice/challenge/7?category=3&originalEvent=1&page=1)  
- **☕ File Type:** Java  


## Introduction

In this post, I'll share my experience solving a picoCTF challenge called 'Vault Door Training.' This challenge involves analyzing a piece of Java source code to find a hidden password. It's a great introduction to basic reverse engineering and source code analysis.

## My Experience

I tried to solve the question, and here it is:

<hr/>
Your mission is to enter Dr. Evil's laboratory and retrieve the blueprints for his Doomsday Project. The laboratory is protected by a series of locked vault doors. Each door is controlled by a computer and requires a password to open. Unfortunately, our undercover agents have not been able to obtain the secret passwords for the vault doors, but one of our junior agents obtained the source code for each vault's computer! You will need to read the source code for each level to figure out what the password is for that vault door. As a warmup, we have created a replica vault in our training facility. The source code for the training vault is here: VaultDoorTraining.java
<hr/>

At first, I thought, "Oh, it's a Java program, but I’ve never decompiled a Java program before." However, after downloading the file, I realized there was no need to decompile anything because it was the source code, not the bytecode. It turned out to be easier than I expected.


When I opened the file, I saw the code below:
``` java
import java.util.*;

class VaultDoorTraining {
    public static void main(String args[]) {
        VaultDoorTraining vaultDoor = new VaultDoorTraining();
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

    // The password is below. Is it safe to put the password in the source code?
    // What if somebody stole our source code? Then they would know what our
    // password is. Hmm... I will think of some ways to improve the security
    // on the other doors.
    //
    // -Minion #9567
    public boolean checkPassword(String password) {
        return password.equals("w4rm1ng_Up_w1tH_jAv4_eec0716b713");
    }
}

```

If you know a little bit of Java, you can easily find the answer. If not, the answer is hidden in this part:

``` java
 public boolean checkPassword(String password) {
        return password.equals("w4rm1ng_Up_w1tH_jAv4_eec0716b713");
    }
```
The `checkPassword` function does exactly what its name suggests – it checks if the user-entered password is correct. And the password is right there in the source code without any hashing.

The password is:

```
 w4rm1ng_Up_w1tH_jAv4_eec0716b713
```

Therefore, the flag is:
```
picoCTF{w4rm1ng_Up_w1tH_jAv4_eec0716b713}
```

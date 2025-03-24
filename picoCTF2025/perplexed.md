# 🚀 Perplexed

- **📛 Challenge Name:** Perplexed
- **🎯 Difficulty:** Medium  
- **🔗 Challenge:** -  
- **🐧 File Type:** Linux ELF Binary  

---

## Introduction
In this post, we walk through solving the `Perplexed` challenge, leveraging tools like Ghidra and Cutter to reverse-engineer the Linux ELF binary and uncover the flag.

## My Experience

### Ghidra - Main Function

The `main` function of the program prompts the user for a password, then calls the `check` function to validate it. Here's the disassembled code from Ghidra:

```c
bool main(void)
{
  bool isPasswordCorrect;
  char password[268];
  int checkResult;
  
  // Initializing password array to null characters
  memset(password, '\0', sizeof(password));
  
  printf("Enter the password: ");
  fgets(password, 256, stdin);  // Read user input
  
  checkResult = check(password);  // Validate password
  isPasswordCorrect = checkResult != 1;
  
  if (isPasswordCorrect) {
    puts("Correct!! :D");
  }
  else {
    puts("Wrong :(");
  }
  
  return !isPasswordCorrect;
}
```

The `main` function initializes a password array and prompts the user to input a password. The password is checked by calling the `check` function, which returns 0 if the password is correct and 1 if it is incorrect.

### Ghidra - `check` Function

The `check` function is responsible for validating the password by comparing it against a predefined set of "expected" password bits. Here's the code from Ghidra:

```c
undefined8 check(char *password)
{
  size_t pass_len;
  undefined8 result;
  size_t current_password_index;
  char expected_password_bits[36];
  uint password_bit_mask;
  uint expected_bit_mask;
  undefined4 unused_var;
  int bit_index;
  uint char_index;
  int bit_position;
  int password_index;
  
  pass_len = strlen(password);
  if (pass_len == 27) {  // Check if password length is 27
    // Expected password bits defined as a static array
    expected_password_bits[0] = -31;
    expected_password_bits[1] = -89;
    expected_password_bits[2] = 30;
    expected_password_bits[3] = -8;
    expected_password_bits[4] = 'u';
    expected_password_bits[5] = '#';
    expected_password_bits[6] = '{';
    expected_password_bits[7] = 'a';
    expected_password_bits[8] = -71;
    expected_password_bits[9] = -99;
    expected_password_bits[10] = -4;
    expected_password_bits[0xb] = 'Z';
    expected_password_bits[0xc] = '[';
    expected_password_bits[0xd] = -33;
    expected_password_bits[0xe] = 'i';
    expected_password_bits[0xf] = 210;
    expected_password_bits[0x10] = -2;
    expected_password_bits[0x11] = 27;
    expected_password_bits[0x12] = -0x13;
    expected_password_bits[0x13] = -0xc;
    expected_password_bits[0x14] = -0x13;
    expected_password_bits[0x15] = 'g';
    expected_password_bits[0x16] = -0xc;
    
    password_index = 0;
    bit_position = 0;
    unused_var = 0;
    
    for (char_index = 0; char_index < 23; char_index++) {
      for (bit_index = 0; bit_index < 8; bit_index++) {
         if (bit_position == 0) {
           bit_position = 1;
         }
         
         expected_bit_mask = 1 << (7 - bit_index);
         password_bit_mask = 1 << (7 - bit_position);
         
         // Compare the current password bit with the expected password bit
         if (0 < (int)((int)password[password_index] & password_bit_mask) !=
             0 < (int)((int)expected_password_bits[(int)char_index] & expected_bit_mask)) {
           return 1;  // Return 1 if the password bit doesn't match
         }
         
         bit_position++;
         if (bit_position == 8) {
           bit_position = 0;
           password_index++;
         }
         
         if (password_index == pass_len) {
           return 0;  // Password is correct
         }
      }
    }
    result = 0;
  } else {
    result = 1;  // Invalid password length
  }
  return result;
}
```


## Python Script for Flag Recovery

To recover the correct flag, we can reverse the logic in the `check` function using Python. Below is a script that reconstructs the flag:

```python
def recover_password():
    expected_password_bits = [
        -31, -89, 30, -8, ord('u'), ord('#'), ord('{'), ord('a'), -71, -99, -4, ord('Z'), 
        ord('['), -33, ord('i'), 0xD2, -2, 27, -19, -12, -19, ord('g'), -12
    ]

    password_chars = [0] * 27
    bit_position = 0
    password_index = 0

    for char_index in range(23):
        for bit_index in range(8):
            if bit_position == 0:
                bit_position = 1
            
            expected_bit_mask = 1 << (7 - bit_index)
            password_bit_mask = 1 << (7 - bit_position)

            if (expected_password_bits[char_index] & expected_bit_mask) > 0:
                password_chars[password_index] |= password_bit_mask  
                
            bit_position += 1
            if bit_position == 8:
                bit_position = 0
                password_index += 1

            if password_index == len(password_chars):
                return "".join(chr(c) for c in password_chars)

    return "".join(chr(c) for c in password_chars)

recovered_password = recover_password()
print(f"Recovered password: {recovered_password}")
```

### Output:
```
Recovered password: picoCTF{0n3_bi7_4t_a_7im3}
```

## 🎉 The Flag  
```
picoCTF{0n3_bi7_4t_a_7im3}
```

# Pack and Ship - CTF Write-Up
### A write-up for GDSC Cybersecurity Round 2 Enrollments by Humaidh.

## Task Description

Our GDSC Tech Team accidentally left some sensitive data in their latest binary release! They attempted to protect it with basic obfuscation, but we need you to evaluate how secure it really is. Your challenge is to recover the hidden flag.

- **Flag Format:** `gdsc{...}`
- **Provided File:** `Binary Release`

---

## Tools Used :
- **Linux-based OS**  - Kali Linux
- **GNU Debugger** - gdb
---
### **Step 1: Extracting Strings**

The first step I did was to inspect the binary file by running the ``strings`` command.

```sh
strings release | less
```
![Screenshot](https://i.imgur.com/OL5xnfz.png)

Analyzing the output I found that -
- **"Enter the password"**, **"Incorrect password. Please try again"**, **"Correct password! Here is the flag: %s"** - This suggests that the flag can only be revealed after we put the correct password.
- **"deobfuscate"**, **"obfuscated\_password","obfuscated\_flag"**  - These are functions dealing with the obfuscated data.
---
## Reverse Engineering the Binary File

### **Step 2: Finding the Deobfuscation Function**

I used GDB (GNU-Debugger) to analyze the function calls:

```sh
gdb release
```
![Screenshot](https://i.imgur.com/cXs1zON.png)

Earlier I found out that the program has a function named **deobfuscate**, which deciphers the stored password and flag. To find it's memory address, we can use this command :

```sh
disassemble deobfuscate
```
![Screenshot](https://i.imgur.com/YP4dVQW.png)

Entity **Key** has the memory address "0x2020". 
 
To confirm its location, we used:
```sh
info variables
```
![Screenshot](https://i.imgur.com/lMsIcVJ.png)

From the output, we found these 3 entities along with their memory addresses:

- **XOR Key:** `0x2020`
- **Obfuscated Password:** `0x2040`
- **Obfuscated Flag:** `0x2060`

To inspect their contents, we ran:

```sh
x/7bx 0x2020  # XOR key (Ignoring the last character - 0x00)
x/32bx 0x2040 # Obfuscated password
x/16bx 0x2060 # Obfuscated flag
```
![Screenshot](https://i.imgur.com/rRJsCuq.png) 
![Screenshot](https://i.imgur.com/LTJBKWv.png)
![Screenshot](https://i.imgur.com/K2AvJe2.png)
We find hex dumps associated with each of the functions above with the values :
- **XOR Key:** `53 a4 79 b2 c1 e5 7d`

- **Obfuscated Password:** `31 d0 41 c7 80 d4 08 2b e2 4a 87 f1 9c 27 1f d1 0d dd f8 a2 10 22 f0 2e f8 83 95 2d 61 ce 2c c3`
- **Obfuscated Flag:** `34 c0 0a d1 ba 90 13 23 90 1a d9 f0 8b 4b 0c c6 48 dc f5 97 4c 60 91 26 83 f4 ba 13 67 d1 4f da f6 9c 00`
---
## Recognizing the Obfuscation encryption :
After analyzing the key, we find that the key is 7 letters long (ignoring the 0x00 character). 

As said in the problem that a basic-obfuscation method was used to encrypt this file which means that this was a repeated-XOR-encrypted file, where the 7 letter of the key keep repeating until the full flag/password is deciphered.

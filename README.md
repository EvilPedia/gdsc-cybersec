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

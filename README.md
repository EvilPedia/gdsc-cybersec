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

Analyzing the output we can find that -
- **"Enter the password"**, **"Incorrect password. Please try again"**, **"Correct password! Here is the flag: %s"** - suggests that the flag is revealed after entering the correct password
- **"deobfuscate"**, **"obfuscated\_password","obfuscated\_flag"**  - functions dealing with the obfuscated data
---
## 🔧 Reverse Engineering the Binary File

### **Step 2: Finding the Deobfuscation Function**

We can GDB (GNU-Debugger) to analyze the function calls:

```sh
gdb release
```
![Screenshot](https://i.imgur.com/cXs1zON.png)

Earlier we found out that the program has a function named **deobfuscate**, which deciphers the stored password and flag. To find the memory address, we run :

```sh
disassemble deobfuscate
```
![Screenshot](https://i.imgur.com/YP4dVQW.png)

We find a entity **Key** with the memory address "0x2020". 

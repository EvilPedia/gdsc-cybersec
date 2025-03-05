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
It can be installed by -
```
sudo apt install gdb
```
- [**Cyberchef**](https://gchq.github.io/CyberChef/) - An Online AIO Data Analysis Tool
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

### **Step 2: Finding the Deobfuscation Function**

I used GDB (GNU-Debugger) to analyze the function calls:

```sh
gdb release
```
![Screenshot](https://i.imgur.com/cXs1zON.png)

Earlier we found out that there are 3 variables related to the obfuscation of the flag/password, to find more info about that we run the command :
```sh
info variables
```
![Screenshot](https://i.imgur.com/lMsIcVJ.png)

From the output, we found these 3 entities along with their memory addresses:

- **XOR Key:** `0x2020`
- **Obfuscated Password:** `0x2040`
- **Obfuscated Flag:** `0x2060`

### **Step 3: Dumping hex values from the Functions:**


To inspect their contents, we run:

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
### **Step 4: Recognizing the Obfuscation encryption :**
After analyzing the key, we find that the key is 7 letters long (ignoring the 0x00 character). 

As said in the problem that a basic-obfuscation method was used to encrypt this file which means that most probably this was a repeated-XOR-encrypted file, where the 7 letter of the key keep repeating until the full flag/password is deciphered.

### **Step 5: Decrypting the flag/password using the XOR Key:**

We can now use [Cyberchef](https://gchq.github.io/CyberChef/) to decrypt the file using the XOR operation.

#### To decrypt the  password :
- Step 1: Paste the hex data of `<obfuscated_password>` in the input field.
- Step 2 : Drag and drop the `From Hex` function from the Operations Tab.
- Step 3 : Drag and drop the `XOR` function from the Operation Tab.
- Step 4 : Paste the hex data of the `<KEY>` in the Key field under the `XOR` function.

![Screenshot](https://i.imgur.com/f2G5iui.png)

If we look at the output, we find that Cyberchef has decoded the password and has found it to be `bt8uA1uxF350yZLuto9GmqTWJBpP2jUq` .

I repeated the same steps to find the decrypted data for `<obfuscated_flag>` :

![Screenshot](https://i.imgur.com/fOs6TTa.png)

To double check, if the flag decrypted was right or not, we can enter the password when prompted :
![Screenshot](https://i.imgur.com/5D5Eqqu.png)

**Found Password:**

```
bt8uA1uxF350yZLuto9GmqTWJBpP2jUq
```
 **Found Flag:**

```
gdsc{unp4ck1n6_b1n4r135_15_n4u6h7y}
```


---
# Conclusion

- **GDB** helped us to find the functions, the memory of those functions, and the values stored inside the function in the form of hex-values.
- **XOR encryption** is a common obfuscation method, easily reversible.
- **Cyberchef** helped us to decrypt the `<obfuscated_password` and the `<obfuscated_flag>` functions by using the hex data from the  `<KEY>` function.

#### Solving this CTF Challenge required binary analysis, reverse engineering and cryptographic skills.

---

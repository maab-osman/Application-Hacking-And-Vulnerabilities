# H5 - Hey its Alive!

## a) Vulnerability Investigation Report: Tapo C200 Firmware

In this exercise, my task was to investigate the TP‑Link Tapo C200 camera firmware without having physical access to the device. The goal was to obtain the firmware, decrypt it, and analyze its contents to identify potential security issues.

Since I did not have the camera itself, I relied entirely on publicly available firmware and reverse engineering tools on my Debian virtual machine.

---

## Obtaining the firmware

The first step was finding the firmware file.

Based on the research article (Margaritelli, 2025), TP‑Link stores firmware in a public AWS S3 bucket. This means anyone can list and download firmware files without authentication.

I installed the AWS CLI tool on my Debian VM and used the following command:

```
aws s3 ls s3://download.tplinkcloud.com/ --no-sign-request --recursive
```
This command listed a very large number of firmware files for many TP‑Link devices as seen below:

<img width="1228" height="546" alt="Screenshot 2026-02-22 at 12 02 34 AM" src="https://github.com/user-attachments/assets/bafb12a5-e43b-4ef8-8f35-af4b0748ef7c" />
At first, I felt overwhelmed because the output was extremely long and difficult to read.

So I filtered the results to find the Tapo C200 firmware:

```
aws s3 ls s3://download.tplinkcloud.com/ --no-sign-request --recursive | grep C200
```
This helped me locate the correct firmware file.


Then I downloaded it using:

```
aws s3 cp s3://download.tplinkcloud.com/firmware/Tapo_C200v3_en_1.4.2_Build_250313_Rel.40499n_up_boot-signed_1747894968535.bin

```
And then I moved it to my my downloads as seen below:

<img width="1224" height="91" alt="Screenshot 2026-02-22 at 12 03 52 AM" src="https://github.com/user-attachments/assets/777e660f-f5d0-459c-b8cb-c040f2eb0cd2" />


## Decrypting the firmware

After downloading the firmware, I tried to analyze it using binwalk. `binwalk` is a an open-source command-line tool designed for analyzing, reverse-engineering, and extracting data from binary files, specifically firmware images (Kali, 2026).

```
binwalk Tapo_C200v3_en_1.4.2_Build_250313_Rel.40499n_up_boot-signed_1747894968535.bin

```
However, binwalk could not detect a filesystem as seen below.

<img width="1001" height="125" alt="Screenshot 2026-02-22 at 12 10 35 AM" src="https://github.com/user-attachments/assets/910aea33-0b87-48c3-8aef-12e600e1019a" />


This made me realize the firmware was encrypted which was mentioned in the article earlier (Margaritelli, 2025).


## Installing tp‑link‑decrypt tool

Following the article, I downloaded the tp‑link‑decrypt tool from GitHub in order to decrypt the firmware:

```
git clone https://github.com/robbins/tp-link-decrypt

```
But when I tried to compile it using:

```
make
```
I faced errors. This was challenging and it resorted me to use some AI-assitance (OpenAI GPT-4) and according to it the errors were caused by:

  - missing dependencies
  - missing permissions

To fix it, I began by installing required packages:

```
sudo apt install build-essential libssl-dev
```

Then I gave permissions to scripts:

```
chmod +x preinstall.sh
chmod +x extract_keys.sh
```
After that I ran:
```
./preinstall.sh
./extract_keys.sh
```
This script downloaded TP‑Link GPL source code and extracted encryption keys.

Finally, I compiled successfully using `make`

This created `obj/tp-link-decrypt.o` which is the compiled object file and `bin/tp-link-decrypt` which is the final executable that we need for the next step. 
        

## Decrypting and Analyzing the firmware

Next, I decrypted the firmware by running:

```
./bin.tp-link-decrypt \ ~/Downloads/Tapo_C200v3_en_1.4.2_Build_250313_Rel.40499n_up_boot-signed_1747894968535.bin
```
The tool basically opened the `.bin` file, parsed the firmware header and detected `RSA-2048` which a signature verification used by the firmware.
The tool then wrote 
```
Tapo_C200v3...bin.dec
```
This is a raw decrypted firmware image that is ready for filesystem extraction.
Now to analyze, I finally ran:

```
binwalk -e firmware.bin.dec
```
This time, it showed useful information as seen below:

<img width="1226" height="550" alt="Screenshot 2026-02-22 at 12 44 48 AM" src="https://github.com/user-attachments/assets/bcb19c96-9476-4b67-acaa-3ccee3bd49cf" />

It revealed things like the linux kernel, SquashFS filesystem and the bootloader.

Then the contents where extarcted by:

<img width="1195" height="509" alt="Screenshot 2026-02-22 at 12 48 44 AM" src="https://github.com/user-attachments/assets/81bc3024-030b-4a33-ab1f-a98c44262ee3" />
 

```
binwalk -e firmware_decrypted.bin
```
## Exploring firmware contents
Inside the extracted folder, I found system files, binaries and configuration files. This confirmed the camera runs embedded Linux.
I then navigated to `squashfs-root` which is the root file system and saw various folders as seen below:

<img width="1013" height="63" alt="Screenshot 2026-02-22 at 1 06 10 AM" src="https://github.com/user-attachments/assets/b7583f73-9a12-45a1-95be-9fc4a9634db8" />

To begin my analysis, I first ran 

```
strings bin/main | grep -i password
```
This resulted in:

<img width="1329" height="591" alt="Screenshot 2026-02-22 at 1 13 57 AM" src="https://github.com/user-attachments/assets/863ff196-26a9-4130-bf5b-1cc5982e1485" />


Thats when things got interesting so I decided to load `bin/main` into Ghidra to trace the cross-references (XREFS) of password is %s. This allowed me to see the actual function calls where the password variable was being passed to a logging function, confirming that the data was not hashed before being recorded. From my analysis, I discovered several important things.

I began by filtering out defined strings containing `pass`
<img width="945" height="594" alt="Screenshot 2026-02-23 at 2 15 19 AM" src="https://github.com/user-attachments/assets/bcc7d7f5-2265-4b1c-93f0-724fbd2115f5" />


This led me to locate the "brain" of the user management system. I identified function `FUN_004b9d68`, which acts as a central registry for all administrative actions.
  - This function registers critical commands such as `change_admin_password` and `check_user_info`.
  - Identifying this helped me to pinpoint exactly which functions handle credential changes and authentication checks.
    
<img width="700" height="576" alt="Screenshot 2026-02-23 at 2 16 54 AM" src="https://github.com/user-attachments/assets/d4d3fc56-42f4-4d80-b867-1d8c0d1297fb" />

Once the management engine was identified, I analyzed how it validates user levels. I discovered a hardcoded logic flaw in the authentication flow.
  - The firmware performs a `strcmp` against the hardcoded string "Operator".
  - If the input matches "Operator," the system uses strcpy to manually overwrite the user's privilege level to "admin".
    
    <img width="492" height="235" alt="Screenshot 2026-02-23 at 2 19 09 AM" src="https://github.com/user-attachments/assets/7e303dbf-b0aa-487e-a1f3-67967f487689" />

Finally, I looked at the very beginning of the program's execution to see where these secrets originate.


<img width="519" height="281" alt="Screenshot 2026-02-23 at 2 21 45 AM" src="https://github.com/user-attachments/assets/8a29d102-390e-4bbb-bc09-4adbf9abf20c" />


  - In the `_INIT_90 function` , the firmware initializes its environment.
  - I identified the initialization of a `factory_passwd` variable alongside default account names for "admin" and "root".
  - This reveals the existence of hardcoded factory-default credentials that remain present in the production firmware.

The investigation concluded that the Tapo C200 security model relies on hardcoded logic rather than secure cryptographic practices. By identifying the "Operator" role-elevation bypass and the plaintext logging of credentials, I have demonstrated that an attacker can gain full administrative control by exploiting these internal architectural flaws.   

## b) Lab1

After unziping the Lab1 file, I began by compiling the source code using the make command, which utilized the `-g` flag to include debugging symbols.

<img width="670" height="51" alt="Screenshot 2026-02-22 at 3 13 58 AM" src="https://github.com/user-attachments/assets/e5c127dd-2f82-4562-9d91-e618f8678765" />


I then rab `gdb` which is a debugger for C, to investigate why it wasn't completing its execution.

<img width="879" height="424" alt="Screenshot 2026-02-22 at 3 18 07 AM" src="https://github.com/user-attachments/assets/01ff105b-a7ec-410a-9176-e2ea78c112eb" />


The program prints `Khoor/#zruog1` and then immediately triggers a Segmentation fault.

GDB identified the crash at `print_scrambled (message=0x0)`. This confirmed that the program was trying to process a null pointer

<img width="916" height="177" alt="Screenshot 2026-02-22 at 3 19 42 AM" src="https://github.com/user-attachments/assets/af3af538-f1b2-488d-9776-d40bf8003d1f" />


To understand the state of the program before it failed, I set a breakpoint at the call for the "bad message". The first message was being handled correctly, but the second call passed a null variable `(bad_message)` to the function.

<img width="592" height="490" alt="Screenshot 2026-02-22 at 3 22 45 AM" src="https://github.com/user-attachments/assets/79482fb3-dbd1-4e34-a5fd-5ec22814592d" />

Based on the AI-assisted analysis of the crash, I modified the code to prevent the null pointer dereference.
I removed the direct call to `print_scrambled(bad_message)` and implemented an if statement to ensure the message is not null before processing it.

The logic of the program useses a +3 ASCII shift so the recovered flag is `Hello, World!`, Thefore By adjusting the value of i to 0, I recovered the intended text.
Also, to prevent the crash I added a conditional if statement to ensure the program only attempts to print a message if the pointer is not NULL.

After these changes seen below, the program was recompiled using `make`

<img width="711" height="682" alt="Screenshot 2026-02-22 at 3 38 56 AM" src="https://github.com/user-attachments/assets/49c6496c-93cf-4fe5-9848-5306193c711c" />

It now runs to completion without crashing and displays the correct flag.

<img width="772" height="69" alt="Screenshot 2026-02-22 at 3 40 00 AM" src="https://github.com/user-attachments/assets/5cacb50a-e846-4c45-a06e-865e9b78cf3f" />

## c) Lab 2 - passtr

After unziping and excecuting the script `./passtr`, The program prompts the user for a password and terminates if the input is incorrect.

<img width="601" height="99" alt="Screenshot 2026-02-22 at 3 51 57 AM" src="https://github.com/user-attachments/assets/6973dabb-e33a-44cb-a4ae-1f58dd8410fc" />


Instead of brute-forcing the input, I inspected the source code using the `cat` command, we see that the password is clearly hardcoded inside a condition.

<img width="1198" height="441" alt="Screenshot 2026-02-22 at 3 53 20 AM" src="https://github.com/user-attachments/assets/a6ea85ab-91dd-4fd7-bd17-d46cde89dafb" />


- The program uses `strcmp` to check the user's input against a fixed string.
- The password is explicitly hardcoded as: `sala-hakkeri-321`.

I re-ran the program and entered the discovered password. The `if` condition evaluated to true, granting access to the flag.

<img width="987" height="116" alt="Screenshot 2026-02-22 at 3 54 39 AM" src="https://github.com/user-attachments/assets/53440611-955d-4e4c-b451-003d7a94aed7" />

In conclusion, By simply reading the source code, the security mechanism was bypassed without the need for complex debugging or reverse engineering.

## d) Lab 3 - Noracrackmes

### crackme03

As the previous crackmes, you are expected to input one right argument.
I began by using the `cat` command to read the source code (crackme03.c). Unlike the previous lab where the password was a simple string, this program calculates the password by adding a mask to a base string
  - Base String (correct): "lAmBdA"
  - Mask Values (mask): {2, 3, 2, 3, 5}

<img width="945" height="824" alt="Screenshot 2026-02-22 at 4 16 05 AM" src="https://github.com/user-attachments/assets/b1405a0d-f002-4a90-a205-41e02715afab" />


The program basically compares the user input to `correct[i] + mask[i]`. By manually adding these values together using the ASCII table, the real password is revealed to be 'nDoEiA' as seen in the source code

<img width="815" height="143" alt="Screenshot 2026-02-22 at 4 15 42 AM" src="https://github.com/user-attachments/assets/a58e6d94-7b27-4f62-a04c-da061f6b6993" />

Inputting this string successfully clears the challenge.

<img width="876" height="61" alt="Screenshot 2026-02-22 at 4 16 27 AM" src="https://github.com/user-attachments/assets/24c5cc27-76a5-4617-ad4a-49c351c6347a" />

### crackme04

Unlike previous challenges, this program accepts any string that meets specific mathematical criteria rather than a single hardcoded word.

I inspected the source code using the `cat` command to understand the validation logic.

<img width="987" height="905" alt="Screenshot 2026-02-22 at 4 29 19 AM" src="https://github.com/user-attachments/assets/36837632-342e-4ffe-98df-8a459895d652" />


  - The input must be exactly 16 characters long.
  - The sum of the ASCII values of all 16 characters must equal a specific total.

I noticed a discrepancy in the code comments. One comment mentioned a sum of 1652, but the actual #define and logic used a different calculation:
  - 16 x 110 + 2 = 1762
  - Which tells us that the program requires a total ASCII sum of 1762.
    
To solve this, I needed a string where the characters average out to the letter 'n' (ASCII value 110), with two characters being slightly higher to reach the +2 requirement.
With the assitance of AI for accuraccy, the solution was concluded to be `nononnnnnnnnnnnn`
  - This string consists of 14 'n's (110) and 2 'o's (111)
  - (14 x 110) + (2 x 111) = 1540 + 222 = 1762.

By providing the string `nononnnnnnnnnnnn`, I satisfied both the length and sum requirements. The program confirmed the input was correct.


<img width="876" height="61" alt="Screenshot 2026-02-22 at 4 29 41 AM" src="https://github.com/user-attachments/assets/c61dc356-a284-4847-95cc-cbbca6dec758" />

---
# References
- Kali. “Binwalk | Kali Linux Tools.” Kali Linux,2026 www.kali.org/tools/binwalk/.
- Margaritelli, Simone. “TP-Link Tapo C200: Hardcoded Keys, Buffer Overflows and Privacy in the Era of AI Assisted Reverse Engineering.” Evilsocket, 17 Dec. 2025, www.evilsocket.net/2025/12/18/TP-Link-Tapo-C200-Hardcoded-Keys-Buffer-Overflows-and-Privacy-in-the-Era-of-AI-Assisted-Reverse-Engineering/. Accessed 21 Feb. 2026.
- TP-Link Cloud Public Firmware Repository. AWS S3 bucket: s3://download.tplinkcloud.com/ (Accessed via AWS CLI with --no-sign-request).
- OpenAI. (2026). ChatGPT (GPT-5) [Large language model]. Retrieved February 22, 2026, from https://chat.openai.com



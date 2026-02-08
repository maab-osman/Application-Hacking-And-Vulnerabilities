# H4: Some Disassembly Required

## x) Summary: Ghidra for Reverse Engineering (Hammond 2022)
* The video tackeles a reverse engineering challenge `Bbbbloat` under picoCTF
* Hammond prefers to run command line utilites `strace`which traces system calls and signals and `ltrace` which intercepts and records the dynamic library calls.
* `objdump -d` disassembles and breaks things, but he figured out its "stripped" which simply means that before releasing a program, developers often run strip which deletes the symbol table to save space and make reverse engineering harder.
* In Ghidra, the first thing done is to look for potentiat strings: Window > `Defined Strings`
* Under the `What's my favorite number?` string, we can find the disassembly view which showcases all of the things present in the binary such as functions
* It notes some cross references (XREF) for given data such as values. Clicking on that function `FUN` to a load effective address instruction.
* In the Decompile pane, We observe multiple variables that include hexadecimal values hinting to a string built somewhere.

<img width="394" height="263" alt="Screenshot 2026-02-05 at 10 19 18 AM" src="https://github.com/user-attachments/assets/65851806-e681-45c2-b8d1-6440585c71af" />

*Ghidra Decompiler view of the bbbloat main logic, showing the comparison of local_48 against a hardcoded hexadecimal value.*

* Hammond identifies a comparison where user input stored in `local_48 is checked` against a specific hexadecimal value.
* So if it returns true, it will run a function `FUN_00101249` BASED of `local_38`. And if its false, it returns `Sorry, that not it`
* Hammond tries pasting the `local_48` into python to understand that number in decimal.
* Running the `Bbbbloat` again, he enters the decimal number again and catches the flag

---

## a) Installation of Ghidra
* **Steps taken:**
    1. I downloaded the Ghidra stable release directly from the official GitHub repository as a compressed ZIP file.
    2. I unzipped the package into my local tools directory and verified the presence of the ghidraRun launch script.
    3. To streamline my workflow, I created a custom shell script `./run_ghidra` in my home directory that points to the Ghidra installation path.
    4. This allows me to launch the Ghidra without navigating to the specific installation folder each time.

```

#!/bin/bash

cd /opt/ghidra || { echo "Failed to navigate to /opt/ghidra"; exit 1; }

echo "Listing files in /opt/ghidra:"
ls

echo "Starting Ghidra..."

./ghidraRun

echo "Script completed."

```
* It has been successfully installed and launched.

<img width="860" height="453" alt="Screenshot 2026-02-05 at 11 10 14 AM" src="https://github.com/user-attachments/assets/4c759b81-1df6-40db-8102-5140fa733120" />


---

## b) rever-C: Reverse Engineering `packd`

* I first imported the `packd` binary, the unpacked version from the last homework into Ghidra to perform static analysis.
  <img width="790" height="280" alt="Screenshot 2026-02-07 at 7 59 04 PM" src="https://github.com/user-attachments/assets/0d539210-b225-4bd4-9def-5598de7e78d1" />
  
* One of the first things I did was to do an auto analysis on the file to Ghidra to map function boundaries and generate initial decompiler output.
* Then I on moved to select the defined strings from the Window menu, this is becasuse it  helps identify where the program interacts with the user and reveals hardcoded sensitive data.
* I found the prompt string: "What's the password?" which is refrenced to the main function.
  <img width="797" height="476" alt="Screenshot 2026-02-07 at 8 01 10 PM" src="https://github.com/user-attachments/assets/9897ffbe-3612-4437-86b5-0651de17d46e" />
  <img width="873" height="206" alt="Screenshot 2026-02-07 at 8 03 03 PM" src="https://github.com/user-attachments/assets/935f4607-5ac0-4885-b39b-b5f2dbcbaeb3" />

* By refrencing the XREFs, I was able to locate the `main` function and open it the decompiler to review the code.
* Reviewing thw code, `local_28` was renamed to `user_input_buffer` and iVar1 to `comparison_result`
* As we can see, The program uses scanf to read a string from the user and stores it in local_28
    <img width="853" height="399" alt="Screenshot 2026-02-07 at 8 06 04 PM" src="https://github.com/user-attachments/assets/e31baa5f-1c77-42f4-839e-d26c9224908c" />
* It then utilizes the `strcmp` function to compare the user's input against the hardcoded string "piilos-AnAnAs".
* If strcmp returns 0, the program executes the success branch to print the flag.
* So the required password in this scenario is identified through decompilation is `piilos-AnAnAs`, resulting in the flag.


---

## c) If Backwards: Modifying `passtr` Binary
* Similar to the previous task, The `passtr` program was imported and analyzed using Ghidra.

<img width="658" height="327" alt="Screenshot 2026-02-07 at 8 53 41 PM" src="https://github.com/user-attachments/assets/7ebc1008-0028-44e4-8449-314d3dcd551c" />

* Similar to the previous task, The `passtr` program was imported and analyzed using Ghidra.
* Again looking at the defined strings, we follow The `What's the password` which navigates us to the `main` function.
* In the main function decompile window as seen below, A local_c is set to 1 only if the XOR-decryption loop matches the stored password.
<img width="575" height="381" alt="Screenshot 2026-02-07 at 9 25 20 PM" src="https://github.com/user-attachments/assets/6d07f95f-a4dd-4b62-a386-da8ab05c7dd1" />

* To modify it, we need to identify the jump instructions in the binary. The first one found is `0x001011f6: JZ` which triggers a failure if the password flag is 0.
* I then modified JZ to JNZ which causes the jump to fail when the password is right.
<img width="778" height="180" alt="Screenshot 2026-02-07 at 9 03 13 PM" src="https://github.com/user-attachments/assets/efd7f8ac-8afa-4188-a1e2-8666ab6b47b4" />
* I exported the file into its original format. Then ran the program again on my terminal but to no avail. This did not work.

<img width="667" height="328" alt="Screenshot 2026-02-07 at 8 54 23 PM" src="https://github.com/user-attachments/assets/55120059-2a6e-4924-ba2e-15b6e752b18f"/>

* Working my way back to the main function decompiler, I noticed there is also, a `strlen` call verifying the input is exactly 16 characters (0x10).

<img width="581" height="126" alt="Screenshot 2026-02-07 at 9 44 00 PM" src="https://github.com/user-attachments/assets/8460c74a-3256-4c33-bca6-6522d59a5127" />

* Going back to the code, we can identify a second jump at `0x00101208: JNZ`, which triggers a failure if the length is not 16.
<img width="778" height="180" alt="Screenshot 2026-02-07 at 9 03 13 PM" src="https://github.com/user-attachments/assets/efd7f8ac-8afa-4188-a1e2-8666ab6b47b4" />


* I then patched the lenght check JNZ to JZ which causes the prohram to jump Failure only if the length is exactly 16.
  
* Exporting the program again and running it, we type in the correct password from last week's excercise: `sala-hakkeri-321`. We get the results we want, denial as seen below.
  
 <img width="667" height="328" alt="Screenshot 2026-02-07 at 8 54 23 PM" src="https://github.com/user-attachments/assets/55120059-2a6e-4924-ba2e-15b6e752b18f"/>
 
<img width="641" height="199" alt="Screenshot 2026-02-07 at 10 04 10 PM" src="https://github.com/user-attachments/assets/b65c0db0-8ba6-44a2-a66f-2df9d53f3ef7" />
 
* On the other hand, the wrong passwords give us  a successful outcome solving this tasks objective.

 <img width="666" height="222" alt="Screenshot 2026-02-07 at 8 59 23 PM" src="https://github.com/user-attachments/assets/8cfa5910-0667-4d2f-b811-419dfce97dbe" />


---

## d) Nora CrackMe: Preparation
* **Setup:*Cloned the repository and compiled binaries using `make`.
<img width="779" height="350" alt="Screenshot 2026-02-08 at 9 14 42 AM" src="https://github.com/user-attachments/assets/8216a25f-fe7e-4fee-9a4b-eb3bd8574a7c" />

  
* **Approach:** Used Ghidra for static analysis of the binaries to find password logic without viewing the `.c` source files.

---

## e) Nora crackme01
<img width="766" height="352" alt="Screenshot 2026-02-08 at 9 15 08 AM" src="https://github.com/user-attachments/assets/e3c8ce9a-6566-473c-bb4c-933fe34de65a" />
<img width="783" height="66" alt="Screenshot 2026-02-08 at 9 15 27 AM" src="https://github.com/user-attachments/assets/75d846d5-ea49-42a4-83de-332928055655" />

<img width="601" height="207" alt="Screenshot 2026-02-08 at 9 17 36 AM" src="https://github.com/user-attachments/assets/f67043a0-3dbd-43e3-83fe-43e2f6efdd43" />
<img width="472" height="401" alt="Screenshot 2026-02-08 at 9 18 06 AM" src="https://github.com/user-attachments/assets/1c8ad59e-db10-4caa-bc6f-2c23b5c43751" />
<img width="843" height="145" alt="Screenshot 2026-02-08 at 9 19 02 AM" src="https://github.com/user-attachments/assets/8a3b1c57-1262-40b9-b2d6-4f5caca7cbba" />




* **Analysis:** Viewed the binary in Ghidra.
* **Logic:** [Explain: e.g., Simple string comparison using strcmp.]
* **Solution:** The password is: `[Insert Password Here]`

---

## e) Nora crackme01e

<img width="821" height="143" alt="Screenshot 2026-02-08 at 9 19 32 AM" src="https://github.com/user-attachments/assets/56a75e28-643c-4bd5-a4da-8e50655f5c2c" />
<img width="522" height="437" alt="Screenshot 2026-02-08 at 9 27 43 AM" src="https://github.com/user-attachments/assets/b2efd677-4300-4106-880b-47a4dad0428b" />


<img width="834" height="76" alt="Screenshot 2026-02-08 at 9 21 00 AM" src="https://github.com/user-attachments/assets/a231207d-d027-4355-b8b3-05412510d0d7" />

* **Analysis:** [Explain any difference from 01, e.g., environmental variables or hidden logic.]
* **Solution:** The password is: `[Insert Password Here]`

---

## f) Nora crackme02

* **Variable
<img width="548" height="453" alt="Screenshot 2026-02-08 at 9 20 10 AM" src="https://github.com/user-attachments/assets/22999135-189e-441d-b497-c50539059ce3" />
<img width="527" height="444" alt="Screenshot 2026-02-08 at 9 21 38 AM" src="https://github.com/user-attachments/assets/ed12ad78-3c3e-4f2b-a53e-de2f05cc8f72" />


 <img width="817" height="103" alt="Screenshot 2026-02-08 at 9 27 12 AM" src="https://github.com/user-attachments/assets/98812f08-f265-45f2-90e6-0f639e9f7175" />


Renaming:**
    * `param_1` -> `argc`
    * `param_2` -> `argv`
    * `iVar1` -> `comparison_result`
* **Program Operation:** [Explain: e.g., This binary checks if the user provided an argument and compares it against a calculated value.]
* **Solution:** The password is: `[Insert Password Here]`
strcmp function in c
strlen
---

## Sources
* Hammond, J. (2022). *Ghidra for Reverse Engineering*. Available at: https://youtu.be/fTGTnrgjuGA.
* Tindall, S. (2023). *NoraCodes / crackmes*. Available at: https://github.com/NoraCodes/crackmes.

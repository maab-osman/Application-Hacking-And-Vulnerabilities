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
* **Ghidra Analysis:**
    * Imported the `packd` binary (unpacked version from previous task).
    * Located the `main` function in the Symbol Tree.
* **Descriptive Variable Names:**
    * `local_x` renamed to `input_buffer`.
    * `local_y` renamed to `stored_password_hash` (or relevant logic).
* **Program Operation:** * [Briefly explain: e.g., The program takes string input, compares it against the hardcoded string "piilos-AnAnAs" using a loop, and prints the flag if they match.]
* **Solution:** The password found in the decompiled logic is `piilos-AnAnAs`.


---

## c) If Backwards: Modifying `passtr` Binary
* **Objective:** Patch the binary so it accepts *wrong* passwords and rejects the *correct* one.
* **Method:**
    1.  Opened `passtr` in Ghidra.
    2.  Located the comparison logic (e.g., a `JZ` (Jump if Zero) or `JNZ` instruction).
    3.  **The Patch:** Used the "Patch Instruction" feature to flip the logic (e.g., changing `JZ` to `JNZ`).
* **Testing:**
    * Input "wrong_password": [Expected: Access Granted]
    * Input "sala-hakkeri-321": [Expected: Access Denied]


---

## d) Nora CrackMe: Preparation
* **Setup:** Cloned the repository and compiled binaries using `make`.
* **Approach:** Used Ghidra for static analysis of the binaries to find password logic without viewing the `.c` source files.

---

## e) Nora crackme01
* **Analysis:** Viewed the binary in Ghidra.
* **Logic:** [Explain: e.g., Simple string comparison using strcmp.]
* **Solution:** The password is: `[Insert Password Here]`

---

## e) Nora crackme01e
* **Analysis:** [Explain any difference from 01, e.g., environmental variables or hidden logic.]
* **Solution:** The password is: `[Insert Password Here]`

---

## f) Nora crackme02
* **Variable Renaming:**
    * `param_1` -> `argc`
    * `param_2` -> `argv`
    * `iVar1` -> `comparison_result`
* **Program Operation:** [Explain: e.g., This binary checks if the user provided an argument and compares it against a calculated value.]
* **Solution:** The password is: `[Insert Password Here]`

---

## Sources
* Hammond, J. (2022). *Ghidra for Reverse Engineering*. Available at: https://youtu.be/fTGTnrgjuGA.
* Tindall, S. (2023). *NoraCodes / crackmes*. Available at: https://github.com/NoraCodes/crackmes.

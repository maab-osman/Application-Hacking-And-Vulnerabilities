# H6 - GNU Debugger

## a) Dynamic Analysis: LLDB for Rust

In this exercise, I transitioned from using `rust-gdb` to `rust-lldb` on my ARM-based MacBook Air. My goal was to replicate the classroom debugging examples and evaluate if LLVM-based tools provide a smoother experience on modern Apple hardware.

This report is based on this repository: https://github.com/hhgitmax/gdb-lecture

Since Rust projects have a specific structure, my first task was ensuring I was in the correct directory so the debugger could "map" the source code to the binary.

Navigating to the project:

```
cd ~/gdb-lecture/concat
ls
# Confirmed presence of Cargo.toml and src/
```

I ran `cargo build` to ensure the debug symbols were up to date.
Instead of standard lldb, I used the Rust wrapper to ensure pretty-printing of Rust types:

```
rust-lldb ./target/debug/concat
```

## Task 1: Watching a Value Change in a Loop

In the `concat project`, I wanted to observe the variable n as it was modified inside a while loop.

Setting a breakpoint at the start of the function and running to the breakpoint:

```
(lldb) b concat
(lldb) run
```

Once stopped inside the function, I set a hardware watchpoint on the variable `n`. As mentioned in the class lecture, this tells the CPU to pause the program whenever the memory address of `n` is written to. On ARM64 architecture, this is handled by dedicated hardware debug registers (ARM, 2021).

```
(lldb) watchpoint set variable n
```
I used the continue command. The program automatically paused every time line 12 (n -= 1) executed.
So on the first iteration `n` changed from 5 to 4 then second one `n` changed from 4 to 3 as seen below:

<img width="788" height="409" alt="Screenshot 2026-03-01 at 1 55 48 AM" src="https://github.com/user-attachments/assets/89633b98-464b-4f88-8db3-5de7779a7c9d" />

## Task 2: Modifying a Function Argument

I transitioned to the `check-pin` project to bypass the password check.

When I reached the breakpoint at line 12 (if check_password(pin, PIN)), I tried to force my input to match the secret by running expression pin = PIN.

I learned that in Rust, const values are often inlined by the compiler. Because they are not stored as named symbols in the final binary, the debugger cannot "see" the name PIN (Rust Foundation, 2024).

I ran l 1 to list the source code from the beginning. I discovered that in my local version of the file, the constant was actually defined as const PIN: usize = 1235;. 

<img width="484" height="174" alt="Screenshot 2026-03-01 at 2 33 12 AM" src="https://github.com/user-attachments/assets/9a343811-dc27-46c8-ba71-f9c9ebf87ef9" />

Once I identified the correct PIN, I used the debugger to align my input with the secret:

```
(lldb) expression pin = 1235
(lldb) register write x0 1235
(lldb) c
```
By matching my input in x0 to the actual constant in x1, the program finally returned the "Access granted!" message.

<img width="347" height="56" alt="Screenshot 2026-03-01 at 2 34 21 AM" src="https://github.com/user-attachments/assets/b9343f77-99a2-46dc-9e5f-178c38ab5d96" />

## Task 3: Modifying a Return Value

Finally, I practiced overriding a function's result to bypass logic entirely, regardless of the input PIN.

I stepped into the `check_password` function and ran the finish command to execute until the point of return.

In Rust on ARM64, a boolean true is represented as a 1 stored in register x0. Even if the function calculated false (0), I manually overwrote it:

```
register write x0 1
```
The main function received the "true" signal and proceeded to grant access as seen below:

<img width="385" height="76" alt="Screenshot 2026-03-01 at 2 37 13 AM" src="https://github.com/user-attachments/assets/f4c8dc43-7185-4585-b4a7-1545b49bb31c" />

## Task 4: Comparison: LLDB vs. GDB
- Having used GDB on Linux previously, rust-lldb felt like moving to a much stricter, but more capable, environment.
- While GDB is generally more "relaxed" with commands letting me get away with a simple list, LLDB was far more strict about syntax requiring explicit `read/write` modifiers for registers (LLVM Project, 2024).
- I initially struggled with errors like "invalid command" when I forgot to specify read or write for registers.
- The most personal takeaway wasn't until I used l 1 and register read x1 that I saw the the constant in my local file had actually been set. LLDB sort of corrected my own misinformation about the code I was looking at.
- While I appreciate GDB’s simplicity, LLDB’s expression command is significantly more powerful for Rust. The ability to evaluate complex expressions quickly and the native way it handles ARM64 registers makes me favor it over gdb.


---

### References
- ARM (2023). Procedure Call Standard for the Arm 64-bit Architecture (AArch64). [online] Available at: https://github.com/ARM-software/abi-aa.
- hhgitmax (2026). GDB Lecture Repository. [online] GitHub. Available at: https://github.com/hhgitmax/gdb-lecture.
- LLVM Project (2024). LLDB Debugger Guide. [online] Available at: https://lldb.llvm.org.
- Rust Foundation (2024). The Rust Programming Language: Optimization and Constants. [online] Available at: https://doc.rust-lang.org/book/.

---
## b) Lab0

- In this introductory lab, I utilized gdb to diagnose errors in a C program.
- Upon execution, the program produced the following unexpected output:

<img width="828" height="190" alt="Screenshot 2026-03-01 at 3 31 47 AM" src="https://github.com/user-attachments/assets/55f856e7-35d1-4115-a7e7-b31ff4f97c2d" />

- I launched the debugger to inspect the program's state at runtime. Unlike my earlier work with `rust-lldb`, I used the standard GNU Debugger for this C-based task.
- I compiled the program with the -g flag to include debug symbols and launched the session:
```
gdb ./buggy_program
```

- I set a breakpoint at the entry point and stepped through the logic:
```
(gdb) break main

(gdb) run

(gdb) next
```
By stepping through the execution, I isolated the bug to the following function call:
<img width="811" height="179" alt="Screenshot 2026-03-01 at 3 34 05 AM" src="https://github.com/user-attachments/assets/efb6d779-95a4-4a48-8812-197ad8bc1bd2" />

- The error was clearly commented, hinting that the array was off by one in the loop boundary condition.
- Understanding this, I navigated to the C code file:

<img width="940" height="363" alt="Screenshot 2026-03-01 at 3 35 52 AM" src="https://github.com/user-attachments/assets/76be482a-4cbc-4e75-9e47-ca52759d7632" />

- Here we can see that on the final iteration, the program attempted to read arr[5], which points to a memory address immediately following the array's allocated space.
- I corrected the loop boundary to ensure it strictly respects the array's size by changing the condition from `<=` to `<`
- After recompiling and running the program, the invalid "Element 5" was eliminated, and the output correctly reflected the array’s intended bounds.

  <img width="687" height="154" alt="Screenshot 2026-03-01 at 3 38 22 AM" src="https://github.com/user-attachments/assets/198a75e4-a094-4e0e-8bf6-318b1426af19" />

---


## c) Lab1

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

---

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


# Dynamic Analysis: LLDB for Rust

In this exercise, I transitioned from using rust-gdb to rust-lldb on my ARM-based MacBook Air. My goal was to replicate the classroom debugging examples and evaluate if LLVM-based tools provide a smoother experience on modern Apple hardware.

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

Once stopped inside the function, I set a hardware watchpoint on the variable `n`. As mentioned in the class lecture, this tells the CPU to pause the program whenever the memory address of `n` is written to.

```
(lldb) watchpoint set variable n
```
I used the continue command. The program automatically paused every time line 12 (n -= 1) executed.
So on the first iteration `n` changed from 5 to 4 then second one `n` changed from 4 to 3 as seen below:

<img width="788" height="409" alt="Screenshot 2026-03-01 at 1 55 48 AM" src="https://github.com/user-attachments/assets/89633b98-464b-4f88-8db3-5de7779a7c9d" />

## Task 2: Modifying a Function Argument

I transitioned to the `check-pin` project to bypass the password check.

When I reached the breakpoint at line 12 (if check_password(pin, PIN)), I tried to force my input to match the secret by running expression pin = PIN.

I learned that in Rust, const values are often inlined by the compiler. Because they are not stored as named symbols in the final binary, the debugger cannot "see" the name PIN.

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
- While GDB is generally more "relaxed" with commands letting me get away with a simple list, LLDB was far more strict about syntax.
- I initially struggled with errors like "invalid command" when I forgot to specify read or write for registers.
- The most personal takeaway wasn't until I used l 1 and register read x1 that I saw the the constant in my local file had actually been set. LLDB sort of corrected my own misinformation about the code I was looking at.
- While I appreciate GDB’s simplicity, LLDB’s expression command is significantly more powerful for Rust. The ability to evaluate complex expressions quickly and the native way it handles ARM64 registers makes me favor it over gdb.





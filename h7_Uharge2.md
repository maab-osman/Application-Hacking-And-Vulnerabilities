# H7: Cryptographic Foundations and Breaking XOR

In this assignment, I explored the foundations of cryptography through the lens of ordinary programming errors. By implementing and breaking simple XOR ciphers, I practiced the "Cryptopals" methodology of understanding encryption through active exploitation.

## x) Summaries
### 1.1 Foundations: Terminology (Schneier, 2015)

- Cryptography is the art of keeping messages secure by transforming plaintext into unreadable ciphertext through encryption.
- Authentication = reciver ascertain it's origin, Integrity = reciver verifies it has not been modified in transit, Nonrepudiation = sender can't falsely deny sending later
- Security should reside in the key, not the secrecy of the algorithm.
- Symmetric algorithims = encryption key can be cakcuated from decryption key and vice versa.
- Public key algorithims (Asymmetric algorithms) = encryption `public` key is different from decryption `private` key
- There are four general types of cryptanalytic attacks.
  1. Ciphertext-only attack : has ciphertext of severa messages
  2. Known-plaintext attack : has both ciphertext and plaintext of those messages
  3. Chosen-plaintext attack : has both ciphertext and plaintext and chooses specific plaintext that gets encrypted
  4. Adaptive-chosen-plaintext attack : can modify his choice based on the results of previous encryption
- Complexity is measured in 3 ways: Data complexity, Processing complexity and storage requirements.
- Restricted algorithims are historically used but dangerous in modern contexts because they cannot be easily replaced if compromised.
  
### 1.4 Simple XOR (Schneier, 2015)
- A standard bitwise operation where the result is 1 if the bits are different and 0 if they are the same.
- It is a symmetric algorithim
- XOR is uniquely useful because it is its own inverse.
- Simple XOR is trivial to break because the key length is often shorter than the message, leading to repeating patterns.
  
### 1.7 Large Numbers (Schneier, 2015)
- Cryptography deals with numbers so large they are difficult to conceptualize.
- Security is often based on the sheer mathematical impossibility of a brute force search within the lifetime of the universe.

### Python Basics for Hackers (Karvinen, 2024)

- While the built-in Python REPL is functional, iPython is better for development.
- The workflow centers on writing scripts and hitting F5 to run them immediately.
- For crypto tasks, the ability to jump between hex, bytes, and base64 using built-in methods like .fromhex() is important.
- Python handles these operations natively on integers and byte-arrays.
- Using assert is a professional way to verify that your code is working as expected.
- Print Debugging for quick checks and Breakpoints for deep inspection of the program state (similar to the GDB/LLDB workflows used in previous week's lab).
- Python’s list comprehensions allow for "loopier loops" that are both shorter and more readable. 

## Task a: 1. Convert Hex to Base64
- I started by opening python3 in my terminal. I knew I needed a library for Base64, so I tried import base64. To see what I could actually do with it, I typed base64. and hit Tab.
- It gave me a few options: base64.b64decode, base64.b64encode, etc.
- Since I wanted to encode the data into Base64, b64encode seemed like the obvious choice.
- I first tried to just throw the hex string directly into the function:
  
  ```
  base64.b64encode("49276d206b696c6c696e...")
  ```
- But I got a TypeError. I learned that b64encode doesn't want a string, it wants bytes.
- I had to figure out how to turn those hex characters into actual binary data using `bytes.fromhex()`.

  ```
  theString = "49276d206b696c6c696e6720796f757220627261696e206c696b65206120706f69736f6e6f7573206d757368726f6f6d"
  converted_binary = bytes.fromhex(theString)
  ```
- Now that I had the data in the right format, I fed it back into the base64 tool I found earlier.
  
  <img width="1205" height="371" alt="Screenshot 2026-03-08 at 7 19 34 AM" src="https://github.com/user-attachments/assets/e2c05b2c-9c60-4d23-8002-4d9d9a481217" />


<img width="1383" height="217" alt="Screenshot 2026-03-08 at 7 19 57 AM" src="https://github.com/user-attachments/assets/dc238edb-9ebc-4d95-bacb-a2ac2eca8f0e" />


  ---

## Task b: 2. Fixed XOR

- Here, the goal was to take two same lenght hex strings and XOR them against each other.
- Just like in Task A, I knew I couldn't XOR the "text" of the hex strings. I had to turn both inputs into raw bytes first using `bytes.fromhex()`
- I tried doing `bytes1 ^ bytes2` but Python gave me another error. It turns out you can't XOR at once. You have to go through them one by one.
- The teacher's tip was to use `enumerate()` to iterate through two sets of bytes at once.
- I also used `bytearray` to collect the results of each XOR operation, as it is mutable and efficient for building binary string

<img width="918" height="594" alt="Screenshot 2026-03-08 at 7 18 45 AM" src="https://github.com/user-attachments/assets/fd1318b3-4d1c-42be-9c21-fb1ed11a501d" />

- After adding a `print(fixed_xor(input1, input2))` statement at the bottom of my script, I successfully verified the hex output matched the target

<img width="825" height="138" alt="Screenshot 2026-03-08 at 7 19 01 AM" src="https://github.com/user-attachments/assets/9a1e766e-8f56-4a78-ae3e-3607bd24fc74" />

---

## Tasck c: 3. Single-byte XOR cipher

- The idea here is that we have a hex string, and someone XOR'd every single character in the original message against one secret key. We don't know the key.
- We try every single possible key, look at the 256 resulting messages, and pick the one that looks like English.
- To knoe if its english, a scoring method can be used.

```
def score_english(input_bytes):
    # A list of common English characters to look for
    # We include lowercase, uppercase, and the space
    common_chars = b"etaoinshrdlu ETAOINSHRDLU"
    score = 0
    for byte in input_bytes:
        if byte in common_chars:
            score += 1
    return score

def crack_single_byte_xor(hex_str):
    ciphertext = bytes.fromhex(hex_str)
    
    best_score = 0
    best_plaintext = ""
    best_key = None

    # Try every possible byte (0 to 255) as a key
    for key_candidate in range(256):
        # XOR every byte in the ciphertext with our candidate key
        current_attempt = bytes([byte ^ key_candidate for byte in ciphertext])
        
        # Check if this attempt looks like English
        current_score = score_english(current_attempt)
        
        # If this is the best English we've seen so far, save it!
        if current_score > best_score:
            best_score = current_score
            best_plaintext = current_attempt
            best_key = key_candidate

    return best_plaintext, best_key

# The hex string from Cryptopals Challenge 3
target_hex = "1b37373331363f78151b7f2b783431333d78397828372d363c78373e783a393b3736"

message, key = crack_single_byte_xor(target_hex)
print(f"Key found: {chr(key)}")
print(f"Decoded Message: {message}")
```

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


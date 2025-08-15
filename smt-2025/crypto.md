## Cryptography CTF Challenge

Part of SMTP 2025

Written by fredora

## welcome
Description
``` 
This message has been... encoded? encrypted? Whatever that is... Anyways, can you read it?

UkxTMjAyNXt2M2tibmwzX3MwX3NnM19icXhvczBfdjBxa2MhfQ==
```

The double equal signs at the end of the string highly suggest that it's a base64 encoded flag. Decoding the base64 will still return gibberish:

```
RLS2025{v3kbnl3_s0_sg3_bqxos0_v0qkc!}
```

But we know the flag starts with the usual 'SMT2025' string, so RLS2025 is just a shifted version of the flag (shifted 1 bit to the right). Decrypting that will return:

```
SMT2025{w3lcom3_t0_th3_crypt0_w0rld!}
```

## eAESy

Description
```
This is a basic implementation of AES. Can you decrypt the message?
```

The source code given to us is this:

```
import os
from Crypto.Cipher import AES					# pip install pycryptodome
from Crypto.Util.Padding import pad, unpad

FLAG = b'SMT2025{REDACTED}'
key = b'smt_progr4m_2k25'

# Function to encrypt the flag
def encrypt(msg, key):
	iv = os.urandom(AES.block_size)
	cipher = AES.new(key, AES.MODE_CBC, iv)
	ciphertext = cipher.encrypt(pad(msg, AES.block_size))	# AES block size is 16 bytes
	return (iv + ciphertext).hex()

# Write the encryption result to flag.enc
with open("flag.enc", "w") as f:
	f.write(encrypt(FLAG, key))

# Encrypted FLAG (from flag.enc): 4abd733e0662e1b1aa3d1996f789209b5c9684941f1717d05000e861cab8491bef153fffd9feb7502ba96144d061517a134456204eb06d250f5266e0e9945c76
```

We have a CBC mode AES encryption mechanism. CBC is a mode in block cipher where the plaintext is divided into an equal sized block, then it'll be encrypted through XOR-ing the output of the last block with the current block's input before the last XOR operation with the actual key. This helps obfuscate patterns in block ciphers so that the encryption process will return random data even if it's encrypted using the same key. When the plaintext can't be neatly divided into the determined block size, padding will be added at the end of the plaintext. Decryption then works the same way but on reverse. 

![CBC Mode](/smt-2025/images/cbc.png)

But what happens to the first block's plaintext? If the encryption and decryption method depends on the previous block, then what will be used to encrypt the first block's plaintext? This is where IV (initialization vector) comes in. 

Therefore, in order to decrypt this flag, we need to reverse the `encrypt` method into `decrypt` by supplying the initialization vector. We also need to unpad the encoded flag to get rid of unneeded data at the end of it. But where's the IV? If we look at this line of code:

```
return (iv + ciphertext).hex()
```

The flag is in hexadecimal encoding where the iv is appended before the ciphertext. Okay, we know where it is, but how big is the size of the IV before it transitions into the actual flag? From this code:

```
iv = os.urandom(AES.block_size)
```

And by looking at the documentation of the `os.urandom` functionality:

```
Syntax: os.urandom(size)

Parameter: 
size: It is the size of string random bytes

Return Value:  This method returns a string which represents random bytes suitable for cryptographic use.
```

We can conclude that the IV's size is as big as AES's default block size, which is 16 bytes according to the documentation:

```
AES (Advanced Encryption Standard) is a symmetric block cipher standardized by NIST . It has a fixed data block size of 16 bytes. Its keys can be 128, 192, or 256 bits long.
```

With this, we're ready to construct our decoding script which is:

```
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

key = b'smt_progr4m_2k25'

with open("flag.enc", "r") as f:
    data = bytes.fromhex(f.read())

iv = data[:16]               # First 16 bytes is the IV
ciphertext = data[16:]      # The rest is ciphertext

cipher = AES.new(key, AES.MODE_CBC, iv)
plaintext = unpad(cipher.decrypt(ciphertext), AES.block_size)

print(plaintext.decode())

# output: SMT2025{3ncrypt1on_4nd_d3crypt1on}
```

## rsaaaaaaaaaa

Description
```
In RSA, the security lies on the fact that N is hard to factorize. What if it isn't?
```

We're given this output:
```
N = 41874700861388258853489749809278828485531955632713300130620929705146462431987
e = 65537
c = 6459193742574944020759123901642766353850952579897308947159707555151928327695
```

And this script that generates it:

```
from Crypto.Util.number import getPrime, bytes_to_long, long_to_bytes	# pip install pycryptodome

FLAG = b"SMT2025{REDACTED}"

# RSA Parameters
p = getPrime(128)
q = getPrime(128) 
N = p*q
e = 65537
phi = (p-1)*(q-1)
d = pow(e,-1,phi)

# Encrypting the flag
c = pow(bytes_to_long(FLAG),e,N)

# Print only the public parameters
print(f"N = {N}")
print(f"e = {e}")
print(f"c = {c}")
```

At first, I thought the problem was e's value being too small but turns out the problem lies on the comically small size of p and q. 128 bits of p and q is apparently too small, and someone probably has tried calling the `getPrime() ` method a lot of times and putting the outputs inside a database. Wait, that's factordb! I used a CLI tool called RsaCtfTool (available on Kali Linux) to automatically find the factors of N and then decrypt the message:

```
(.crypto) 📦[fred@kalilinux rsaaaaaaaaa]$ RsaCtfTool -n 41874700861388258853489749809278828485531955632713300130620929705146462431987 -e 65537 --decrypt 6459193742574944020759123901642766353850952579897308947159707555151928327695
private argument is not set, the private key will not be displayed, even if recovered.
['/tmp/tmp9amk4o0y']

[*] Testing key /tmp/tmp9amk4o0y.
attack initialized...
attack initialized...
[*] Performing factordb attack on /tmp/tmp9amk4o0y.
[*] Attack success with factordb method !

Results for /tmp/tmp9amk4o0y:

Decrypted data :
HEX : 0x0000000000000000534d54323032357b736d346c6c5f7072316d335f6234647d
INT (big endian) : 2042560714541759777531017945383753117713705496608779822205
INT (little endian) : 56716152322296766827953118673252260675642938335221680111187167816688116695040
utf-8 : SMT2025{sm4ll_pr1m3_b4d}
utf-16 : 䵓㉔㈰笵浳水彬牰洱弳㑢絤
STR : b'\x00\x00\x00\x00\x00\x00\x00\x00SMT2025{sm4ll_pr1m3_b4d}'
(.crypto) 📦[fred@kalilinux rsaaaaaaaaa]$ 
```

TL;DR don't use a comically small prime, someone's probably uploaded that to factordb.

## birthday

Description
```
There are 366 possible birthdays in a year, yet only 23 people are needed for the chance of a shared birthday to exceed 50%.

There are 2^32 possibly generated hash from this hash algorithm, yet only [???] generated hash are needed for the chance of a collision to exceed 50%.

Can you find the collision?

Connect using `nc 52.77.77.117 10041`
```

This is a challenge to find a hash collision. How do I know this? From the description and also, the source code of the server:

```
#!/usr/bin/env python3

import hashlib
from secret import FLAG

# Hash algorithm
# 4-byte / 32-bit hash --> 2^32 possible hash values
def custom_hash(string):
    mini_md5 = hashlib.md5(string.encode()).digest()[:4].hex()
    return mini_md5

# Main code

print("\033[1mHappy birthday!\033[0m 🥳🎉🎉")
print("If you give me two different strings that have the same hash value, I will give you the \033[1mFLAG\033[0m.")
print("There's one little catch, however: " + 'both strings have to start with \033[1m"SMT"\033[0m :)')

str1 = input("String 1: ")
str2 = input("String 2: ")

try:
    if str1 == str2:
        print("❌ You have to input two different strings!")

    else:
        if str1[:3] == "SMT" and str2[:3] == "SMT":
            hash1 = custom_hash(str1)
            hash2 = custom_hash(str2)
            print(f"(Hash) String 1: {hash1}")
            print(f"(Hash) String 2: {hash2}")
            if hash1 == hash2:
                print(f"✅ Same hash... Congratulations! You deserve a flag: \033[93m{FLAG.decode()}\033[0m")
            else:
                print(f"❌ Different hash... Try again!")
        else:
            print('❌ Remember, both strings have to start with "SMT"!')

except Exception as e:
    print(f"⚠️ Error: {e}")
```

We're only given the flag when the server computes the same hash from 2 different inputs with the prefix SMT. But how can we find a hash collision? Aren't commonly used hash algorithms supposed to be resistant against collision attacks?

Take a look at the custom hash function. It's using md5 (considered obselete now because a collision has happened before) AND it's trimming that to only a 4 byte/32 bits hash function. This might seem like a challenging task, to bruteforce for 2 inputs (with the SMT prefix) that has the same hash. But modern computers can do millions of calulcations in just seconds. This is my bruteforcing script:

```
def custom_hash(s):
    return hashlib.md5(s.encode()).digest()[:4].hex()

def generate_smt_string(length=8):
    suffix = ''.join(random.choices(string.ascii_letters + string.digits, k=length))
    return "SMT" + suffix

def find_collision():
    seen = {}
    attempts = 0
    while True:
        s = generate_smt_string()
        h = custom_hash(s)
        attempts += 1

        if h in seen and seen[h] != s:
            print(f"Collision found after {attempts} attempts.")
            print(f"String 1: {seen[h]}")
            print(f"String 2: {s}")
            return seen[h], s
        seen[h] = s
```

First, we defined the custom hash function. Second, we try to generate a bunch of SMTxxyyzzz strings using a combination of ascii letters, digits, and with length 8 (randomly picked). There's a higher chance of collision happening if we use a mix of letters and digits. The final and most important step is to iterate through the generation process, putting that inside a dictionary, and only stopping when we found a match.

A match here is described here:
```
if h in seen and seen[h] != s
```
So, if the computed hash *h* at increment *i* has already been inputted inside the dictionary *seen* but the corresponding input string *seen[h]* does NOT equal to the currently generated string *s* at increment *i* (i hope this makes sense...), consider that a collision and return both strings.

Running the script against the server will return this:

```
fred@fedora:~/crypt$ python3 bday.py
Collision found after 128438 attempts.
String 1: SMT65mde431
String 2: SMTEE40wvDL
📩 Server says:
 Happy birthday! 🥳🎉🎉
If you give me two different strings that have the same hash value, I will give you the FLAG.
There's one little catch, however: both strings have to start with "SMT" :)
String 1: 
🎉 Final response:
 String 2: (Hash) String 1: 16c13228
(Hash) String 2: 16c13228
✅ Same hash... Congratulations! You deserve a flag: SMT2025{saengil_chukha_hamnida}
```

This script runs almost on a instant for me.
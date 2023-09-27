+++
title = 'Flare-On 2021 - Known'
date = 2023-09-26T20:12:52-04:00
tags = ["flare", "cracking", "easy", "ransomware"]
description = "In this challenge we're dealing with a ransomware, we have to decrypt the encrypted files by finding the algorithm used for encryption."
draft = false
+++
I really loved this challenge, mainly because it was ransomware and you had to decode the encrypted files to recover the flag.

We are given the following files.
```
.
├── Files
│   ├── capa.png.encrypted
│   ├── cicero.txt.encrypted
│   ├── commandovm.gif.encrypted
│   ├── critical_data.txt.encrypted
│   ├── flarevm.jpg.encrypted
│   └── latin_alphabet.txt.encrypted
└── UnlockYourFiles.exe

2 directories, 7 files
```

Looking at UnlockYourFiles we can see that it's a decryptor for a ransomware, and looking at the `Files` directory, we can see the encrypted files.

Running UnlockYourFiles.exe asks us a key to decrypt the files, however we do not want to pay the ransom so we'll see if we can find a way to decrypt our files.

We can see a base64 encoded string when running UnlockYourFiles, decoding this base64 string reveals the following clue.
```
(>0_0)> It's dangerous to add+ror alone! Take this <(0_0<)
```

Analyzing the binary we can see the following logic.
```nasm
004011fe  80f908            cmp   cl, 0x8
00401201  7d13               jge     0x401216

00401203  8a1c31          mov    bl, byte [ecx+esi]
00401206  8a0439          mov    al, byte [ecx+edi]
00401209  32c3               xor      al, bl
0040120b  d2c0               rol       al, cl
0040120d  2ac1               sub     al, cl
0040120f  880439           mov    byte [ecx+edi], al
00401212  fec1                inc      cl
00401214  ebe8               jmp     0x4011fe
```

We see the following algorithm `rol((al ^ bl), cl) - cl`, so we implement the following python script.

```python
import string

alphabet = b""

def rotl(value, shift, bit_size=8):
    if shift == 0:
        return value
    shift %= bit_size
    return ((value << shift) | (value >> (bit_size - shift))) & ((1 << bit_size) - 1)

with open("./latin_alphabet.txt.encrypted", "rb") as file:
    for line in file.readlines():
        alphabet += line

reg_alphabet = string.ascii_uppercase
final = b""
for x in range(0, 8):
    for i in range(1, 256):
        c = alphabet[x]
        key = i
        if rotl((c ^ key), x) - x == ord(reg_alphabet[x]):
            print(f"FOUND BYTE ! : {hex(i)}")
            final += i.to_bytes()
            break

print(f"FINAL KEY: {final}")
```

This python script will use the `latin_alphabet.txt.encrypted` file because we can guess the file content.

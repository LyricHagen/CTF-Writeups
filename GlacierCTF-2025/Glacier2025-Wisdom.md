# Wisdom (November 22, 2025)

**Category:** rev  
**Points:** 50  
**Files Provided:** challenge (ELF executable), sha256sum

## Challenge Description

"You must be very wise to pass this one"

## Step 1: Reconnaissance

The first thing I did was check what files we were working with. The `sha256sum` file just contained the hash of the challenge binary for verification. The main file was `challenge`, a Linux ELF 64-bit executable.

Since I'm on macOS, I couldn't run it directly (got an "exec format error"). I ran `file challenge` to confirm it was a Linux binary, then decided to analyze it statically instead.

I ran `strings challenge` to see what text was inside:

```bash
strings challenge > output.txt
```

Some stuff I found:
- `"Enter thy wisdom: "` - asks for input
- `"You are very wise!"` - success message  
- `"Unwise..."` - failure message
- `FLAG`, `MAGIC`, `KEY` - symbols/variables
- `check_flag` - a validation function

This told me the program checks if your input matches some expected value.

## Step 2: Rev w/ Ghidra

I opened the binary in Ghidra. After letting it auto-analyze, I looked at the `main` function in the decompiler window:

```c
undefined8 main(void)
{
  bool bVar1;
  size_t sVar2;
  char acStack_48 [64];
  
  printf("Enter thy wisdom: ");
  fgets(acStack_48,0x40,stdin);
  sVar2 = strcspn(acStack_48,"\n");
  acStack_48[sVar2] = '\0';
  sVar2 = strlen(acStack_48);
  if ((sVar2 == 0x2e) && (bVar1 = check_flag(acStack_48), bVar1 != 0)) {
    puts("You are very wise!");
    return 0;
  }
  puts("Unwise...");
  return 0;
}
```

This basically says that input must be exactly `0x2e` (46 decimal) characters long, input gets 
passed to `check_flag()` for validation, and if it's valid, it prints the success message.

## Step 3: Understanding the Encryption Algorithm

Next, I looked at the `check_flag` function:

```c
bool check_flag(void *param_1)
{
  int iVar1;
  long lVar2;
  
  lVar2 = 0;
  do {
    *(byte *)((long)param_1 + lVar2) =
         ((*(byte *)((long)param_1 + lVar2) ^ (&KEY)[lVar2]) - (char)lVar2) + (char)MAGIC;
    lVar2 = lVar2 + 1;
  } while (lVar2 != 0x2e);
  iVar1 = memcmp(param_1,FLAG,0x2e);
  return iVar1 == 0;
}
```

This function encrypts your input using this algorithm:
```
encrypted[i] = ((input[i] ^ KEY[i]) - i) + MAGIC
```

Then it compares the encrypted result to the `FLAG` array using `memcmp`. If they match, you get the flag.

## Step 4: Extracting the Data

In Ghidra's Symbol Tree, I found and clicked on `FLAG`, `KEY`, and `MAGIC` to view their values in the Listing window.

**FLAG** (46 bytes starting at 0x00401380):
```
af 0f 09 18 4c 47 33 44 64 0e bc 75 bd a5 d6 ee
a0 c9 22 3a b9 cf 3c d6 eb e7 fd 45 be f8 20 b0
2b 6e a7 fe 02 49 73 84 a2 78 f0 88 c2 52
```

**KEY** (46 bytes starting at 0x00401340):
```
36 d1 d9 db 89 a5 be de 5e e6 0f 12 02 1a e1 c0
0b 4c a3 b0 08 e9 a0 d0 d1 ea 88 71 23 87 d0 41
d8 04 09 a2 fd 20 02 28 0d 75 8d 66 a8 5c
```

**MAGIC** (at 0x00403034):
```
0x0D15EA5E (only the lowest byte is used: 0x5E)
```

## Step 5: Reversing the Algorithm

To find the original flag, I needed to reverse the encryption. Starting with:
```
encrypted[i] = ((input[i] ^ KEY[i]) - i) + MAGIC
```

I worked backwards:
1. Subtract MAGIC: `encrypted[i] - MAGIC = (input[i] ^ KEY[i]) - i`
2. Add i: `encrypted[i] - MAGIC + i = input[i] ^ KEY[i]`
3. XOR with KEY: `input[i] = (encrypted[i] - MAGIC + i) ^ KEY[i]`

## Step 6: Writing the Solver

I wrote a Python script to decode the flag:

```python
#!/usr/bin/env python3

FLAG = bytes([
    0xaf, 0x0f, 0x09, 0x18, 0x4c, 0x47, 0x33, 0x44, 0x64, 0x0e, 0xbc, 0x75,
    0xbd, 0xa5, 0xd6, 0xee, 0xa0, 0xc9, 0x22, 0x3a, 0xb9, 0xcf, 0x3c, 0xd6,
    0xeb, 0xe7, 0xfd, 0x45, 0xbe, 0xf8, 0x20, 0xb0, 0x2b, 0x6e, 0xa7, 0xfe,
    0x02, 0x49, 0x73, 0x84, 0xa2, 0x78, 0xf0, 0x88, 0xc2, 0x52
])

KEY = bytes([
    0x36, 0xd1, 0xd9, 0xdb, 0x89, 0xa5, 0xbe, 0xde, 0x5e, 0xe6, 0x0f, 0x12,
    0x02, 0x1a, 0xe1, 0xc0, 0x0b, 0x4c, 0xa3, 0xb0, 0x08, 0xe9, 0xa0, 0xd0,
    0xd1, 0xea, 0x88, 0x71, 0x23, 0x87, 0xd0, 0x41, 0xd8, 0x04, 0x09, 0xa2,
    0xfd, 0x20, 0x02, 0x28, 0x0d, 0x75, 0x8d, 0x66, 0xa8, 0x5c
])

MAGIC = 0x5E

result = []
for i in range(46):
    temp = (FLAG[i] - MAGIC + i) & 0xFF
    input_char = temp ^ KEY[i]
    result.append(input_char)

flag = ''.join(chr(c) for c in result)
print("Flag:", flag)
```

Running this gave me the flag: `gctf{Ke3P_g0iNg_Y0u_goT_tH1s_00055ba509ea6138}`.

## Takeaways

- Strings in binaries can reveal important clues about program behavior.
- XOR encryption is reversible; you can decrypt by XORing again with the same key.
- Static analysis with tools like Ghidra lets you solve challenges without running the binary.
- Understanding the math behind an encryption algorithm lets you reverse it algebraically rather than brute-forcing.

# Intro (rev) (November 22, 2025)

**Category:** rev  
**Files Provided:** challenge (ELF executable), sha256sum

## Challenge Description

"Introductory rev challenge"

## Solution

I opened the binary in Ghidra and initially tried searching for flag strings ("gctf{"), but there were hundreds of matches.

Instead, I looked at the decompiled main function and found the key part:

```
fgets(local_758,0x100,stdin);
local_10 = strchr(local_758,10);
if (local_10 != (char *)0x0) {
  *local_10 = '\0';
}
iVar1 = strcmp(local_758,local_508);
if (iVar1 == 0) {
  puts("Correct");
}
```

The program stores 200 fake flags in local variables, then asks for input and compares it with `strcmp` against exactly one flag: `local_508`.

**Flag:** `gctf{bd88c4d4_w3lc0m3_t0_r3v_574dc8aa}`.

## Takeaways

- Focus on logic, ignore obvious fake flags.
- Simple challenges can be solved by just reading the decompiled code.

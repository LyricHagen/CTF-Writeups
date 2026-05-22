# Remoose (May 15, 2026)

**Category:** rev  
**Points:** 192  
**Files Provided:** chall

## Challenge Description

"I changed just one little thing and my racing moose won't run anymore!"

## Step 1: Exploring

`file chall` returned `data` - the magic header was broken. `xxd chall | head` showed two abnormalities:

```
00000000: 7f45 4c4b 0201 0120 2020 2020 2020 2020  .ELK...
```

- Byte 3 is `0x4B` ('K') instead of `0x46` ('F'): the magic is `.ELK` not `.ELF`. ELF -> ELK (like... an elk). That's why the challenge is called "remoose", I'm guessing.
- Every NUL byte is replaced with `0x20` (space). The file contains 0 NULs and 14106 spaces.

## Step 2: Repairing

Replace `0x20` -> `0x00` globally and set byte 3 back to `0x46`:

```
data = bytearray(open('chall','rb').read())
fixed = bytearray(0x00 if b == 0x20 else b for b in data)
fixed[3] = 0x46
open('chall_v2','wb').write(fixed)
```

`file chall_v2` now reports a proper ELF64 PIE executable. I'm on macOS so I can't run it; the system LLVM objdump failed on the broken string table (some original spaces inside `.strtab` got nulled too), but Homebrew's `x86_64-elf-objdump` worked.

## Step 3: Reading the Chain

`main` does nothing but call `flag()`. Each `flagN` function emits a substring via hardcoded `putchar`/`printf` immediates, then tail-calls the next:

```
flag :  putchar('t','j','c','t') + printf("f{")   = "tjctf{"
flag1:  putchar('5','m')                          = "5m"
flag2:  putchar('a','1','1','_')                  = "a11_"
flag3:  putchar('m','0')                          = "m0"
flag4:  putchar('0','s','3') + printf("%c", '}')  = "0s3}"
```

One call in `flag4` looked off (target 0x1010 inside `_init` instead of the putchar PLT at 0x1030) - that's because the rel32 offset stored in the original file was a legitimate `0x20`, which my global swap zeroed out. Doesn't matter for reading the flag.

Concatenating:

`tjctf{5ma11_m00s3}`

## Takeaways
- `file` returning "data" on something that should be an ELF = the magic header is corrupted. Check the first 16 bytes by hand.
- Bulk byte substitutions (NUL <-> SPACE) are reversible but lossy in the wrong direction - some original 0x20s inside instruction immediates / strtab will get wrongly nulled. Fine for static analysis, breaks runtime.
- System LLVM objdump is strict about ELF correctness; the GNU x86_64-elf cross-toolchain version disassembles more aggressively when sections are mildly broken.


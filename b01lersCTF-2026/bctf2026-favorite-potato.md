# Favorite Potato (April 18, 2026)

**Category:** rev  
**Points:** 215  
**Files Provided:** code.bin.gz, test.bin, favorite_potato.py, screenshot.png  
**Connection:** `ncat --ssl favorite-potato.opus4-7.b01le.rs 8443`

## Challenge Description

"Yay, at last - I managed to upgrade my old potato. Now I can run suuuuper loooong binaries that nobody can reverse (RAM is still 64k but I only need a minimal amount)."

## Step 1: Exploring

The "potato" is a Commodore 64 / 6502. favorite_potato.py runs the binary on the server using a `run_c64()` helper that POKEs a random (A, X, Y) into the 6502 registers, `SYS`-es into the code, and PEEKs the registers back. The menu gives you "T)est" (one shot against test.bin) or "R)eal" (20 outputs from code.bin, and you have to send back all 20 inputs in order to get the flag.) Essentially, the code is a pure function `f: (A, X, Y) -> (A', X', Y')` on 24 bits. Since it's deterministic on a finite space, it has to be a bijection.

test.bin was only 9 bytes, so I could hand-disassemble it: PHP, CLC, ADC #$2A, DEX, INY, INY, PLP, RTS. That's `(A+42, X-1, Y+2)`, so given an output, the input is `(A'-42, X'+1, Y'-2)`.

Meanwhile, `gunzip -c code.bin.gz | wc -c` returned 5,820,001 bytes (lots of 6502.) The flavor text ("RAM is still 64k but I only need a minimal amount") hinted that the real computation was small, and the larger file was just bloat.

## Step 2: Reverse-engineering the Binary

Counting byte frequencies showed there were no memory-access opcodes - only register/stack operations like PHA/PLA, PHP/PLP, TYA/TAY, TXA/TAX, TXS/TSX, LSR, EOR, ADC, SBC, BNE, BCC, DEX, INX. The code only ever touched A, X, Y, SP, flags, and the hardware stack page.

My first thought was to just brute force it - write an emulator, run all 2^24 inputs, build a lookup table. I wrote a C 6502 emulator, but a single forward call took 30ms (about 32 million instructions), so a full sweep would have taken roughly 5.8 days single-threaded. (too slow, even with OpenMP.)

Looking at the binary more carefully, I saw the same patterns repeating thousands of times, e.g.:

```
08 48 e8 38 e9 01 d0 fa 68 28
PHP PHA INX SEC SBC#1 BNE-6 PLA PLP
```

The BNE target is at `INX`, not `SEC`, so `INX` runs every iteration. The loop runs A_orig times before the zero flag fires, and the net effect is `X += A` with flags restored at the end. This 10-byte block appeared 10,000 times in the binary.

Working through more patterns, every block of bytes in the file matched one of exactly 10 macro primitives: `A += imm`, `A ^= imm`, `X += A`, `Y += A`, `A = ror8(A, n)`, swap A/X, swap A/Y, swap X/Y, `A ^= Y`, and RTS. Sizes ranged from 4 bytes (the XOR_A wrapper) to 90 bytes (the bit-serial XOR_A_Y block.)

The PHP/PLP wrappers around everything just meant "save/restore flags so the bloat doesn't disturb the outside world." The TSX/INX/TXS tricks inside the swap and XOR macros shifted the stack pointer without reading, letting the code pop values back in a different order.

I wrote a Python parser that scanned the binary and greedily matched one macro at a time (each macro represented as a tuple of bytes with `None` for operand slots.) It covered the entire file, parsing out 250,001 macros and consuming all 5,820,001 bytes with none left over.

So the whole 5.82MB binary was just 250,001 simple operations drawn from a 10-item vocabulary. I verified by running the parsed macros in Python and comparing to the C emulator on various inputs (bit-exact match on every one.)

## Step 3: The Solution

I initially planned to run a fast sweep across all 2^24 inputs to build the inverse lookup table. But then I realized that every single macro is individually invertible.

- `A += imm` reverses with `A -= imm`
- `A ^= imm`, `A ^= Y`, and all three swaps are their own inverses
- `X += A` reverses with `X -= A` (A doesn't change during X_ADD_A, so the "current A" is the same as when the forward op was applied)
- `A = ror8(A, n)` reverses with `A = rol8(A, n)`, i.e., ror by (8-n)

So `f^(-1)` is just: walk the macro list in reverse, apply each macro's inverse. No lookup table and no sweep, just one pass of 250K simple ops per query, ~25ms in Python.

```
def inverse(A, X, Y):
    for _, name, operands, _ in reversed(OPS):
        if name == "ADD_A":    A = (A - operands[0]) & 0xff
        elif name == "XOR_A":  A ^= operands[0]
        elif name == "X_ADD_A":X = (X - A) & 0xff
        elif name == "Y_ADD_A":Y = (Y - A) & 0xff
        elif name == "ROR_A":
            n = operands[0] & 7
            if n: A = ((A << n) | (A >> (8-n))) & 0xff
        elif name == "SWAP_AX":A, X = X, A
        elif name == "SWAP_AY":A, Y = Y, A
        elif name == "SWAP_XY":X, Y = Y, X
        elif name == "XOR_A_Y":A ^= Y
    return A, X, Y
```

Checked w/ `inverse(forward(x)) == x` for some random inputs before touching the server.

The solver script connected with `ssl` + `socket`, sent "R", regex-parsed the 20 "Final output #i: A=.. X=.. Y=.." lines, called `inverse(A, X, Y)` on each, and sent back the 20 answers. All 20 quickly inverted and the server printed:

`bctf{Nev3r_underst00d_why_we_n33d_TSX_and_TXS_unt1l_n0w..:D}`

## Takeaways
- A big reverse-engineering binary usually hides a small computation behind wrapping and padding; look for repeated patterns rather than trying to emulate from scratch
- When a function is a composition of invertible primitives, you can compute `f^(-1)` by reversing the list and inverting each step (no bruteforce sweep needed)
- PHP/PLP wrappers hint that whatever's inside doesn't leak flag state to the outside, so you can analyze each block in isolation

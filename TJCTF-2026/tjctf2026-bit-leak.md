# Bit Leak (May 15, 2026)

**Category:** crypto  
**Points:** 280  
**Files Provided:** server.py  
**Connection:** `nc tjc.tf 31001`

## Challenge Description

"our security monitor only ever tips us off about the parity of RSA decryptions. turns out 'even or odd' isn't much of a secret. can you recover the message one bit at a time?"

## Step 1: Exploring

The server generates 512-bit RSA (two random 256-bit primes, e=65537), prints n/e/ciphertext on connect, and lets you submit up to 2100 ciphertexts. For each, it returns only `pow(candidate, d, n) & 1` - the LSB of the decryption. The flag is the original plaintext.

This is the textbook RSA LSB parity oracle.

## Step 2: The Attack

RSA is multiplicatively homomorphic, so `c * 2^e mod n` decrypts to `2m mod n`. Since n is odd:
- if `2m < n`, the product is even (LSB = 0)
- if `2m >= n`, you get `2m - n`, which is odd (LSB = 1)

So each query is one bit of binary search on m. Iterate with `c * (2^i)^e mod n` and recover m in ~log2(n) = 512 queries.

To avoid precision drift over 512 iterations, I tracked bounds as Python Fractions:

```
lo, hi = Fraction(0), Fraction(n)
mult = pow(2, e, n)
cur = c
for _ in range(n.bit_length()):
    cur = (cur * mult) % n
    if oracle(cur) == 0: hi = (lo + hi) / 2
    else:                lo = (lo + hi) / 2
```

## Step 3: Off-by-a-Few

After 512 queries (~1 minute), `int(hi)` decoded to `tjctf{parity_isnt_privacyz`. Final-bit rounding leaves the recovered integer a few off; my candidate filter short-circuited on "contains tjctf" before checking deltas. Trying `m + d` for `d` in `[-10, 10]` locally, `d = +3` gave:

`tjctf{parity_isnt_privacy}`

## Takeaways
- Just one bit of leakage is plenty if RSA is unpadded - LSB of `2m mod n` is one bit of binary search.
- Use Fractions (or tracked integer bounds), not floats - precision dies long before bit 512.
- Always sweep a small delta window around the recovered integer; don't let the candidate filter stop at the first near-match.


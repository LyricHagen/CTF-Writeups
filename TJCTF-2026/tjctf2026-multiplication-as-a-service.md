# Multiplication as a Service (May 15, 2026)

**Category:** crypto  
**Points:** 234  
**Files Provided:** server.py  
**Connection:** `nc tjc.tf 31313`

## Challenge Description

"Mendacem oportet esse memorem." (A liar should have a good memory.)

## Step 1: Exploring

The server stores the flag bytes as a big integer `d`, advertises the curve `y^2 = x^3 + 2x + 3` over `F_10007`, and on each query reads `(x, y)`, runs `scalar_mul(d, (x % P, y % P))`, and prints `Q`.

Reading `point_add`, the formulas reference A but never B. And nothing validates that `(x, y)` actually lies on the advertised curve. So if I submit a point on any twist `y^2 = x^3 + 2x + B'`, every intermediate point in `scalar_mul` stays on that curve too - and the server computes `d * P` on a curve of my choice. Classic invalid curve attack.

## Step 2: The Attack

Plan: pick ~50 different B', each giving a curve order with small prime factors, find a generator on each, send it, receive `Q = d * P`. Solve the DLP on the small curve via Pohlig-Hellman + BSGS to get `d mod n_B'`. CRT the residues until the combined modulus exceeds the flag's bit length.

`p = 10007` is tiny, so curve orders fall within Hasse's bound `p + 1 ± 2*sqrt(p)`, and counting them by enumerating x and testing if `x^3 + 2x + B'` is a QR is O(p). I scanned B' = 0..199 and greedily kept curves contributing new prime-power factors to the running lcm:

```
L = 1
for B' in range:
    n = curve_order(B')
    if lcm(L, n) > L:
        keep B'; L = lcm(L, n)
    if L.bit_length() > 500: break
```

53 curves got me a 506-bit lcm. For each, I found a full-order generator by walking x = 0, 1, 2... and checking `(n_B' / l) * P != O` for every prime `l | n_B'`.

## Step 3: Running It

One TCP connection, 53 (x, y) submissions, parse the Q lines. For each curve, Pohlig-Hellman over the prime-power factors of `n_B'` (each subgroup of size at most ~5000, BSGS needs ~70 steps). All 53 DLPs succeeded; sanity-checked each with `k * P == Q` before accepting.

CRT over the 53 residues (with a coprime-tolerant CRT for shared small factors like 2, 3 - they agree on overlaps) gave d, and `d.to_bytes(...)`:

`tjctf{per_mendacium_ad_veritatem}`

## Takeaways
- If short Weierstrass addition formulas don't reference B, and the server skips point validation, you can swap in any twist curve and the server will silently compute on it.
- p tiny enough for naive point counting means you can pick curves with very smooth orders - Pohlig-Hellman becomes trivial.
- The Latin epigraph fit perfectly: lying to the server (off-curve points) led to the truth (d).


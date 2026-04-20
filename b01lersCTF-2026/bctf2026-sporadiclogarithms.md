# Sporadic Logarithms (April 18, 2026)

**Category:** crypto
**Points:** 224
**Files Provided:** chall.py
**Connection:** `ncat --ssl sporadiclogarithms.opus4-7.b01le.rs 8443`

## Challenge Description

"I heard the discrete logarithm problem can be solved efficiently in all sporadic groups without even knowing the group. Can you prove it to me?"

## Step 1: Exploring

The provided chall.py sets up a blackbox group `GL(3, 65537)` and asks you to recover a secret exponent `x` over 5 rounds. Each round, the server picks a small-order conjugator `c` (order `m` between 2 and 8), a random `g`, and computes `h = s_{g,phi}(x)` where `phi(a) = c*a*c^-1`.

You only get handles, not actual matrices. The blackbox exposes `mul`, `inv`, `phi`, `eq`, and `submit`. The bound on `x` is `262144 = 2^18` and you get 10000 queries per round.

Looking at `hol_pow`, the function `s_{g,phi}(x)` is the first component of `(g, c)^x` in the holomorph. Working it out by induction, this equals `g * phi(g) * phi^2(g) * ... * phi^(x-1)(g)`.

## Step 2: The Approach

Since `c^m = I`, we have `phi^m = identity`. So if you let `P = g * phi(g) * ... * phi^(m-1)(g)` and `T_r = g * phi(g) * ... * phi^(r-1)(g)` for `r < m`, you can group the product into chunks of size `m`:

`s_{g,phi}(qm + r) = P^q * T_r`

This reduces the problem to discrete log in the cyclic subgroup `<P>` with `m` possible cosets. Baby-step-giant-step works perfectly here. The blackbox's `_add_elem` dedupes by matrix value, so equal matrices return identical handles - you can use the handles themselves as collision keys in a Python dict.

I wrote the solver: compute all `phi^i(g)`, build `T_r` and `P`, then BSGS with baby step `P^j * T_r` and giant step `P^(-iB) * h`.

## Step 3: The Bug

My first attempt passed 3 out of 5 rounds, then failed on a round with `c_order=8`. Re-running, it failed on a different round with `c_order=4`. The pattern: it kept failing when `x_secret % m != 0`.

The issue was that I'd written the giant step as `cur = mul(cur, PnegB)` (right-multiplication) when the BSGS math requires `cur = mul(PnegB, cur)` (left-multiplication). In a commutative group these are the same, but `GL(3, 65537)` is non-commutative. The bug only manifested when the residue `r_secret` was nonzero, because `h = P^q * T_0 = P^q` is just a power of P, and powers of P commute with each other.

After flipping the multiplication order:

```python
if i < max_i:
    cur = mul_q(PnegB, cur)
```

I ran the script again and all 5 rounds passed, giving me the flag:

`bctf{th3_m0n5t3r_gr0up_w45_t0_1mpr4ct1c4l_:(}`

## Takeaways

- When a "discrete log in a generic group" challenge gives you a small-order automorphism, the holomorph collapses into BSGS over `<P>` plus a small number of cosets.
- Blackbox handle systems often dedupe by element value - that means you can do collision-based attacks just by collecting handles, no `eq` needed.
- In non-commutative groups, left vs right multiplication matters. Test your algorithm on a case where the answer ISN'T trivially symmetric, or you'll miss the bug.

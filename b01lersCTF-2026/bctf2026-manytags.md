# Many Tags (April 18, 2026)

**Category:** crypto  
**Points:** 214  
**Files Provided:** many_tags.py  
**Connection:** `ncat --ssl manytags.opus4-7.b01le.rs 8443`  

## Challenge Description

"I'm leaking too many tags..."

## Step 1: Exploring

Connecting to the server gives me the flag's GCM nonce, ciphertext, and tag, plus a menu where option 1 returns a random (nonce, ciphertext, tag) triple. I have a budget of 704 queries.

Looking at many_tags.py, the server uses AES-GCM but doesn't return the real tag. For each query, it computes the real tag, then overwrites the low 64 bits:

```
tag_low = (ghash_value ^ fault_value) & MASK64
```

`ghash_value = C*H^2 + L*H` in GF(2^128), where H is the GCM auth subkey and L is the length block. `fault_value = (fault_words[0] << 32) | fault_words[1]`, and the fault words come from a `random.Random` seeded with the 32-byte AES master key itself.

So each query leaks `low_64(C*H^2 + L*H) XOR fault_value`. Both H and the MT19937 state are unknown, but if I can recover the master key, I can decrypt the flag.

## Step 2: The Approach

The key observation is that both sides of the equation are GF(2)-linear. Multiplication by a constant in GF(2^128) is linear over GF(2), the Frobenius map `x -> x^2` in characteristic 2 is also linear, and every MT19937 output bit is a fixed XOR of the 19968 initial state bits.

So it's a linear system in 128 + 19968 = 20096 unknowns (H bits + MT state bits), and each query gives 64 equations. 320 queries gives me 20480, enough to solve. After recovering the state, I still need to invert CPython's `init_by_array` to get back the master key.

I wrote a solver that symbolically simulates MT19937 (each output bit is a Python bigint showing which state bits XOR to give it), builds the matrix, and solves via bit-packed Gaussian elimination using numpy uint64 arrays. The init_by_array inverter first reverses the second loop (the one with multiplier 1566083941), then reverses the first loop (with 1664525) using iterations k=2..9 to recover all 8 key words directly:

```
key[k%8] = S_mid[k+1] - (state_0[k+1] ^ ((S_mid[k] ^ S_mid[k]>>30) * 1664525)) - (k%8)
```

The main script connects via SSL, gathers 320 queries, builds the ~20000x20000 matrix, runs Gaussian elimination (~70 seconds), extracts H and the MT state, inverts to get the master key, then `AESGCM.decrypt`s the flag:

`bctf{too_fragil}`

## Takeaways
- When outputs are masked by PRNG data and the PRNG is seeded with a value you want, you can often set up a joint linear system in the unknown and the PRNG state.
- GF(2) linearity is easy to miss - squaring in characteristic 2 is linear.
- CPython's MT19937 `init_by_array` is fully reversible given the post-init state.

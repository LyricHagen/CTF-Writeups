# Celestial Vault (May 15, 2026)

**Category:** crypto (bridging)  
**Points:** 499  
**Files Provided:** celestial_observatory.png, spectral_key.png, vault.bin, vault_meta.txt

## Challenge Description

"the vault runs on time. light shows what time hides."

## Step 1: Exploring

vault_meta.txt described an XOR stream cipher: `seed = SHA256(passphrase_ascii)`, `block_i = SHA256(seed || u32be(i))`, output is a ZIP. So I needed the ASCII passphrase to decrypt vault.bin.

celestial_observatory.png ("plate A") was a 4x4 grid of analog clocks labeled A..P, some with gold rings and some grey. spectral_key.png ("plate B") had the same 4x4 grid with some dots lit and connected by purple lines; dot C had a yellow ring around it (start point).

## Step 2: Reading the Plates

I sampled an annulus around each clock to classify the ring color (gold: R>>B, grey: B>R). 8 clocks were gold: B, C, F, G, H, I, M, N.

For the spectral key, I masked the purple line pixels, then for every pair of lit letters I parameterized the line between them and counted the fraction of pixels on the mask. Density 1.00 = connected. One false positive: H-N's midpoint sits exactly on K, so H-K and K-N flagged too. The true 7 edges formed an open path from C (yellow start) to G:

`C -> H -> N -> M -> I -> F -> B -> G`

For each gold clock, I masked yellow/light-blue/red hands separately, found the tip pixel (farthest from center), and converted angles to (h, m, s). In path order:

C 04:19, H 01:57, N 09:06, M 12:42, I 06:33, F 09:24, B 03:08, G 02:51

## Step 3: The Passphrase

I tried ~700 candidate formats - HHMM, HHMMSS, path letters, hours as A=1..L=12, total seconds, theme phrases, etc. - all wrong.

Insight: an analog clock can't distinguish AM from PM, so each of the 8 clocks has 2 readings - 256 possible 24-hour patterns. For each pattern I checked the first 4 keystream bytes against `vault.bin[0:4] XOR b"PK\x03\x04"` (`cdb2c430`) - a 2-hash check per candidate.

One pattern matched:

`p = "04191357210600421833092415080251"`

i.e. AM PM PM AM PM AM PM AM across the path.

## Step 4: Decrypting

Standard XOR-stream decrypt produced a valid ZIP (`PK\x03\x04`) containing `scan/final_microfilm.png` - graph paper with red cursive across the middle. Masking red pixels and rotating 180°, the handwriting read:

`tjctf{constellations_reveal_the_flag}`

## Takeaways
- A 2-hash brute force is fine when you can verify a candidate against a few known-plaintext bytes (the ZIP magic, here).
- An analog clock inherently can't encode AM/PM - if the puzzle's input space is too constrained, look for a hidden bit per element.
- "Bridging" challenges = expect to combine image analysis, graph reasoning, and crypto in one chain.


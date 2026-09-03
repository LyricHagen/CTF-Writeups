# Obscure Crusher 1 (May 15, 2026)

**Category:** forensics  
**Points:** 289  
**Files Provided:** chall.bin (169 bytes)

## Challenge Description

"What if I make it so you need 3 keys to unlock the flag..."

## Step 1: Exploring

file chall.bin says "Mac OS X icon, 256 bytes, icns type," but the file is only 169 bytes. Strings showed icns, name, ttf, xy, lzma, KLZMA_DATA. The bytes at 0x6D were 5D 00 00 80 00 - byte-for-byte an LZMA1 ALONE header. So everything screams "this is a compressed icns/ttf file with three layers."

I burned a long time on that path:
- LZMA1 ALONE / RAW at every offset, every (lc, lp, pb) tuple, every dict size
- LZMA2, XZ, AUTO at every offset
- bzip2, zlib/gzip/deflate, LZ4, zstd, brotli
- XOR with single-byte keys and every keyword visible in the file

All failed. The compression-themed framing was bait.

## Step 2: The Breakthrough

Assume the 41-byte payload at 0x80 starts with tjctf{:

```
payload[0..5] = 3A 1D 09 0D 07 67
"tjctf{"      = 74 6A 63 74 66 7B
derived key   = 4E 77 6A 79 61 1C
```

Byte 0x4E ('N') doesn't appear anywhere in the file - no contiguous slice matches. But XOR'ing the payload against the 41-byte slice starting at file offset 0x07:

:tjctf\x0fD\x04q\x1b\x0c\x1e...

Positions 1-5 spell "tjctf." That's because file[0x07] = 0x00 (so payload[0] = ':' passes through as a leading separator) and file[0x08..0x0C] = "icns\x01" decodes positions 1..5 cleanly. Five correct ASCII bytes in a row is no coincidence.

## Step 3: Chaining the Three Keys

key[0..4] = "icns\x01" decodes "tjctf"; for position 5 to be {, I need key[5] = 0x0F XOR 0x7B = 't'. The next structural fragment in the file is the "name" element's data at 0x53: "ttf\x02xy" - starts with 't'. Plugging that in gives "tjctf{0bscu".

For "obscure" the next two key bytes must be 'l', 'z' - which is exactly where "lzma" starts at 0x72. The cycle ends with 'K' from "KLZMA_DATA" at 0x76. The three concatenated structural fragments form a 16-byte repeating key:

```
key = b"icns\x01" + b"ttf\x02xy" + b"lzmaK"   # 5 + 6 + 5 = 16
```

## Step 4: Decrypting

```
data = open('chall.bin', 'rb').read()
key = data[0x08:0x0D] + data[0x53:0x59] + data[0x72:0x77]
enc = data[0x81:]
plain = bytes(b ^ key[i % 16] for i, b in enumerate(enc))
print(plain.split(b'}')[0] + b'}')
```

tjctf{0bscur3_crush3r_1cns_ttf_lzm3}

The flag literally names the three fragments. The "3 keys" hint was literal.

## Takeaways
- When standard tooling uniformly fails, the format probably isn't standard - stop trying decompressors and start doing known-plaintext analysis.
- Repeating-XOR with a tjctf{ crib gives you 6 key bytes for free. If they don't appear contiguously, try concatenating structural fragments.
- File regions labeled with familiar magic strings can be the key material, not the data.


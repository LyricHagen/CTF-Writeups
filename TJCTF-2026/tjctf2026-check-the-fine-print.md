# Check the Fine Print (May 15, 2026)

**Category:** forensics  
**Points:** 201  
**Files Provided:** logo.png

## Challenge Description

"what? the details matter? but why can't i see them..."

## Step 1: Exploring

logo.png was 65 KB for a 150x150 flat-color image - way too large. `exiftool logo.png` flagged it:

`Warning : [minor] Trailer data after PNG IEND chunk`

The IEND chunk lives at offset 14268; everything after (50741 bytes) is appended. The trailer starts with `PK\x03\x04` - a ZIP, containing 248 PNGs named 001.png through 248.png.

## Step 2: Inspecting the Tiny PNGs

Each was 19x9 RGB. Rendering them as ASCII art, they were just hand-drawn renderings of their own index number (001 = "1", 248 = "248", etc.) - the literal "fine print." Pixel content was a red herring.

But dumping the hex of the first few PNGs revealed that the IHDR's compression-method byte (file offset 26) varied between 0 (valid per spec) and 1 (invalid but ignored by all decoders):

```
001 IHDR tail:  ... 08 02 00 00 00 ...
002 IHDR tail:  ... 08 02 01 00 00 ...
```

248 PNGs × 1 bit = 248 bits = 31 bytes - the right length for a short flag.

## Step 3: Decoding

Walk all 248 files, grab `data[26]`, concatenate, parse 8-bit MSB-first:

```
bits = "".join(str(open(f'extracted/{i:03d}.png','rb').read()[26]) for i in range(1,249))
msg = "".join(chr(int(bits[i:i+8], 2)) for i in range(0, 248, 8))
```

`tjctf{wow_you_actually_read_it}`

## Takeaways
- PNG ignores anything past IEND - exiftool will tell you when there's trailer data.
- Spec-mandated header bytes that are "always 0" are great smuggling channels - decoders ignore them and they survive byte-for-byte.
- If a file is much bigger than its visible content, the payload is in the container, not the pixels.


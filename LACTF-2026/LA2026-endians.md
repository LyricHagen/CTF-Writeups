# Endians (February 7, 2026)

**Category:** misc  
**Points:** 100  
**Files Provided:** gen.py, chall.txt

## Challenge Description

"I was reading about Unicode character encodings until one day, my flag turned into Japanese! Does little-endian mean the little byte's at the end or that the characters start with the little byte?"

## Step 1: Exploring

chall.txt contained the following:

`氀愀挀琀昀笀㄀开猀甀爀㌀开栀　瀀攀开琀栀㄀猀开搀　攀猀开渀　琀开最㌀琀开氀　猀琀开㄀渀开琀爀愀渀猀氀愀琀椀　渀℀紀`

And gen.py was quite short:

```
text = "lactf{REDACTED}"
endian = text.encode(encoding="???").decode(encoding="???")
with open("chall.txt", "wb") as file:
    file.write(endian.encode())
```

I figured that the encoding was swapping bytes somehow. UTF-16 has a little-endian variant (UTF-16-LE) and a big-endian variant (UTF-16-BE), which differ in byte order. If you encode text as one and decode as the other, the bytes get flipped.

## Step 2: The Solution

It looked like the chall.txt text was created by encoding the flag with UTF-16-LE and decoding those same bytes as UTF-16-BE, so to reverse this, I'd just have to do the opposite of that (i.e. encode the text as UTF-16-BE and decode those bytes as UTF-16-LE.)

I wrote a Python script.
```
python3 << 'EOF'
with open("chall.txt", "r", encoding="utf-8") as f:
    garbled = f.read().strip()

flag = garbled.encode("utf-16-be").decode("utf-16-le")
print(flag)
EOF
```

Which gave me the flag:

`lactf{1_sur3_h0pe_th1s_d0es_n0t_g3t_l0st_1n_translati0n!}`


## Takeaways
- Encoding with one variant and decoding with another creates predictable and reversible changes

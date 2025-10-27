# DEADFACE CTF: Secret Frog (October 25, 2025)

**Category:** Steganography  
**Points:** 80  
**Files Provided:** froggy-steg.png  

---

## Challenge description

We’re given a PNG file named **froggy-steg.png** with the hint that something's hidden inside the image.

---

## Step 1: Inspecting the File

I started by using **binwalk**, a tool that scans binaries for embedded files and data:

```
binwalk -e "/Users/myname/Downloads/froggy-steg.png"
```

binwalk identified that there was another file embedded inside the PNG, specifically a GIF image located after the PNG data.

---

## Step 2: Extracting the Hidden Data

To manually extract the hidden file, I used the dd command. The offset shown by binwalk was 1223461, meaning the hidden GIF data started at that byte position.

```
dd if="/Users/myname/Downloads/froggy-steg.png" of="/Users/myname/Downloads/hidden.gif" bs=1 skip=1223461 status=progress
```

Here’s what each part does:

    if — input file (the original PNG)
  
    of — output file (where the hidden data will go)
  
    bs=1 — reads one byte at a time for precise extraction
  
    skip=1223461 — skips the first 1,223,461 bytes
  
    status=progress — shows progress as it runs

---

## Step 3: Analyzing the Extracted GIF

Once the extraction was done, I opened the new file (hidden.gif).

Scrolling through the frames, there was text appearing near the end of the animation; the flag was clearly written across the last 30-or-so frames.

---

## Takeaways:

Always check for embedded files in image challenges; binwalk is useful for this.

Use dd to extract data at a specific offset.

If you find an embedded GIF or video, scroll through all frames, since hidden text could show up near the end.

The combination of binwalk + dd is a common CTF steganography extraction technique.

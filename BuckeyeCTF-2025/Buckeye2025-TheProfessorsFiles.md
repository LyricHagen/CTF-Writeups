# DEADFACE CTF: The Professor's Files (November 8, 2025)

**Category:** Beginner/Forensics  
**Points:** 50
**Files Provided:** OSU_Ethics_Report.docx 

---

## Challenge description

"This is a good forensics challenge to start with. Use the help button to ask for a hint if you get stuck.

A professor uploaded their ethics report for review, but something about it seems… off. The document’s
formatting feels inconsistent, almost like it was edited by hand rather than exported normally. The metadata 
looks strange too, and there’s a rumor that the professor hides sensitive data in plain sight."

---

## Step 1: Converting the File & Extracting the Zip

I started with 3 commands: **cd, cp, and unzip,**, which respectively navigated to my downloads folder, copied the provided docx file
into zip format, and unzipped the file into a folder called "extracted."

```
cd /Users/myname/Downloads
cp OSU_Ethics_Report.docx OSU_Ethics_Report.zip
unzip OSU_Ethics_Report.zip -d extracted
```

---

## Step 2: Searching For the Flag

I now had a folder with all of the files stored within the provided docx file. It had 3 folders: _rels, docProps, and word, and 1 file called \[Content_Types\].xml.

I traversed each of the files and looked at their text contents. Eventually, I came across the file theme1.xml, located at /extracted/word/theme/theme1.xml.

Lines 14-20 were as follows:

      <a:accent5><a:srgbClr val="FFD700"/></a:accent5>  <!-- gold -->
      <a:accent6><a:srgbClr val="DC143C"/></a:accent6>  <!-- crimson -->
      <!-- bctf{flag_redacted} -->

      <a:hlink><a:srgbClr val="0000FF"/></a:hlink>
      <a:folHlink><a:srgbClr val="800080"/></a:folHlink>
    </a:clrScheme>

As you can see, the flag is right there among the code.

---

## Takeaways:

Docx files can be turned into zip files and extracted.

In simpler challenges, flags can simply be hidden among the code.

# Mutation Mutation (February 7, 2026)

**Category:** web  
**Points:** 100  
**Site:** mutation-mutation.chall.lac.tf

## Challenge Description

"It's a free flag! You just gotta inspect the page to get it. Just be quick though... the flag is constantly mutating. Can you catch it before it changes? 🧬"

## Step 1: Exploring the Website

the website is loading i can't write anything here yet

## Step 2: The Solution

aaa

I think this is what worked.

```
setTimeout(() => {
    const allFlags = Array.from(seenFlags);
    const textarea = document.createElement('textarea');
    textarea.style.position = 'fixed';
    textarea.style.top = '10px';
    textarea.style.left = '10px';
    textarea.style.width = '80%';
    textarea.style.height = '80%';
    textarea.style.zIndex = '999999';
    textarea.value = allFlags.join('\n\n');
    document.body.appendChild(textarea);
    textarea.focus();
    textarea.select();
    console.log('flags above');
}, 5000);
```

so then i got the flag

`lactf{forgot}`


## Takeaways
- In web challenges, after you explore the website, do static analysis
- Often in CTFs, it's easier to run console commands than to try to do something manually on the website

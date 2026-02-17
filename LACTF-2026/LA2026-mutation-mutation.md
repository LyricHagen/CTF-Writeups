# Mutation Mutation (February 7, 2026)

**Category:** web  
**Points:** 100  
**Site:** mutation-mutation.chall.lac.tf

## Challenge Description

"It's a free flag! You just gotta inspect the page to get it. Just be quick though... the flag is constantly mutating. Can you catch it before it changes? 🧬"

## Step 1: Exploring the Website

As I'm writing this writeup, the website isn't loading, so I don't recall *exactly* what its contents were. But I know that when you went to Inspect Element and looked in the Elements tab, you'd see many, many comments flashing by, most of which were fake flags.

By screenshotting the page, I could see the flag - something like "lactf{constant_mutation_is_fun_[25ish strange characters]}. It wasn't easy to tell what these characters were - it was uncommon, difficult-to-parse Unicode characters, so my attempts to parse it manually failed. I tried right-clicking on the element to copy it as it flashed by, but it'd just paste as "undefined."

## Step 2: The Solution

I felt that the solution would involve a console script that took all the comments from the Elements tab ans pasted them into an easily-copypastable list, so I experimented with different JS scripts that took in the comments and outputted them in the console. Most of these scripts resulted in unfamiliar errors, but the following script is the one that ended up working. It outputted all the flags in an easily copiable way, and from there, I could just choose the one that was clearly not one of the fake ones.

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

Resulting in the flag:

`lactf{с0nѕtаnt_mutаtі0n_1s_fun!_🧬_👋🏽_ІlІ1| ض픋ԡೇ∑ᦞ୞땾᥉༂↗ۑீ᤼യ⌃±❣Ӣ◼ௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌௌ}`


## Takeaways
- Sometimes, text isn't simply copiable, and you have to use the console to "move it over."

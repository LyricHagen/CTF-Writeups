# The Trial (February 7, 2026)

**Category:** web  
**Points:** 100  
**Site:** the-trial.chall.lac.tf

## Challenge Description

"I think the main takeaway from Kafka is that bureaucracy is bad? Or maybe it's that we live in a society."

## Step 1: Exploring

Upon opening the website, I'm greeted with a slider spinning clockwise, with an "I'm Feeling Lucky" button and a "Submit" button floating around (the former of which takes you to a rickroll.)
Text reads "I want the ____," and by dragging the slider, the 4 characters at the end of that line change to ostensibly random characters.
I figured that if you get it to say "I want the flag" and click submit, then you'd get the flag. The hard part is dragging the slider just right to get that precise string of characters.

## Step 2: The Website

I saved the webpage to look at it statically. In `The Trial.html`, you can see that the value gets encoded using a cipher "kjzhcyprdolnbgusfiawtqmxev". The encoding converts the slider value (0-456975) into a 4-character word using base-26 conversion with the aforementioned custom alphabet.
Looking at this for loop, we can see the math behind the slider's stored value:
```
for (let i = 0; i < 4; i ++) {
    s += cm[n % cm.length];
    n = Math.floor(n / cm.length);
}
```
Basically, it takes in a number n and converts it to a 4-character string using the cm like a custom alphabet. Each iteration extracts the least significant base-26 digit via modulo, appends the corresponding character, then shifts right via division to process the next digit. In the cipher string, 'f' is at index 16, 'l' is at index 10, 'a' is at index 18, and 'g' is at index 13. So, I could get the target value doing this equation, multiplying the indices by increasing powers of 26 (0 to 3) and adding the products together:

`(16*1)+(10*26)+(18*26^2)+(13*26^3) = 240932`

I ran this in the console, which moved the slider for me and set the text to "I want the flag":

`document.getElementById("val").value = 240932;`

It worked - I clicked submit, and I got the flag.

`lactf{gregor_samsa_awoke_from_wait_thats_the_wrong_book}`


## Takeaways
- In web challenges, after you explore the website, do static analysis
- Often in CTFs, it's easier to run console commands than to try to do something manually on the website

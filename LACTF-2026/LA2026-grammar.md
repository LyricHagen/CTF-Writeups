# Grammar (February 7, 2026)

**Category:** misc  
**Points:** 103  
**Files Provided:** tree.png, grammar-notes.txt  

## Challenge Description

"Inspired by CS 131 Programming Languages, I decided to make a context-free grammar in EBNF for my flag! But it looks like some squirrels have eaten away at the parse tree..."

## Step 1: Exploring

This may be a hard-to-visualize description, but tree.png showed a black circle at the top, with 7 lines going out to 7 black circles, them a row of multicolored circles with lines going down to stacks of black circles, with a line of black squares along the bottom.

grammar-notes.txt provided an EBNF grammar, which defined a flag as a start ("lactf{"), followed by words separated by underscores, followed by an end ("}"). Each word is made of fragments, and each fragment is one of 5 types:

cd = consonant, digit; vc = vowel, consonant; vd = vowel, digit; c = consonant; d = digit.

The terminals were defined as chains of productions:
```
con = "f" | con2;
con2 = "g" | con3;
con3 = "p" | con4;
con4 = "t" | con5;
con5 = "r";

vow = "e" | vow2;
vow2 = "o" | vow3;
vow3 = "u";

dig = "0" | dig2;
dig2 = "1" | dig3;
dig3 = "4" | dig4;
dig4 = "5";
```

So, for any nonterminal, the number of intermediate black circles in its stack tells you which terminal to pick: 0 black circles = first option, 1 black circle = second option, and so on.

## Step 2: The Approach

The notes clarified that circles are nonterminals and boxes are terminals, with colored circles representing specific named nonterminals and black circles being ambiguous.
The color pattern across the 3 words was given as ABACDE BC EAEA, where A = orange, B = blue, C = green, R = red, and E = purple.
I manually went through the photo and counted the black circles. A branch into two stacks meant a two-part fragment like cd, vc, or vd. A single column meant c or d.
Counting with depth as (number of black circles - 1):

Word 1: O(2); U(4,0); O(0); G(0,3); R(3); P(1,4)

Word 2: U(2,2); G(2,1)

Word 3: P(0,1); O(1); P(0,4); O(3)

Rather than guessing the color-to-type mapping manually, I wrote a Python script that tried all valid combinations, like singles (c/d) for O and R, doubles (cd/vc/vd) for U, G, and P, and filtering out the ones that went out of bounds.

```
con = ["f", "g", "p", "t", "r"]
vow = ["e", "o", "u"]
dig = ["0", "1", "4", "5"]

words = [
    [('O',[2]),('U',[4,0]),('O',[0]),('G',[0,3]),('R',[3]),('P',[1,4])],
    [('U',[2,2]),('G',[2,1])],
    [('P',[0,1]),('O',[1]),('P',[0,4]),('O',[3])],
]

def decode(word, mapping):
    result = []
    for color, depths in word:
        t = mapping[color]
        try:
            if t == 'c': result += [con[depths[0]]]
            elif t == 'd': result += [dig[depths[0]]]
            elif t == 'cd': result += [con[depths[0]], dig[depths[1]]]
            elif t == 'vc': result += [vow[depths[0]], con[depths[1]]]
            elif t == 'vd': result += [vow[depths[0]], dig[depths[1]]]
        except IndexError:
            return None
    return ''.join(result)

for o in ['c','d']:
    for u in ['cd','vc','vd']:
        for g in ['cd','vc','vd']:
            for r in ['c','d']:
                for p in ['cd','vc','vd']:
                    m = {'O':o,'U':u,'G':g,'R':r,'P':p}
                    ws = [decode(w, m) for w in words]
                    if all(ws):
                        print(f"lactf{{{ws[0]}_{ws[1]}_{ws[2]}}}")
```

This produced 12 candidate flags, and one was clearly the right one.

`lactf{pr0fe55or_p4u1_eggert}`

## Takeaways
- When you can't determine a mapping analytically, enumerate all valid combinations

# ooo (February 7, 2026)

**Category:** rev  
**Points:** 100  
**Files Provided:** ooo.py

## Challenge Description

"Surely everyone knows the difference between the Cyrillic small letter o, the Greek small letter omicron, and the Latin small letter o, right? :)"

## Step 1: Exploring

I opened the provided Python file and saw a wall of functions named after Unicode characters that look like 'o'. For example, the first three:
```
def о(a, b):
    return a+b
def ο(a, b):
    return a-b
def օ(a, b):
    return a*b
```
And an array:
```
ὁ = [205, 196, 215, 218, 225, 226, 1189, 2045, 2372, 9300, 8304, 660, 8243, 16057, 16113, 16057, 16004, 16007, 16006, 8561, 805, 346, 195, 201, 154, 146, 223]
```

Basically, the core logic was:
- Functions perform basic operations (add, subtract, multiply, etc)
- A target array contains 27 integers
- The program checks if `guess[i] + guess[i+1] == target[i]` for each position

## Step 2: The Approach

This check mattered most, since if it returned true, I *wouldn't* get the flag:
```
if (о(ὄ(ό,ὃ),ὂ(ό,ὃ)) != ὁ[ơ(ö,ȯ(օ(ό,ὃ),ό))]):
```

And looking at the functions:
- о(a,b) = a+b
- ὄ(a,b) = a (returns first arg)
- ὂ(a,b) = b (returns second arg)

So this simplifies to: `a + b != target[index]` - each adjacent pair of characters must sum to the target value at that index.

I wrote a Python script that propagates forward from the known prefix.
```
target = [205, 196, 215, 218, 225, 226, 1189, 2045, 2372, 9300, 8304, 660, 8243, 16057, 16113, 16057, 16004, 16007, 16006, 8561, 805, 346, 195, 201, 154, 146, 223]
flag = list("lactf{")

for i in range(len(flag)-1, len(target)):
    next_char = target[i] - ord(flag[i])
    flag.append(chr(next_char))

print(''.join(flag))
```

It outputted the flag.

`lactf{gоοօỏơóὀόὸὁὃὄὂȯöd_j0b}`


## Takeaways
- A lot of constraint-based CTFs often allow forward propagation from known values; when you have `f(x[i], x[i+1]) = constant`, you can solve iteratively if you know the starting values

# Six Seven (February 7, 2026)

**Category:** crypto  
**Points:** 102  
**Files Provided:** chall.py  
**Connection:** nc chall.lac.tf 31180

## Challenge Description

"When bro uses RSA encryption to talk crap behind my back but I lowkenuinely just factor his public key because I know he picked [67]* as his primes"

## Step 1: Exploring

When you connect to the server, it gives you a curl command as proof of work. The provided Python script implements RSA encryption, but with a unique feature - the primes p and q only consist of the digits 6 and 7, looking at the key function. 

```
def generate_67_prime(length: int) -> int:
    while True:
        digits = [secrets.choice("67") for _ in range(length - 1)]
        digits.append("7")

        test = int("".join(digits))
        if isPrime(test, false_positive_prob=1e-12):
            return test
```

This makes it easier to find out what those primes are. We know that the last digit has to be 7 (since it wouldn't be prime if it ended in 6.) It should work to write a script to solve for the digits one by one.

## Step 2: The Approach

Since multiplication carries right to left, we can determine the i-th digits of p and q by looking at the i-th digit of n. By starting at the ones place (which is 7 for both p and q,) we can use DFS to find the factors.
If a combination of digits at a certain position doesn't produce the correct digit in n (considering the carry,) we can prune that branch and try a different combination.

This script worked (n and c are provided by the challenge server):

```
import sys
from Crypto.Util.number import long_to_bytes

sys.setrecursionlimit(3000)

n = 51202956324807380818338545478187089431218212535702934445085394441459541360742080988492198755389272427711443695387558607572707591659547298251321294196613973665520814155231223665671907401900207258226131729066003599912888193851775549994895306495552655464617558118703078260579487498465320378132140091221353873698642019291970140534392148917693794122517944228503126779986646323517087062469449573489083820708123637050571583036796077127557857438857838262815135250826595832103376800775286494580241781267251070284691069329
c = 20495817949262349343290941507657596170548640633721636664437188980303849207309002911854545707487437880408407057017206234684881980638528960426470593376489254399651397760233026102659078853537938523174078589630408393411142994088247070561475756984130026435777948191214444776849169651164802283047023658895481706978566200464421668380164366849858248625008105083922004820558501944049242823643603333506368607040549304833781716196315750034933388056630666769744063795994794661721496411808803255900268546960653557611713976011
e = 65537

def find_primes(curr_p, curr_q, depth):
    if depth == 256:
        if curr_p * curr_q == n:
            return curr_p, curr_q
        return None

    mod = 10**(depth + 1)
    target = n % mod

    for dp in [6, 7]:
        for dq in [6, 7]:
            next_p = dp * (10**depth) + curr_p
            next_q = dq * (10**depth) + curr_q
            
            if (next_p * next_q) % mod == target:
                res = find_primes(next_p, next_q, depth + 1)
                if res:
                    return res
    return None

print("factoring.")
result = find_primes(7, 7, 1)

if result:
    p, q = result
    print(f"p: {p}")
    print(f"q: {q}")
    
    phi = (p - 1) * (q - 1)
    d = pow(e, -1, phi)
    m = pow(c, d, n)
    
    flag = long_to_bytes(m)
    print(f"flag: {flag.decode()}")
else:
    print("didn't work.")
```

Providing the flag:

`lactf{wh4t_67s_15_blud_f4ct0r1ng_15_blud_31nst31n}`

## Takeaways
- If RSA primes only use specific digits, that's a security vulnerability because that makes them more predictable
- Digit-by-digit reconstruction is powerful in solving equations with large numbers and helpful restrictions

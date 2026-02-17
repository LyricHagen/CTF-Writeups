# The Clock (February 7, 2026)

**Category:** crypto  
**Points:** 102  
**Files Provided:** chall.py, output.txt

## Challenge Description

"Don't run out of time."

## Step 1: Exploring

output.txt was just the following:
```
Alice's public key:  (13109366899209289301676180036151662757744653412475893615415990437597518621948, 5214723011482927364940019305510447986283757364508376959496938374504175747801)
Bob's public key:  (1970812974353385315040605739189121087177682987805959975185933521200533840941, 12973039444480670818762166333866292061530850590498312261363790018126209960024)
Encrypted flag:  d345a465538e3babd495cd89b43a224ac93614e987dfb4a6d3196e2d0b3b57d9
```

chall.py was more of a detailed cryptosystem, for which the first part that seemed important was the clockadd(P1,P2) function:

```
def clockadd(P1,P2):
  x1,y1 = P1
  x2,y2 = P2
  x3 = x1*y2+y1*x2
  y3 = y1*y2-x1*x2
  return x3,y3
```

These formulas are the additional formulas for sine and cosine, implying that the challenge is a Diffie-Hellman key exchange.
On a clock curve, every point (x, y) must satisfy x^2 + y^2 = 1 (mod p), so I could find the GCD of x^2 + y^2 - 1 to find p. I used an online calculator to get the value of p.

## Step 2: The Approach

Since it's a clock curve, usually p+1 is the group order if p = 3 (mod 4). I checked p+1 on FactorDB and saw that it nicely factored into many small primes.

As far as I understood, this was a setup for the Pohlig-Hellman algorithm, so I could solve the secret modulo each small prime factor and then combine them using the Chinese Remainder Theorem.
This Python script ended up being the working one, performing the Pohlig-Hellman attack:

```
import hashlib
from Crypto.Cipher import AES

p = 13767529254441196841515381394007440393432406281042568706344277693298736356611
factors = [39623, 41849, 42773, 46511, 47951, 50587, 50741, 51971, 54983, 55511, 56377, 58733, 61843, 63391, 63839, 64489]
all_factors = [4] + factors

base_pt = (13187661168110324954294058945757101408527953727379258599969622948218380874617, 5650730937120921351586377003219139165467571376033493483369229779706160055207)
alice_pub = (13109366899209289301676180036151662757744653412475893615415990437597518621948, 5214723011482927364940019305510447986283757364508376959496938374504175747801)
bob_pub = (1970812974353385315040605739189121087177682987805959975185933521200533840941, 12973039444480670818762166333866292061530850590498312261363790018126209960024)
enc_flag = "d345a465538e3babd495cd89b43a224ac93614e987dfb4a6d3196e2d0b3b57d9"

def clock_mul(P1, P2):
    x1, y1 = P1
    x2, y2 = P2
    return (x1*y2 + y1*x2) % p, (y1*y2 - x1*x2) % p

def clock_pow(P, n):
    res = (0, 1)
    while n > 0:
        if n % 2 == 1: res = clock_mul(res, P)
        P = clock_mul(P, P)
        n //= 2
    return res

def chinese_remainder(n, a):
    from functools import reduce
    sum = 0
    prod = reduce(lambda a, b: a * b, n)
    for n_i, a_i in zip(n, a):
        p_i = prod // n_i
        sum += a_i * pow(p_i, -1, n_i) * p_i
    return sum % prod

order = p + 1
reminders = []

for q in all_factors:
    g_q = clock_pow(base_pt, order // q)
    a_q = clock_pow(alice_pub, order // q)
    
    found = False
    curr = (0, 1)
    for i in range(q):
        if curr == a_q:
            reminders.append(i)
            found = True
            break
        curr = clock_mul(curr, g_q)
    if not found:
        print(f"failed on factor {q}")

alice_secret = chinese_remainder(all_factors, reminders)
print(f"recovered Alice's secret: {alice_secret}")

ss = clock_pow(bob_pub, alice_secret)
key = hashlib.md5(f"{ss[0]},{ss[1]}".encode()).digest()
cipher = AES.new(key, AES.MODE_ECB)
print("flag:", cipher.decrypt(bytes.fromhex(enc_flag)).decode())
```

Resulting in:

`lactf{t1m3_c0m3s_f4r_u_4all}`

## Takeaways
- If a prime isn't provided in a curve-based challenge, try to recover it using the curve equation and GCD.
- Small prime factors imply that the Pohlig-Hellman attack will be effective.

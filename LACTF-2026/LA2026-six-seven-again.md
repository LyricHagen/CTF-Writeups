# Six Seven Again (February 7, 2026)

**Category:** crypto  
**Points:** 109  
**Files Provided:** chall.py  
**Connection:** nc chall.lac.tf 31181

## Challenge Description

"LA CTF will take place on Feburary 6 and Feburary 7, 2026."

## Step 1: Looking Around

The provided Python file shows how the RSA keys are being generated. The prime q is a standard 670-bit prime, but p is generated using a specific unique template: 67 6s as the prefix, 67 characters either 6 or 7 in the middle, and then 67 7s as the suffix.

```
def generate_super_67_prime() -> int:
    while True:
        digits = ["6"] * 67
        digits += [secrets.choice("67") for _ in range(67)]
        digits += ["7"] * 67

        test = int("".join(digits))
        if isPrime(test, false_positive_prob=1e-12):
            return test
```

Based on bit amounts, we can assume p and q are roughly the same size. We can't brute force, but since we know so much about p, we can use the Coppersmith's attack.

## Step 2: The Solution

In RSA, if you know a significant amount of the bits/digits of a prime, you can generally find the remaining unknown part. p = prefix + middle + (x*10^67) + suffix, where x is an integer of digits either 0 or 1, representing the choice between 6 and 7 in the middle section.
Since SageMath's small_roots() needs a monic polynomial, I multiplied the entire polynomial by the modular inverse of 10^67 mod n.

I got the proof of work from the server and received values for n and c, so I could then run this SageMath script:

```
n = [n goes here; from nc server]
c = [c goes here; from nc server]
e = 65537

prefix_val = int("6" * 67 + "0" * 134)
base_middle = int("6" * 67 + "0" * 67)
suffix_val = int("7" * 67)
constant_part = prefix_val + base_middle + suffix_val

P.<x> = PolynomialRing(Zmod(n))
inv_coeff = pow(10**67, -1, n)
f = x + (constant_part * inv_coeff)

X = int("1" * 67)
roots = f.small_roots(X=X, beta=0.48)

if roots:
    p_val = int(constant_part + int(roots[0]) * 10**67)
    q_val = n // p_val
    d = pow(e, -1, (p_val-1)*(q_val-1))
    m = pow(c, d, n)
    print(bytes.fromhex(hex(m)[2:]).decode())
```

Providing the flag:

`lactf{n_h4s_1337_b1ts_b3c4us3_667+670=1337}`

## Takeaways
- Primes with highly structured patterns are vulnerable to Coppersmith-like attacks.

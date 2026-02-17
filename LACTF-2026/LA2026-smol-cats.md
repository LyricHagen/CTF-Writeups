# Smol Cats (February 7, 2026)

**Category:** crypto  
**Points:** 101  
**Connection:** `nc chall.lac.tf 31225`

## Challenge Description

"My cat walked across my keyboard and made this RSA implementation, encrypting the location of the treats they stole from me! However, they already got fed twice today, and are already overweight and needs to lose some weight, so I cannot let them eat more treats. Can you defeat my cat's encryption so I can find their secret stash of treats and keep my cat from overeating?"

## Step 1: Exploring

Upon connecting to the server, I saw 3 given numbers, presumably for RSA: n, e, and c. n and c were very long numbers, and e was 65537. 
Based on the challenge name and the length of n, I assumed the vulnerability here is the shortness of the primes.

## Step 2: The Exploit

Basically, the solution would be to reverse RSA's cryptographic math, which I read up on online before writing the script.
At first, I wrote a script that factored n, but that took too long. So, I took the value of n to FactorDB to factor it for me. It returned the two primes, p and q, which I could use to calculate the totient phi, i.e. (p-1)(q-1).
With phi, I could find d = e^(-1) mod phi, and then m = c^d (mod n).

Here's the Python script that gave me the number that I could give back to the netcat prompt to get the flag:
```
python3 << 'EOF'
p = 380482810003058866731517032917
q = 411803818318780776510100465561
n = 1566842721141464224911889139468732372740122816731181294157297
e = 65537
c = 792846592787735820272443914422049799022252281541176553211701

phi = (p - 1) * (q - 1)
d = pow(e, -1, phi)
m = pow(c, d, n)
print(m)
EOF
```

`lactf{sm0l_pr1m3s_4r3_n0t_s3cur3}`

## Takeaways
- RSA is only secure if the primes are sufficiently large.
- FactorDB is useful for finding primes when it takes too long to compute them manually.

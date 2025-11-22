# Guessy Timezones (November 22, 2025)

**Category:** rev  
**Points:** 50  
**Files Provided:** challenge (ELF executable), deploy.bat, deploy.sh, flag.txt, sha256sum

## Challenge Description

"This is a multiline challenge description!
Feel free to add more lines.
NOTE: The handout contains empty `deploy.sh` and `deploy.bat` files. Those files don't matter and you can ignore them!"

Connection: `nc challs.glacierctf.com 13386`

## Step 1: Exploring the Application

When connecting to the server, we're greeted with a timezone clock app that has several commands:
- `help`: display help message
- `zones`: list available timezones
- `time`: get time for a specific timezone
- `solve`: enter the "magic number" to get the flag
- `exit`: close the app

The challenge asks us to guess a "magic number" to win.

## Step 2: Reverse Engineering with Ghidra

I opened the challenge binary in Ghidra and examined the `printTimeAndSetWinningNumber` function:
```c
strftime(acStack_98,0x10,"%Y%m%d%H%M%S",__tp);
uVar1 = strtol(acStack_98,(char **)0x0,10);
__printf_chk(2,"THIS IS THE STRING: %s\n",acStack_98);
__printf_chk(2,"THIS IS THE SEED: %d\n",uVar1 & 0xffffffff);
srand((uint)uVar1);
winning_number = rand();
```

The key findings:
- The program takes the current time in the specified timezone
- Formats it as `YYYYMMDDHHMMSS` (e.g., `20251122220244`)
- Uses that timestamp as a seed for `srand()`
- Generates the winning number with `rand()`
- **Prints both the seed string and numeric seed value to stdout!**

## Step 3: Exploiting the Debug Output

The program helpfully prints the seed it's using. To solve:

1. Connect to the server and run the `time` command with any timezone:
```
> time
Please enter the timezone (format: Continent/City): America/Los_Angeles

Time in America/Los_Angeles : 2025-11-22 22:02:44 America

THIS IS THE STRING: 20251122220244
THIS IS THE SEED: 351419604
```

2. The seed is printed right there: `351419604`

## Step 4: Generating the Winning Number

Since the server runs Linux and uses Linux's `rand()` implementation, I needed to replicate this in a Linux environment. I used Docker:
```bash
docker run --rm -it gcc:latest bash
```

Inside the Docker container, I created a C program:
```c
#include <stdio.h>
#include <stdlib.h>

int main() {
    srand(351419604);  // Use the seed from the server
    printf("%d\n", rand());
    return 0;
}
```

Compiled and ran it:
```bash
gcc solve.c -o solve
./solve
```

This outputs the winning number that matches what the server generated.

## Step 5: Submitting the Answer

Back in the nc session:
```
> solve
Enter the magic number: [output from C program]
```

The flag is then revealed: `gctf{S33ded_P5eud0_R4nd0mn3ss_1s_4lw4ys_b4d!}`.

## Takeaways

- Debug output left in production code can give useful info.
- Random number generation with predictable seeds (like timestamps) is not secure.
- Different operating systems have different `rand()` implementations, e.g. macOS and Linux produce different results with the same seed.
- Time-based challenges require working quickly before the connection times out and the seed changes (in other challenges, this may require automation.)

# Tic Tac No (February 7, 2026)

**Category:** pwn  
**Points:** 101  
**Files Provided:** chall.c, Dockerfile, chall  
**Connection:** `nc chall.lac.tf 30001`  

## Challenge Description

"Tic-tac-toe is a draw when played perfectly. Can you be more perfect than my perfect bot?"

## Step 1: Exploring

Looking at the provided code, chall.c implements a tic-tac-toe game where the computer uses a minimax algorithm to ensure it never loses. The goal is to reach the code block that opens flag.txt, which only triggers if winner == player. I tried connecting to the server to try out the game, just playing normally, and lost all my attempts; since I could go out of bounds, I tried to win by placing X's in areas where the computer couldn't place anything, but that wasn't effective at all.

Looking at the code again, the playerMove() function has an OOB write vulnerability:
```
void playerMove() {
   int x, y;
   do{
      printf("Enter row #(1-3): ");
      scanf("%d", &x);
      printf("Enter column #(1-3): ");
      scanf("%d", &y);
      int index = (x-1)*3+(y-1);
      if(index >= 0 && index < 9 && board[index] != ' '){
         printf("Invalid move.\n");
      }else{
         board[index] = player; // Should be safe, given that the user cannot overwrite tiles on the board
         break;
      }
   }while(1);
}
```
The if statement checks if a spot is already taken iff the index is within the valid board range, i.e. 0-8. If we provide coordinates resulting in an index outside of that range, the index-checking condition becomes false, and writes our player character 'X' to memory outside the board array. (I tried the obvious ones like position 5, 5 - positions that made indices clearly greater than 8. Those didn't work, so I had to look into the memory.)

## Step 2: Analysis

I looked at the chall file in Ghidra to see if there was anything useful memory-wise in there. In the Listing View, I found the addresses of the player, computer, and board, and subtracted `computer-board` to get -22 bytes, so I assumed that with an index of -22, I'd override the computer's O-placing. (The calculation was: 0x104051 - 0x104067 = -0x16 = -(16+6) = -22.) Based on playerMove()'s `int index = (x-1)*3+(y-1)`, the position -5, -3 would result in board[-22] = 'X' (since (-5-1)*3 + (-3-1) = -22,) changing the computer's variable from 'O' to 'X' and helping me win.

This was the series of moves that worked:

-5, -3

-5, -4

2, 2

`lactf{th3_0nly_w1nn1ng_m0ve_1s_t0_p1ay}`


## Takeaways
- As with most pwn challenges, if a program doesn't strictly validate that an index is within bounds, you can overwrite nearby variables in memory

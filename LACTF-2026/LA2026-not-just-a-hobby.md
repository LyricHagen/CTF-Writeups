# Not Just a Hobby (February 7, 2026)

**Category:** misc  
**Points:** 107  
**Files Provided:** v.v  

## Challenge Description

"It's not just a hobby!!!"

## Step 1: Exploring

The file provided, v.v, was a very long Verilog hardware description language file. Here's how the file started:

```
module v (
	input [6:0] x,
	input [6:0] y,
	output reg [3:0] vga_r,
	output reg [3:0] vga_g,
	output reg [3:0] vga_b
);

	always @(*) begin
		if ((x == 7'd588 && y == 43) || (x == 482 && y == 7'd513) || (x == 7'd279 && y == 617) ||
		(x == 210 && y == 424) || (x == 7'd161 && y == 7'd596) || (x == 7'd356 && y == 503) ||
		(x == 290 && y == 7'd575) || (x == 7'd235 && y == 639) || (x == 7'd366 && y == 7'd42) ||
		(x == 7'd404 && y == 639) || (x == 599 && y == 238) || (x == 7'd632 && y == 7'd202) ||
		(x == 7'd185 && y == 7'd593) || (x == 7'd416 && y == 7'd448) || (x == 586 && y == 518) ||
		(x == 32 && y == 7'd272) || (x == 587 && y == 7'd415) || (x == 7'd394 && y == 419) ||
		(x == 518 && y == 213) || (x == 7'd17 && y == 7'd338) || (x == 7'd242 && y ==
```

And here's how it ended:

```
		(x == 7'd504 && y == 7'd547) || (x == 7'd394 && y == 7'd476) || (x == 7'd121 && y == 270) ||
		(x == 7'd369 && y == 7'd322) || (x == 200 && y == 7'd392) || (x == 7'd199 && y == 7'd132) ||
		(x == 215 && y == 592) || (x == 7'd508 && y == 7'd515) || (x == 326 && y == 213) ||
		(x == 429 && y == 392)) begin
			vga_r = 4'h0;
			vga_g = 4'h0;
			vga_b = 4'h0;
		end
	end
endmodule
```

It defines a module v that takes in 2 7-bit inputs, x and y, and 3 4-bit VGA color outputs vga_r, vga_g, and vga_b. The logic is a single, very long if statement, that sets the output to black when the current (x, y) matches any of the listed pairs.
So, the clear solution was to visualize this file in some way, and surely it would show the flag.

## Step 2: The Solution

I first tried extracting all the coordinates and plotting them with a simple script. That just gave me static.

Although the 7-bit signal width ([6:0]) means coordinates should be capped at 127, the file contains values up to 640, implying that it "wraps around." Applying module 128 to all of them still produced noise.
The key insight was that some coordinates had the 7'd size prefix on both x and y, and some had only one or neither. Perhaps, only coordinates where both values had the 7'd prefix were real pixels.

I wrote a script to split the coordinates into two groups based on whether both had the 7'd prefix.

```
import re
from PIL import Image
import numpy as np

with open("v.v", "r") as f:
    content = f.read()

coords = []
for line in content.split("||"):
    match = re.search(r'\(x == (7\'d)?(\d+) && y == (7\'d)?(\d+)\)', line)
    if match:
        x_prefix, x_val, y_prefix, y_val = match.groups()
        if x_prefix and y_prefix:
            coords.append((int(x_val) % 128, int(y_val) % 128))

img = np.ones((128, 128, 3), dtype=np.uint8) * 255
for x, y in coords:
    img[y, x] = [0, 0, 0]

Image.fromarray(img).resize((512, 512), Image.NEAREST).save("flag.png")
```

Rendering those coordinates on a 128x128 grid with modulo 128 applied, I could see the flag written across the image.

`lactf{graph1c_d3sign_is_My_PA55i0N!!1!}`

## Takeaways
- When something looks like static when visualized, look for some kind of filter - something's throwing it off.
- Intentional inconsistencies can be part of the challenge (in this case, the 7'd Verilog prefix.)

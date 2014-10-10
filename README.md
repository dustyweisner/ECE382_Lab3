ECE382_Lab3
===========

Serial Peripheral Interface - "I/O"


__*LOGIC ANALYZER*__

I started the "SPI - I/O" Lab with the logic analyzer section. First, I tested my Nokia 1202 Booster Pack to see if pressing the button, SW3, would create little lines given the program, Lab3.asm. The picture given below shows that the input and output are working fine.

![](https://github.com/dustyweisner/ECE382_Lab3/blob/master/Images/LCD_SW3_IO.jpg?raw=true)

When SW3 is detected as being pressed and released in the lines of code below (lines 56-62 of lab3.asm), the MSP430 generates 4 packets of data that are sent to the Nokia 1202 display, causing a vertical bar to be drawn.

            while1:
                        bit.b	  #8, &P2IN				; bit 3 of P1IN set?
                        jnz 	  while1					; Yes, branch back and wait
            while0:
                        bit.b	  #8, &P2IN				; bit 3 of P1IN clear?
                        jz		  while0					; Yes, branch back and wait

The following table was created by finding the 4 calls to writeNokiaByte that generate these packets,scanning the nearby code to determine the parameters being passed into this subroutine, and writing a brief description of what is trying to be accomplished by each call to writeNokiaByte:

|__Line__|__R12__|__R13__|__Purpose__|
|:-----|:-----|:-----|:-----|
|66|`0x0001`|`0x00E7`| Draws an 8 pixel high beam with a 2 pixel hole in the center|
|276|`0x0000`|`0x00B1`|Masks out any weird upper nibble bits and masks in "B0" as the prefix for a page address|
|288|`0x0000`|`0x0010`|Masks out upper nibble and 10 is the prefix for a upper column address - width of beam|
|294|`0x0000`|`0x0001`|Writes a command and sets up call to make a copy of the top of the stack|

Next I configured the logic analyzer to capture the waveform generated when the SW3 button is pressed and released, as shown below:

![](https://github.com/dustyweisner/ECE382_Lab3/blob/master/Images/AnalyzerCommandWaveform.JPG?raw=true)

Then I decoded the data bits of each 9-bit waveform by separating out the MSB, which indicates command or data.

|__Line__|__Command/Data__|__8-Bit Packet__|
|:-----|:-----|:-----|
|66|Data|`0xE7`|
|276|Command|`0xB1`|
|288|Command|`0x10`|
|294|Command|`0x01`|

After each button press, the command 8-bit packet for both lines 276 and 294 increase by 1. This is indicated on the LCD screen by the line with the gap being transposed down the screen (or up depending on orientation). The data packet at line 66 is never changed because it is the information which draws the height of the beam. The Command packet at line 288 also never changes because it is the set width of the beam.

From lines 93 to 100, Reset is cleared. The following waveform was captured in the logic analyzer, showing reset on the falling edge:

![](https://github.com/dustyweisner/ECE382_Lab3/blob/master/Images/AnalyzerResetFallingEdge.JPG?raw=true)

The native write operation to the Nokia 1202 will overwrite any information that is was on the display with new information. However, that is not the best course of action in this application. The new bits being added to the image may be merged using the AND, OR, XOR operators. To do this, I treated a black pixel as a logic 1 and a white pixel as a logic 0. The pixel values from the same locations are combined using a logical operator and placed at the corresponding location in the destination imaged. The results are located below.

![](https://raw.githubusercontent.com/dustyweisner/ECE382_Lab3/master/Images/Lab3AnalyzerBitMap.bmp?raw=true)


__*CODE FUNCTIONALITY AND RESULTS*__

__Required functionality__ was to create a block on the LCD that is 8x8 pixels. The location of the block was passed into the subroutine via r12 and r13. __A-functionality__ was to move the 8-pixel block one block in the direction of the pressed button (up, down, left, right). Both functionalities were demoed and had perfect results with full points.


__Required Functionality__
To begin to make an 8x8 block, I first looked at the given code to find out how pixels were displayed. The given code started out displaying an 8-pixel line with a 2-pixel gap with each button press. First, I decided I needed to discover how the line was drawn from the code. From the analyzer logic, I quickly saw a pattern in the Data packet of `0xE7`: the number `0xE7` written in binary is `11100111`, which if 1 represents an on pixel and 0 an off, the number `0xE7` is exactly the 8-pixel line with the 2-pixel gap. Consequently, I looked for code that contained the data packet. I found the following code:

                        mov		#0xE7, R13
                        call	  #writeNokiaByte

To change the line to an 8-pixel line with no gap, I converted `11111111` to hex, `0xFF`, which worked. My next big problem was to find out how to make 8 8-pixel lines next to each other to make an 8x8 block. I decided the most efficient way would be using a for loop in assembly style of course. Before I made the for loop, however, I needed to know which register changed the columns. After playing around with the code, I found that `#writeNokiaByte` had a built in increment for the row. I honed that for my for loop in the following code:

                        mov		#0xFF, R13
                        mov		#0, r9
            writeBlock	call	#writeNokiaByte
                        inc.b	r9
                        cmp		#8, r9
                        jnz		writeBlock

This chunk of code successfully created an 8x8 block, thus fulfilling the requirements of the lab.


__A Functionality__
To acheive A functionality, I first looked for something in the code that would tell me how to configure the arrow buttons to move the 8x8 block. Finally I found this chunk of comment code:

            ; Buttons on the Nokia 1202
            ;	S1		P2.1		Right
            ;	S2		P2.2		Left
            ;	S3		P2.3		Aux
            ;	S4		P2.4		Bottom
            ;	S5		P2.5		Up
            ;
            ;	7 6 5 4 3 2 1 0
            ;	0 0 1 1 1 1 1 0		0x3E

I decided that I didn't need the Aux button for this code, and instead used the Right, Left, Bottom, and Up buttons. Because I needed to use the button push still, I just converted the bit 3 code for the "while button is pushed" to code for bits 1 (right), 2 (left), 4 (bottom), and 5 (up). The following code is what resulted:

            while1:								; 0 = pressed
            	bit.b	#2, &P2IN				;RIGHT PRESS? bit 1 of P1IN set?
            	jz		clrRight
            	bit.b	#4, &P2IN				;LEFT PRESS?  bit 2 of P1IN set?
            	jz		clrLeft
            	bit.b	#16, &P2IN				;BOTTOM PRESS? bit 4 of P1IN set?
            	jz		clrBottom
            	bit.b	#32, &P2IN				;TOP PRESS?  bit 5 of P1IN set?
            	jz 		clrTop
            	jmp 	while1

I also needed to check for if the button is released before the actual movement code. So not only did I wait for the button to clear but I called three subroutines: to first clear the display, to move the box in the direction pushed, and to write the box on the display. Here was my code for such logic:

            clrRight                bit.b	#2, &P2IN		;1 = NOT PRESSED
            		jz          clrRight
            		call	#clearDisplay	;Clear Display
            		call	#writeRight		;Changes values
            		call	#write		;Draws values
            		jmp	while1
            		
            clrLeft	            bit.b	#4, &P2IN
            		jz	clrLeft
            		call	#clearDisplay
            		call	#writeLeft
            		call	#write
            		jmp	while1
            		
            clrBottom	            bit.b	#16, &P2IN
            		jz	clrBottom
            		call	#clearDisplay
            		call	#writeBottom
            		call	#write
            		jmp         while1
            		
            clrTop		bit.b	#32, &P2IN	           	; bit of P2IN clear?
            		jz	clrTop
            		call	#clearDisplay
            		call	#writeTop
            		call	#write
            		jmp	while1

`#clearDisplay` was fortunately already made, but the `#writeRight`, etc., had to be created using logic of the register usage for changing columns and rows. For `#writeRight` I used `add #8, r11` to shift the column pointer right. Oppositely, for `#writeLeft` I used `sub #8, r11`. For `#writeBottom` I used `inc r10` to shift the row pointer down, and for `#writeTop` I used `dec r10` to shift the row pointer up. 

Finally, to display the moved block, I used `mov #NOKIA_DATA, R12` and the block of code used to make an 8x8 block and returned to my `while1` loop. This correctly did the A functionality but I needed to fix one thing: the block would not start of displayed on the screen. This easy fix only entailed me using this same coding in the main before the `while1` loop starts.

This concludes the A functionality of Lab 3 Serial Peripheral Interface - "I/O".

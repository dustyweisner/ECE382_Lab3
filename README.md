ECE382_Lab3
===========

Serial Peripheral Interface - "I/O"


__*LOGIC ANALYZER*__

I started the "SPI - I/O" Lab with the logic analyzer section. First, I tested my Nokia 1202 Booster Pack to see if pressing the button, SW3, would create little lines given the program, Lab3.asm. The picture given below shows that the input and output are working fine.

![](https://github.com/dustyweisner/ECE382_Lab3/blob/master/Images/LCD_SW3_IO.jpg?raw=true)

When SW3 is detected as being pressed and released in the lines of code below (lines 56-62 of lab3.asm), the MSP430 generates 4 packets of data that are sent to the Nokia 1202 display, causing a vertical bar to be drawn.

`while1:`  
`	       bit.b	#8, &P2IN				; bit 3 of P1IN set?       `  
`	       jnz 	while1						; Yes, branch back and wait`  
`while0:                                                   `  
`	       bit.b	#8, &P2IN				; bit 3 of P1IN clear?     `  
`	       jz		while0						; Yes, branch back and wait`

The following table was created by finding the 4 calls to writeNokiaByte that generate these packets,scanning the nearby code to determine the parameters being passed into this subroutine, and writing a brief description of what is trying to be accomplished by each call to writeNokiaByte:

|__Line__|__R12__|__R13__|__Purpose__|
|:-----|:-----|:-----|:-----|:-----|
|||||
|||||
|||||
|||||

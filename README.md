# HelloPIC_Interrupts
Sample code for the PIC 12F1572 interrupt handling.

For a very basic introduction about Interrupts on PIC see:
https://picguides.com/beginner/interrupts.php

The Interrupt handler gets registered in main.c, with     
`IOCAF2_SetInterruptHandler(IOCAF2_CGK_InterruptHandler);`

The ISR (`IOCAF2_CGK_InterruptHandler()`) is called everytime the RA2 is pulled to LOW.

More in detail information about De-Bouncing Switches: 
http://www.ganssle.com/debouncing-pt2.htm

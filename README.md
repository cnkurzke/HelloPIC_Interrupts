# About

This is part of my adventure of learning how to use PIC Microcontrollers in the real world.
If you want to learn more about what a PIC is, here are plenty of online resources. E.g. see here:

* https://en.wikipedia.org/wiki/PIC_microcontrollers
* https://www.microchip.com/en-us/products/microcontrollers-and-microprocessors
* https://pic-microcontroller.com/pic16f877-based-projects-pic-microcontroller-downloadable/

My intention is to document my learning journey, so others can hopefully find it useful on
their own research and learning path. I strongly encourage everyone who finds a mistake, 
be it an oversight or something out of date, *please file a bug, or even better, contribute a fix!*

## My Setup

I try to do this as cleaply as possible, so that it can be reproduced by everyone.

My primary workstation is a standard PC with Ubuntu (currently version 20.04), and I use the 
PicKit 3. If you intend to purchase a programmer, be aware that the PicKit 2 is no longer 
supported by the current (2022) generation of development tools.

As the software environmnet, I am not happy with the Microchip tools, but it is the quickest way to
get started.
In fact, I installed a Virtual Machine, and run the MPLab IDE inside the VM. But if you have a
more "traditional" operating system, you should be fine to run this on your standard computer.
Follow the steps here:
https://www.microchip.com/en-us/tools-resources/develop/mplab-x-ide

# What Does It Do

I started with a very simple problem. For a car I'm currently restoring, I needed something
to control the "Transmission Reverse Lock-Out". For a commercial device, look at 
https://shiftsst.com/tremec-6-speed-reverse-lockout-control-module.html

What's relevant for this level of detail:
*  The goal is to drive an automotive grade 12 volt Solenoid which is wired to the car in a 
   "High-Side Switch" configuration.
   What this means:
   *  The module we build will have to be able to deliver high current, and it is an 
      "inductive load".
      Therefore I chose a beefy MOSFET to drive the signal to the transmission.
      
   *  The solenoid is permanently connected to "ground", and we have to close a switch to 12V to
      activate the solenoid and unlock the reverse gear. This means we have a 
      P-Channel MOSFET in "High Side Switching" configuration.
      For an example see here: ![72e085290102973445c4eddf933898884fec76e3](https://user-images.githubusercontent.com/210847/164800283-fc96e098-bd20-4f0d-8c8a-52a04c4c3b73.jpg)

*  The logic implemented by this device is to:
  *  Whenever the vehicle is turned on: Activate the solenoid for 30 seconds 
  *  When the driver pushes a trigger buttom: Activate the solenoid for 30 seconds 

This is a fairly simple timer circuit.

# Hardware Choices

After a bit of research, I decided the 8-pin PIC 12F1572 is more than capable.
It has 6 GPIOs, and holds 2KB memory.
See the datasheet here: https://www.microchip.com/en-us/product/PIC12F1572

For the MOSFET I decided to use the: https://www.sparkfun.com/datasheets/Components/General/FQP27P06.pdf
It can easily drive 20+ Amp, and most important, it has an integrated "fly back diode" to handle switching the inductive load.

Here's an early sketch of a circuit: 

<img src="https://github.com/cnkurzke/HelloPIC_Interrupts/blob/master/docs/PIC_Timer-notebook.jpg" alt="Drawn circuit diagram" width="400px">

It became apparent rather quickly that the biggest challenge would be, how to drive the 12V MOSFET from the 5V PIC output.




# HelloPIC_Interrupts
Sample code for the PIC 12F1572 interrupt handling.

For a very basic introduction about Interrupts on PIC see:
https://picguides.com/beginner/interrupts.php

The Interrupt handler gets registered in main.c, with     
`IOCAF2_SetInterruptHandler(IOCAF2_CGK_InterruptHandler);`

The ISR (`IOCAF2_CGK_InterruptHandler()`) is called everytime the RA2 is pulled to LOW.

More in detail information about De-Bouncing Switches: 
http://www.ganssle.com/debouncing-pt2.htm

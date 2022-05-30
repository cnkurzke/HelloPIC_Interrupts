# About

<img src="https://github.com/cnkurzke/HelloPIC_Interrupts/blob/master/docs/PIC_Timer-render.jpg" alt="3D render PCB" width="200px" align="right">

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
      <img src="https://github.com/cnkurzke/HelloPIC_Interrupts/blob/master/docs/hi-lo_sideSw.png" alt="Low-side vs. high-side" width="200px" align="right">

*  The goal is to drive an automotive grade 12 volt Solenoid which is wired to the car in a 
   "High-Side Switch" configuration.
   What this means:
   *  The module we build will have to be able to deliver high current, and it is an 
      "inductive load".
      Therefore I chose a beefy MOSFET to drive the signal to the transmission.
      
   *  The solenoid is permanently connected to "ground", and we have to close a switch to 12V to
      activate the solenoid and unlock the reverse gear. This means we have a 
      P-Channel MOSFET in "High Side Switching" configuration.
      For an example see here: https://www.elektormagazine.com/news/high-side-low-side-switching

    * <img src="https://github.com/cnkurzke/HelloPIC_Interrupts/blob/master/docs/PIC12_OpenCollector.png" alt="Open Collector with Clamping Diode" width="200px" align="right">
      The P-Channel MOSFET needs to be driven by a 12V Gate voltage, annd can unfortunately not
      be driven by the PIC output pin. Even though the Datasheet mentions that PIC outputs are
      able to work as "open Collector" - this is only thue for voltages within the operating
      voltage level. The circeled Diode in the diagram shows that the PIC has internal 
      clamping Diodes for protection.
      
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

## MOSFET Drriver

It turns out that the PIC output pins, even though they are "Open Collector" capable, they are protected by a diode, which prevents to pull-up to 12V.

I looked at various MOSFET Drivers, but they are really not needed, for the low (basically DC) switching  freqencies.
In this case, a simple NPN BJT in Open Collector mode is suficcient.

## Prototype

After a bit of breadboarding, I had the first version up and running rather quickly. 

<img src="https://github.com/cnkurzke/HelloPIC_Interrupts/blob/master/docs/PIC_Timer-breadboard.jpg" alt="Breadboard with components" width="700px">


# KiCAD Schematics

I decided to create "REAL" circuit boards. After a bit of research, the free software "KiCAD" cauught my interest. It's everything  I need, and way, way more!
See: https://www.kicad.org/

After some trial and error and watching tutorials like: https://www.youtube.com/watch?v=ia2n7P3Csac

I created this schematic:
<img src="https://github.com/cnkurzke/HelloPIC_Interrupts/blob/master/docs/PIC_Timer-schematic.png" alt="Drawn circuit diagram" width="700px">

The source for this is here: https://github.com/cnkurzke/HelloPIC_Interrupts/tree/master/schematics


# PCB Layout

With the help of KiCAD, i turned this schematic into a circuit board, and Gerber files: 
https://github.com/cnkurzke/HelloPIC_Interrupts/tree/master/schematics/KiCAD_project/PIC_Timer/gerbers

The Gerber files describe the PCB and are used to manufacture the final boards:
<img src="https://github.com/cnkurzke/HelloPIC_Interrupts/blob/master/docs/PIC_Timer-PCB.png" alt="Printed Circuit Board" width="400px">

... stay tuned, the boards will arrive in 5 days

To be continued


# SOFTWARE 

## HelloPIC_Interrupts
Sample code for the PIC 12F1572 interrupt handling.

For a very basic introduction about Interrupts on PIC see:
https://picguides.com/beginner/interrupts.php

The Interrupt handler gets registered in main.c, with     
`IOCAF2_SetInterruptHandler(IOCAF2_CGK_InterruptHandler);`

The ISR (`IOCAF2_CGK_InterruptHandler()`) is called everytime the RA2 is pulled to LOW.

More in detail information about De-Bouncing Switches: 
http://www.ganssle.com/debouncing-pt2.htm

---
title: "General Information"
date: 2020-11-05T17:20:49+01:00
draft: false
---

Welcome at the PIC Guide. Here I will post some guides regarding the PIC-microproccessor. They will contain code examples together with explanation of the workings of the microcontroller. In this guide the [PIC16F1829](https://www.microchip.com/wwwproducts/en/PIC16F1829) will be used. This is a 8-bit microprocessor with 1kb of RAM. 

The guides are always structured in the same way. The only difference is if the guide features C and/or PIC-AS. The guides are structerd as follows:

* General info about the guide
* A schema to test the code on
* A list of registers used with their use case
* General info about the program
  * C program
    * Extra info about the C program
  * PIC-AS program
    * Extra info about the workings of the microcontroller
    * Extra info about the PIC-AS program

## Requirements

* Little knowledge of the C programming language
* [Microchip MPLAB IDE](https://www.microchip.com/mplab/mplab-x-ide)
* [XC8 compiler](https://www.microchip.com/en-us/development-tools-tools-and-software/mplab-xc-compilers) and PIC-AS Assembler (PIC-AS Assembler comes with the XC8 compiler)
* Pickit [3](https://www.microchip.com/Developmenttools/ProductDetails/PG164130), [4](https://www.microchip.com/developmenttools/ProductDetails/PG164140) or similar PIC-programmer
* a [PIC16F1829](https://www.microchip.com/wwwproducts/en/PIC16F1829) (Other PIC-microcontrollers are also useable but There might be some minor differences check the datasheet of your PIC for the differences.)
* Some electrical components (Each guide will have a schema but you have to select your own components)

## References in the guides
The guide will have some references to some pdfs

* [Microchip PIC16F1829 Datasheet](http://ww1.microchip.com/downloads/en/DeviceDoc/40001440E.pdf)
* [Microchip MPLAB® XC8 C Compiler User's Guide](http://ww1.microchip.com/downloads/en/devicedoc/50002053g.pdf)
* [Microchip MPLAB® XC8 PIC® Assembler User's Guide](https://ww1.microchip.com/downloads/en/DeviceDoc/MPLAB%20XC8%20PIC%20Assembler%20User's%20Guide%2050002974A.pdf)
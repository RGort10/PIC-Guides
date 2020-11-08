---
title: "Guide 3: Analog Reading"
date: 2020-11-07T20:02:16+01:00
draft: true
---

In this guide we will make a simple program that reads a push button and displays it's state on a LED.

# Schema
The schema is still relativly easy. All you will need is 8 LEDs, 8 resistors and of course the PIC16F1829. If your LEDs exceed the maximum current draw of the PIC16F1829, 25mA per IO-pin and 80mA total, then make sure you add 8 transistors to reduce the current drawn from the microprocessor. All the electrical specifications are specified in section 30.0 of the [datasheet].

# Registers
This is a table with all the registers used in this guide
| Register | Use case                                |
| -------- | --------------------------------------- |
| TRISC    | Used for setting the pinmode of pin RC0 |
| LATC     | Used for writing to PORTC               |
| OSCCON   | Used for setting the clock speed        |





[datasheet]: http://ww1.microchip.com/downloads/en/DeviceDoc/40001440E.pdf
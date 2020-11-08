---
title: "Guide 1: Blink"
date: 2020-11-05T17:34:50+01:00
draft: false
---

In this guide I will explain how to let a LED blink.

# Schema
The schema for the first guide is pretty simple. It only consists of 1 LED and 1 resistor. The resistor is used to limit the current to the LED so it won't blow up. The PIC16F1829 is powered with a voltage of 5V.

![schema]


# Registers
This is a table with all the registers used in this guide
| Register | Use case                                |
| -------- | --------------------------------------- |
| OSCCON   | Used for setting the clock speed        |
| TRISC    | Used for setting the pinmode of pin RC0 |
| LATC     | Used for writing to PORTC               |


# Code
The code of a blinking led is pretty straight forward. It consists of some initialization and a endless loop.

## Initialization of the LED
The LED needs to be initializated. For this, it is important to set the pinmode. The pinmode of a IO-pin is either a input or a output. In case of a LED the pinmode output is used. According to register 12-16, TRISC, the value `1` is input and `0` is output.

## Endless loop
The endless loop is comparable with the `void loop()` of the [Arduino](https://arduino.cc). In the endless loop the LEDs are turned on and of with a delay of 500ms. Writes are done to the LATC. This is needed to avoid that the data get corrupt when writing to the same port two instructions one after the other. This is because of how the pic handles bit manipulations: Read-Modify-Write. Because of this it is advised to always use the LAT register when writing to a port instead of writing it directly.

## C
### Delay macro
The `__delay_ms()` is a macro of the compiler to delay for a certain amount of milliseconds. For this `_XTAL_FREQ` needs to be defined to let the macro know how fast the clock is. This is because the delay function is a simple piece of code that loops till the time is over. So if the clock is two time as fast. The delay will be two times as fast.

```c
/*
 * File:   Guide1-blink.c
 * Author: Renzo Gort
 *
 * Created on Nov 05, 2020
 */
#include <xc.h>
#define _XTAL_FREQ 1000000      // CPU 1MHz // Used for macro functions like '__delay_ms()'

#define HIGH 1                  // Used for readability
#define LOW 0

// Config will be explained in a future guide
#pragma config FOSC=INTOSC
#pragma config WDTE=OFF
#pragma config PWRTE=OFF
#pragma config MCLRE=OFF
#pragma config CP=OFF
#pragma config CPD=OFF
#pragma config BOREN=ON
#pragma config CLKOUTEN=OFF
#pragma config IESO=OFF
#pragma config FCMEN=OFF
#pragma config WRT=OFF
#pragma config PLLEN=OFF
#pragma config STVREN=OFF
#pragma config BORV=HI
#pragma config LVP=OFF

int main(void) {
  OSCCON = 0b01011000;        // Set the CPU speed on 1MHz

  TRISCbits.TRISC0 = 0;       // RC0 is output
  LATCbits.LATC0 = 0;          // Initialize LED off
  
  while(1) {                  // Endless Loop
    LATCbits.LATC0 = HIGH;    // Turn on LED RC0
    __delay_ms(500);          // Wait 500 milliseconds
    LATCbits.LATC0 = LOW;     // Turn off LED RC0
    __delay_ms(500);          // Wait 500 milliseconds
  }
  
  return 0;
}
```

## PIC-AS
### Psect
A Psect is a section in the code. It can contain a few settings: code origin, type of content, addressing format, etc. In this program the `reset_section` is an important piece. This section sets the address of the `main_section` so the microproccessor knows where to find it. In a later guide a further explanation will be provided. If you want to know more now read section 4.9.40 of the [Microchip MPLAB® XC8 PIC® Assembler User's Guide](https://ww1.microchip.com/downloads/en/DeviceDoc/MPLAB%20XC8%20PIC%20Assembler%20User's%20Guide%2050002974A.pdf).

### Registers
The PIC16F1829 memory is divided into 32 banks. Each bank consists of 4 parts. 2 of these parts are availible in all banks and for the other two parts the programmer must explicitly say which bank to use. This can be done with the assembler directive `banksel`. For example if the program wants to use register `LATC` the program will be something like:

```asm
banksel LATC  ; Select the bank that consists LATC
movlw   0xF0  ; Move 0xF0 into the working register
movwf   LATC  ; Write the working register to LATC
```

#### Core registers
1 part of the registers that is available in all banks are the core registers. These registers affect the basic operation of the PIC-microproccessor directly. The core registers are listed below.

* INDF0
* INDF1
* PCL
* STATUS
* FSR0L
* FSR0H
* FSR1L
* FSR1H
* BSR
* WREG
* PCLATH
* INTCON

#### Common RAM
The second part of the registers that are available in all banks is the common RAM. The common RAM can be used by a programmer without having to worry about which bank is selected. This is useful for data that is regulary used throughout the program.

#### Special Function Registers
The special function registers have to be selected before use(with the directive `banksel`). These registers contain for example: `LATC`, `TRISC` and `OSCCON`. For a full list of registers per bank see table 3-3, 3-4, 3-5, 3-6 and 3-7 of the [datasheet].

#### General Purpose RAM
The general purpose RAM consists of up to 80-bytes per bank. According to table 3-3, 3-4, 3-5, 3-6 and 3-7 of the [datasheet] only bank 0 till 11 have the full 80-bytes where bank 12 have 48-bytes and bank 13 till 31 have 0-bytes of general purpose RAM. This adds up to a total of 1008-bytes. Together with the 16-bytes of common RAM this is 1024-bytes of RAM.


### Delay function and variables
In the program two variables are used: `Delay1` and `Delay2`. These variables are use for the inner and outer loop of the delay function. The inner delay consists of exactly 5 instructions. With this number a delay can be calculated with a simple formula: `Delay_time = (Delay1 * 5 / (clock_speed / 4 / 1000)) * Delay2`. In this program a delay of 500 ms is used. Each instructioncycle takes 4 clockcycles. Also in this formula the clockspeed is devided by 1000 for use with milliseconds instead of seconds. Together with a Delay1 of 50 the following calculation can be made:

```
Delay_time = (Delay1 * 5 / (clock_speed / 4 / 1000)) * Delay2
Delay2 = Delay_time / (Delay1 * 5 / (clock_speed / 4 / 1000))
Delay2 = 500 / (50 * 5 / (500000 / 4 / 1000))
Delay2 = 500 / (250 / (125000 / 1000))
Delay2 = 500 / (250 / 125)
Delay2 = 500 / 2
Delay2 = 250
```

### Code
```asm
; Specify the PIC16F1829 as processor for security
processor 16f1829

#include <xc.inc>

; Config will be explained in a future guide
config FOSC=INTOSC
config WDTE=OFF
config PWRTE=OFF
config MCLRE=OFF
config CP=OFF
config CPD=OFF
config BOREN=ON
config CLKOUTEN=OFF
config IESO=OFF
config FCMEN=OFF
config WRT=OFF
config PLLEN=OFF
config STVREN=OFF
config BORV=HI
config LVP=OFF


; Variables with addresses in the common register space
Delay1 equ 0x70
Delay2 equ 0x71

psect reset_section, abs, class=CODE, delta=2
reset_vector:
  pagesel start
  goto start


psect main_section, class=CODE, delta=2
start:     
  movlw 00111000B	  ; Set CPU clock speed on 500kHz
  movwf OSCCON

  banksel TRISC     ; select the bank which contains TRISC     
  bcf     TRISC, 0  ; make IO Pin RC0 an output         
    
  banksel LATC      ; select the bank which contains LATC
  bcf     LATC, 0   ; Turn off Pin RC0


loop:    
  banksel LATC      ; select the bank which contains LATC
  bcf     LATC      ; Turn off LED RC0
    
  ; Wait 500 milliseconds
  movlw   250       ; Move a value of 250 into the working register
  callw    delayLoop ; Call the delay function: 250*2ms= 500ms delay


  bsf     LATC, 0   ; Turn on LED RC0
    
  ; Wait 500 milliseconds
  movlw   250       ; Move a value of 250 into the working register
  callw    delayLoop ; Call the delay function: 250*2ms= 500ms delay
    
  ; Go back to the beginning of the loop for an endless loop
  goto    loop
    

    
;==================================================;
; Delay loop: 
;     Delay2: <value> * 2ms = delay time
;==================================================;
delayLoop:
  movwf   Delay2

delayLoopOuter:
  movlw   50            ; 50 * 5 instructions = 250 instr. 
  movwf   Delay1        ; 250 / 125000 instructions/s = 2ms.
                  
delayLoopInner:
  nop                   ; A nop takes 1 cycle to complete
  nop                   ; Where a bra instruction takes two cycles
  decfsz  Delay1, f     ; Because of this each innerloop takes 
  bra     delayLoopInner; 1 + 1 + 1 + 2 = 5 cycles 
                        ; Together with 50 loops that is 
                        ; 50 * 5 instructions = 250 instructions

  decfsz  Delay2, f     ; Decrement the delay2 variable to zero
  bra	    delayLoopOuter; If Delay2 is not equal to zero
                        ; Do the loop again
  
  return                ; The loops have finished so delay is done
    
    
    
  end     reset_vector  ; Tell assembler to stop assembling
```

[schema]: /schemas/guide1-blink.svg "Guide 1: Blink | Schema"
[datasheet]: http://ww1.microchip.com/downloads/en/DeviceDoc/40001440E.pdf
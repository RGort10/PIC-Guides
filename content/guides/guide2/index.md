---
title: "Guide 2: Digital Reading"
date: 2020-11-07T14:23:39+01:00
draft: false
---

In this guide we will make a simple program that reads a push button and displays it's state on a LED.

# Schema
The schema for this guide consists of 5 parts: one PIC16F1829, one LED, two resistors and one push button. The push button is configured as a [pull-up](https://en.wikipedia.org/wiki/Pull-up_resistor). This means that it's normal state is HIGH.

![schema]


# Registers
This is a table with all the registers used in this guide
| Register | Use case                                 |
| -------- | ---------------------------------------- |
| OSCCON   | Used for setting the clock speed         |
| TRISC    | Used for setting the pinmode of pin RC0  |
| LATC     | Used for writing to PORTC                |
| TRISA    | Used for setting the pinmode of pin RA2  |
| PORTA    | Used for reading from PORTA              |
| ANSELA   | Used for setting digital mode of pin RA2 |


# Code
The code for guide 2 is straight forward. It consists of 2 parts: Initialization of the LED and button, and a loop in where the state of the button is getting displayed.

## Initialization of the LED and button
The initialization of the LED is the same as in [guide 1](/guides/guide1). In that guide is explained that you have to set a pinmode and set the prefferd values in the latches. This guide we also have to initialize a button. A button is an input. With an input it is not necessary to set the prefferd value but it is important to select a digital or analog function. If the analog function is select then a digital read on PORTx will result is a value of `0` even if the actual value is `1`. It's important to note that according to section 12.2.1, 12.3.1 and 12.4.1 of the [datasheet] the default function is analog.

## Endless Loop
In the endless loop the state of the button will be read with the PORTA register. The difference between PORTA and LATC when reading is that PORTA reads the IO-pin while a read of LATC will result in the value of the port latches. A figure of where different reads and writes takes place can be find in figure 12-1 of the [datasheet].

## C
If you want the LED to turn on when the button is pressed. Replace `LATCbits.LATC0 = PORTAbits.RA2;` with `LATCbits.LATC0 = PORTAbits.RA2 ^ 1;` OR `LATCbits.LATC0 = !PORTAbits.RA2;`
```c
/*
 * File:   Guide2-digitalReading.c
 * Author: Renzo Gort
 *
 * Created on Nov 07, 2020
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

  TRISAbits.TRISA2 = 1;       // RA2 is input
  ANSELAbits.ANSA2 = 0;       // RA2 is digital  

  while(1) {                  // Endless Loop
    // Display the state of the button on the LED
    LATCbits.LATC0 = PORTAbits.RA2;

    __delay_ms(10);           // We don't need to update the LED
                              // 250000 times per second
                              // 100 times is enough
  }
  
  return 0;
}
```

## PIC-AS
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
  bcf     LATC, 0   ; turn off Pin RC0 

  banksel TRISA     ; select the bank which contains TRISC     
  bsf     TRISA, 2  ; make IO Pin RA4 an input         

  banksel ANSELA    ; select the bank which contains LATC
  bcf     ANSELA, 0 ; make IO Pin RA4 digital 


loop:    
  banksel PORTA 
  movf	  PORTA, w  ; Load value of button in working register
  
  banksel LATC
  btfsc   w, 2
  bsf     LATC, 0   ; If button is pressed turn on LED

  btfss   w, 2
  bcf     LATC, 0   ; If button is not pressed turn off LED

  ; Wait 10 milliseconds
  movlw   5         ; Move a value of 5 into the working register
  callw   delayLoop ; Call the delay function: 5*2ms= 10ms delay
    
  ; Go back to the beginning of the loop for an endless loop
  goto    loop
    

    
;==================================================;
; Delay loop:                                      ;
;     W Register: <value> * 2ms = delay time       ;
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

[schema]: /schemas/guide2-digitalReading.svg "Guide 2: Digital Reading | Schema"
[datasheet]: http://ww1.microchip.com/downloads/en/DeviceDoc/40001440E.pdf
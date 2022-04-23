# Alarm Clock using ATmega32 Microcontroller

In this project we are going to design a simple Alarm clock using ATMEGA32 timers. ATmega32A microcontroller has a 16 bit timer, and we will be using that timer to count the seconds and develop a digital clock.

All the digital clocks have a crystal inside of them which is the heart of clock. This crystal not only present in clock but present in all computing real time systems. This crystal generates clock pulses, which is needed for timing calculations. Although there are some other ways to get clock pulses but for accuracy and higher frequency most prefer crystal based clock. We are going to connect a crystal to ATMEGA32 for getting accurate clock.

![Image1](https://github.com/Krishuk/M2_7_Segment_Display/blob/main/2_Design/AVR-Alarm-Clock.jpg)

### Circuit Diagram and Working Explanation:

![Image2](https://github.com/Krishuk/M2_7_Segment_Display/blob/main/2_Design/AVR-Alarm-Clock-Circuit.gif)

For accurate timing, we have connected a 11.0592MHz crystal for clock. Now for disabling the internal clock of ATMEGA we have to change its LOW FUSE BITS. Remember we are not touching the high fuse bits so the JTAG communication would be still enabled.

For telling ATMEGA to disable internal clock and to work on external we need to set:                             

LOW USE BYTE = 0xFF or 0b11111111.

In circuit PORTB of ATMEGA32 is connected to data port LCD. Here one should remember to disable the JTAG communication in PORTC of ATMEGA by changing the high fuse bytes, if one wants to use the PORTC as a normal communication port. In 16x2 LCD there are 16 pins over all if there is a black light, if there is no back light there will be 14 pins. One can power or leave the back light pins. Now in the 14 pins there are 8 data pins (7-14 or D0-D7), 2 power supply pins (1&2 or VSS&VDD or gnd&+5v), 3rd pin for contrast control (VEE-controls how thick the characters should be shown), and 3 control pins (RS&RW&E)

In the circuit, you can observe that I have only took two control pins. This gives the flexibility of better understanding, the contrast bit and READ/WRITE are not often used so they can be shorted to ground. This puts LCD in highest contrast and read mode. We just need to control ENABLE and RS pins to send characters and data accordingly.

The connections which are done for LCD are given below:

PIN1 or VSS to ground

PIN2 or VDD or VCC to +5v power

PIN3 or VEE to ground (gives maximum contrast best for a beginner)

PIN4 or RS (Register Selection) to PD6 of uC

PIN5 or RW (Read/Write) to ground (puts LCD in read mode eases the communication for user)

PIN6 or E (Enable) to PD5 of uC

PIN7 or D0 to PB0 of uC

PIN8 or D1 to PB1 of uC

PIN9 or D2 to PB2 of uC

PIN10 or D3 to PB3 of uC

PIN11 or D4 to PB4 of uC

PIN12 or D5 to PB5 of uC

PIN13 or D6 to PB6 of uC

PIN14 or D7 to PB7 of uC

In the circuit you can see we have used 8bit communication (D0-D7) however this is not a compulsory, we can use 4bit communication (D4-D7) but with 4 bit communication program becomes a bit complex. So as shown in the above table we are connecting 10 pins of LCD to controller in which 8 pins are data pins and 2 pins for control.

Switch one is for enabling adjust feature between alarm and time. If the pin is low, we can adjust alarm time by pressing buttons. If its high buttons are for adjusting just TIME. There are FOUR buttons present here, first is for increment MINUTES in alarm or time. Second is for decrement MINUTES in alarm or time. Third is for increment HOUR in alarm or time. FOURTH is for decrement HOURS in alarm or time.

The capacitors present here is for nullifying the bouncing effect of buttons. If they are removed the controller might count more than one each time the button is pressed. The resistors connected for pins are for limiting the current, when the button is pressed to pull down the pin to the ground.
Whenever a button is pressed, The corresponding pin of controller gets pulled down to ground and thus the controller recognizes that certain button is pressed and corresponding action is taken.
First of all, the clock we choose here is 11059200 Hz, dividing it by 1024 gives 10800. So for every second we get 10800 pulses. So we are going to start a counter with 1024 prescaler to get the counter clock as 10800 Hz. Second we are going to use the CTC (Clear Timer Counter) mode of ATMEGA. There will be a 16 bit register where we can store a value (compare value), when the counter counts up to the compare value an interrupt is set to generate.
We are going to set the compare value to 10800, so basically we will have a ISR (Interrupt Service Routine on every comparison) for every second. So we are going to use this timely routine to get the clock we needed.

![Image3](https://github.com/Krishuk/M2_7_Segment_Display/blob/main/2_Design/timer-counter.gif)

BROWN(WGM10-WGM13): These bits are for selecting mode of operation for timer.

![Image3](https://github.com/Krishuk/M2_7_Segment_Display/blob/main/2_Design/waveform-generation_0.gif)

Now since we want the CTC mode with compare value in OCR1A byte, we just have to set WGM12 to one, remaining are left as they are zero by default.
RED (CS10,CS11,CS12): These three bits are for choosing the prescalar and so getting appropriate counter clock.

![Image4](https://github.com/Krishuk/M2_7_Segment_Display/blob/main/2_Design/Clock-Select-Bit.gif)

Since we want a 1024 as prescaling, we have to set both CS12 and CS10.
Now there is an another register which we should consider:

![Image5](https://github.com/Krishuk/M2_7_Segment_Display/blob/main/2_Design/timer-counter-interrupt.gif)

GREEN (OCIE1A): This bit must be set for getting an interrupt on compare match between counter value and OCR1A value(10800) which we set.

![Image5](https://github.com/Krishuk/M2_7_Segment_Display/blob/main/2_Design/output-compare.gif)

OCR1A value (counter compare value), is written in above register.



mclock
======

Multimode Matrix Clock

Changes Feb 08, 2016
____________________
This is RETIRED source. I had combined my breadboard projects into my "breadboard_collections". The latest source resides there. Please use it for the latest fixes.

Changes Dec 03, 2015
____________________
Add mclock.hex ready to flash firmware.
change so that source now compiles on both gcc and CCS.

Changes Dec 01, 2015
____________________
Clock builder Terry M. took the time to build this project and found out that the sharing of the xin xout pins between led driving and time keeping (via 32khz crystal) is a bad idea. I.e. the clock xtal could not oscillate.

I tried it and cannot get the problem resolve, there is always a osc fault interrupt and mclock cannot keep time at all! Now I got it to work initially I can no longer reproduce. This change is to resolve this problem by sacrifying C7 (column 7) of the LED matrix, so that xin is isolated and used for xtal oscillation only. This indeed solve the time keeping issue at the expense of 8 less dots of display (now 8x7 dots). I also re-arrange various display modes to reflect this change.

* When you build this, be sure to "isolate" LED matrix C7 (see breadboard layout) so that it is NOT connected. You can either cut away / bend C7 pin on LED matrix, or insulate it with breadboard wire coat / skin (cat5 wire works perfectly)

There is a "#define NO_C7" directive at the top of the source code, commenting it out will allow for build of previous 8x8 version.

Thanks Terry M. to point out the bug in the previous build.


Description
___________

This is a multi-mode clock project based on the msp430g2452. It can be assembled with minimal parts. With limited 8x8 pixels display resolution, this 12 hour clock shows time in 6 different modes. This project is based on a older attiny 2313 project I did a few years ago.

HHMM mode, typical hours plus minutes scrolling digits with colon separator.

Seconds mode, shows only seconds.

Tix mode (shown below), led matrix is divided into quadrant, the upper quadrants shows the hour in bcd (binary coded decimal) values. they are represented by the number of dots to indicate the digits. the lower quadrants show the minute in bcd. i.e. for 12:37 it shows no dot + 9 dots on the upper half and 3 dots + 7 dots on the lower half.

Dice mode (shown below), the led matrix is divided into two set of 'dices'. with the upper pair showing hour from 1 - 12, the lower pair of dice shows minutes in 5 minute increments. i.e. for 9:45 it shows dice value upper 9 + lower 9 (9 hour, 9 x 5 min).

Binary (really it's bcd, or binary coded decimal) mode, (shown below) the hour, minute and second digits are show as binary dot on different columns in the led matrix. the columns 0 and 1 (from left) represents the hour digits, column 2 is blanked, columns 3 and 4 represent the minute digits, colum 5 is blanked, columns 6 and 7 represents the second digits.


The circuit employs row and column multiplexing to drive the leds, one row at a time, this gives a 12.5% duty cycle when "sets" of leds (8 of them in each of the 8 rows) are turn on briefly. current limiting resistors are eliminated to save breadboard estate and as we are not constantly driving individual leds, they are not going to be damaged.

The control (user interface) is also arranged so that we only use one tactile button for input. the firmware capture long button presses (press and hold) for menu rotation and normal button presses for menu selection.

By migrating this project from an AVR mcu to a msp430 mcu I had made it possible to keep time a lot more accurately. During display (i.e. led on) the project runs at 1Mhz DCO. The msp430 mcu has factory calibrated clock values. When not displaying, this project enters a LPM3 (low-power mode 3) to conserve power. At LPM3 the DCO clock cannot be used and the project switches to use a 32Khz crystal based AClk to keep time.

Features
________

* Minimal component count, 4 parts.
* Battery operated from 3V to 3.6V.
* Use of watchdog timer to keep time, power-down sleep mode (LPM3) takes uA power.
* 32Khz crystal to keep accurate time when sleep.
* Runs 1Mhz DCO calibrated clock when active (displaying time).
* This is a 12H clock, not 24H and has no AM/PM indicator.
* Easter egg tetris-like game.


Parts list
__________

* msp430g2452 (or other G series dip 20pin devices w/ 4k+ flash)
* 8x8 LED matrix display (red only, this is a 3V project)
* tactile button
* 32Khz clock crystal
* 2x LR44 button cell or 3V-3.6V other battery source

Application Notes
_________________

* Short key press in display mode cycles through HHMM, seconds, tix, dice, binary and sleep modes.
* Long press enters setup mode, subsequent long press rotates thru menu.
* Menu items cycles thru 'Set Clock', 'Dimmer', 'Auto-off'.
* In 'Set Clock' setup mode, short presses increment digit values (hours, minutes) and long press confirms.
* In 'Dimmer' setup mode, short presses cycles through available brightness levels, long press confirms setting.
* In 'Auto-off' setup mode, short presses toggle the auto-off on and off. With auto-off turned on, the clock displays time for 15 seconds and turn itself into LPM3 sleep mode to converse power. With auto-off turned off, display is on continously.
* When in sleep mode, MCU goes in power down mode, consuming less than 30uA of power, 32Khz crystal w/ watchdog timer is used to keep time. A pin interrupt is enabled to allow for wake up via tactile button. In this mode the main clock is disabled to conserve power.
* Led segment multiplexing includes time delays to compensate for brightness differences for individual rows.


Breadboard Layout
_________________

The 8x8 led matrix has dot size of 1.9mm and is of common cathode, if you have common anode type, you can change a few lines in the code for adoption. see the following diagram and see if you have the right pin-outs. It appears they are quite common and if you purchase via ebay most suppliers have the same pin-out even if the model number is different.

                       
           +=====================================================+
           |  .  .  +(w2)----------------o--(w3)----------+.  .  |
           |  .  . ==== ===  .  .  .  .  .  o--(w4)-----o |.  .  |
           |  .  .|    |   | .  +--+--+--+--+--+--+--+  . |.  .  |
           |  .  o|     |   |o C7 C6 R1 C0 R3 C5 C3 R0  . |.  .  |
           |  .  ||    |   | .  |    b2 b3           |  . |.  .  |
           |  .  | ==== ===  +--+--+--+--+--+--+--+--+--+ |.  .  |
           |     |  |       |G b6 b7  T  R a7 a6 b5 b4 b3||      |
           |     |  |       |                            ||      |
           |     |  |       |+ a0 a1 a2 a3 a4 a5 b0 b1 b2||      |
           |  .  |  o  .  . .+--+--+--+--+--+--+--+--+--+ |.  .  |
           |  .  +(w1)-------o  +--+--+--+--+--+--+--+  o-+.  .  |
           |  .  .  .  .  .  .  R4 R6 C1 C2 R7 C4 R5 R2 .  .  .  |
           |  .  .  .  .  .  .  x  x  o  o  .  .  .  .  .  .  .  | x-x and o-o connect to game buttons
           |  .  .  o-Button-o  .  .  .  .  .  .  .  .  .  .  .  |
           |  .  .  .  .  .  .  .  .  .  .  .  .  .  .  .  .  .  |
           +=====================================================+
                               

Schematic
_________

             MSP430G2452
             -----------------
            |                 |        /|\
            |                 |  _==_   | 
            |              RST|--o  o---+
            |                 |  |
            |                 |  |  +---------------+ 
            |    column 0 P2.3|--+--|/              |
            |    column 1 P1.2|-----|/              |
            |    column 2 P1.3|-----|/              |
            |    column 3 P2.5|-----|/              |
            |    column 4 P1.5|-----|/              |
            |    column 5 P2.6|-----|/              |
            |    column 6 P2.7|-----|/              |
            |    column 7 P2.6|-----|/              |
            |                 |     +---------------+
            |                 |      | | | | | | | |        32Khz Xtal
            |       row 0 P2.4|------+ | | | | | | |   P2.6 --o[ ]o-- P2.7
            |       row 1 P2.2|--------+ | | | | | |
            |       row 2 P2.1|----------+ | | | | |   game buttons (left + right)
            |       row 3 P1.7|------------+ | | | |          _==_
            |       row 4 P1.0|--------------+ | | |   P1.0 --o  o-- P1.1
            |       row 5 P2.0|----------------+ | |
            |       row 6 P1.1|------------------+ |          _==_
            |       row 7 P1.4|--------------------+   P1.2 --o  o-- P1.3
            |                 |
             -----------------



Assembling
__________

* Follow breadboard layout and place two jumper wires on mini breadboard
* Place msp430g2452 mcu
* Place 32Khz crystal
* Place Tactile Button
* Place power source (I am using 2xLR44 w/ magnets as holders)
* Finally place 8x8 led matrix on top of msp430g2452

Additional Notes
________________

Although the project now uses a handful of icons for menu selection, I had included 38 characters in the rom. i.e. digits 0-9, letters A-Z, a '.' and a space character. It may be useful to create other projects.

The tic mode and dice mode pattern selection is not true random as we had code size restriction.

Per row leds brightness are compensated in software by adjusting how long a row of leds stays on and off. i.e. for rows with all 8 column leds on, we stay longer to make them appears to be as bright as those rows that have only one or two leds on.

Watchdog timer is used as this allows for sleep mode to prolong battery life. This allows us to go sleep at LPM3 for maximum power savings while retain accurancy via the 32Khz clock crystal.

If you have any questions, visit 43oh.com and find me in the forums.






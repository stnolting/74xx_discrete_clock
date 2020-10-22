# 74xx Discrete Clock

[![license](https://img.shields.io/github/license/stnolting/74xx_discrete_clock)](https://github.com/stnolting/74xx_discrete_clock/blob/master/LICENSE)


## Table of Content

* [Introduction](#Introduction)
* [General Architecture](#General-Architecture)
* [Counter Module](#Counter-Module)
* [Mother and Display Board](#Mother-and-Display-Board)
* [Clock Module](#Clock-Module)
* [Assembled Clock](#Assembled-Clock)
* [Accuracy](#Accuracy)
* [Copyright](#Copyright)



## Introduction

I have always been a fan of digital logic. I had a bunch of different 74xx TLL/CMOS ICs collecting dust in my shelves and I
wanted to do something cool with them. A simple 4-bit CPU would be awesome, but a retro-style clock seems to be cool, too.

See the project log on [hackaday.io](https://hackaday.io/project/171731-74xx-discrete-clock).


Design principles:
* clock with hours, minutes and seconds display, 24h format
* 7-segment display to have the nice 70s/80s charm
* everything is driven by a single 1Hz clock
* everything runs on 5V
* use discrete-logic only - no "hidden" AVR or something like that
* use the stuff I have in my shelves
* solder-friendly parts only
* use drilled boards instead of messy breadboards - no PCBs
* simple architecture
* suitable as project for a rainy (or Corona-lockdowned) weekend
* modular setup, so I can replace things if I mess something up

![front view](https://github.com/stnolting/74xx_discrete_clock/blob/master/img/final_front.jpg)



## General Architecture

The clock uses 4-bit binary counter ICs (74HC161) for counting hours, minutes and seconds, which are interconnected in a classic ripple-carry scheme.
The output of the counters is decoded to drive classy 7-segment displays. There are 6 counter modules, which are plugged into a
motherboard. This motherboard provides the power supply and interconnects the counter modules with each other and also with
the display board. The display board contains the decoders and the actual 7-segment displays. All modules are clocked by clock module, which
generates the 1Hz base clock.

Hardware overview:
* 6 "programmable" counter modules
* 1 clock module to generate the 1Hz clock
* 1 display board with 6 7-segment displays
* 1 mother board to connect everything



## Counter Module

The counter modules are the actual heart of the clock. There are six counter modules in total - one for each digit. 
All these counter modules are identical (I will come back to this "issue"...). They are based on 74HC161 binary counter. The counting range is defined
via a 4-bit DIP switch, so each module can be configured to drive any of the displays. After reset or after an overflow, the counter starts counting
at zero an counts until it reaches the "programmed" value again. I am using a 4-bit comparator based on 4 XOR2 gates (74HC86) and a final NOR4 gate (74HC4002).
The output of this comparator is HIGH when the current counter state is equal to the value programmed via the DIP switches. The comparator output is connected
to one input of a cheap AND2 gate (based on two diodes and a resistor). The other input of the cheap AND2 is driven by the module's carry input (CI).
When the comparator detects a match AND the module's carry input is HIGH, the cheap AND2 outputs a HIGH level. This output is inverted by the second NOR4
gate and brought to the low-active parallel load input of the counter chip. Since this is a synchronous control signal, the counter is reset in the next clock
cycle (all parallel input signals of the counter are tied to low). The signal driving the parallel load also represents the module's carry output, which
is low-active (!CO). The status of the carry output is indicated by a red LED.

The enable input of the counter chip is driven by a cheap OR2 (two diodes and a resistor). The first input is driven by the module's carry input (CI).
The second input is driven by an on-board button, so the counter can be manually set (to define the initial time). There is no need for any kind of
key debouncing, since the counter enable is only evaluated at a rising clock edge (which runs at 1Hz).

Clock and reset signals of the counter chip are driven by the global reset (!RST) and clock (CLK) signals.

Schematic: [counter_module.pdf](https://github.com/stnolting/74xx_discrete_clock/blob/master/schematic/counter_module.pdf) [counter_module.sch](https://github.com/stnolting/74xx_discrete_clock/blob/master/schematic/counter_module.sch) [counter_module.png](https://github.com/stnolting/74xx_discrete_clock/blob/master/schematic/counter_module.png)

Used logic chips:
* [74HC161](https://github.com/stnolting/74xx_discrete_clock/blob/master/datasheets/74HC161.pdf): 4-bit counter with synchronous parallel load
* [74HC4002](https://github.com/stnolting/74xx_discrete_clock/blob/master/datasheets/74HC_HCT4002_Q100.pdf): Dual NOR4 gate
* [74HC86](https://github.com/stnolting/74xx_discrete_clock/blob/master/datasheets/74HC_HCT86.pdf): Quad XOR2 gate

### Pictures

![counter module](https://github.com/stnolting/74xx_discrete_clock/blob/master/img/counter_module.jpg)

![after ages... all six counter_modules](https://github.com/stnolting/74xx_discrete_clock/blob/master/img/counter_modules.jpg)



## Mother and Display Board

The clock module and the counter modules are plugged into the mother board. This board provides the power supply (based on a simple 7805) and
takes care of the interconnection of all modules. The carry output of one counter module is connected to the carry input of the next one. Since the
carry outs are low-active and the carry ins are high-active, an inverter (74HC04) is required between each stage.

The outputs from the counter modules are brought to the display board. Each output pin as a pull-down resistor to prevent the display decoders from going
crazy when a module is unplugged. Also, the carry out signals, the global clock and the global reset are terminated using pull-up/pull-down resistors, respectively

As the name already suggests, the display board carries the six 7-segment displays and the BCD-to-7-segment decoders (74HC4511). The latches of the decoders
are always transparent. Also, the decoder's lamp test or output enable features are not used. There are two LEDs between the HRS and MIN displays
and between the MIN and SEC display. These LEDs are driven (via a transistor on the mother board) by the main clock signal to give the clock a nice radio clock look.


Schematic (mother board): [mother_board.pdf](https://github.com/stnolting/74xx_discrete_clock/blob/master/schematic/mother_board.pdf) [mother_board.sch](https://github.com/stnolting/74xx_discrete_clock/blob/master/schematic/mother_board.sch) [mother_board.png](https://github.com/stnolting/74xx_discrete_clock/blob/master/schematic/mother_board.png)

Schematic (display board): [display_board.pdf](https://github.com/stnolting/74xx_discrete_clock/blob/master/schematic/display_board.pdf) [display_board.sch](https://github.com/stnolting/74xx_discrete_clock/blob/master/schematic/display_board.sch) [display_board.png](https://github.com/stnolting/74xx_discrete_clock/blob/master/schematic/display_board.png)

Used logic chips (mother board):
* [74HC04](https://github.com/stnolting/74xx_discrete_clock/blob/master/datasheets/74HC_HCT04.pdf): Hex inverter

Used logic chips (display board):
* [74HC4511](https://github.com/stnolting/74xx_discrete_clock/blob/master/datasheets/74HC_HCT4511.pdf): BCD to 7-segment decoder

### Pictures

![mother/display board top](https://github.com/stnolting/74xx_discrete_clock/blob/master/img/mother_board_top.jpg)

![mother/display board bottom](https://github.com/stnolting/74xx_discrete_clock/blob/master/img/mother_board_bottom.jpg)

![mother/display board](https://github.com/stnolting/74xx_discrete_clock/blob/master/img/mother_board.jpg)



## Clock Module

I am using a straight forward circuit for the NE555 here, but with an additional diode to have identical HIGH and LOW times.
These times are defined by the capacitor (2x1µF Wima foil capacitors) and two potentiometers. I wrote a simple program (for the NEO430​)
to measure the HIGH and LOW times so I can adjust the potentiometers to try to come as close as possible to 1Hz. Well, it worked - somehow.
I fixed the potentiometers with some fancy nail polish. Of course the precision is not the best (e.g. thermal influence on the capacitors and resistors).

Schematic: [clock_module.pdf](https://github.com/stnolting/74xx_discrete_clock/blob/master/schematic/clock_module.pdf) [clock_module.sch](https://github.com/stnolting/74xx_discrete_clock/blob/master/schematic/clock_module.sch) [clock_module.png](https://github.com/stnolting/74xx_discrete_clock/blob/master/schematic/clock_module.png)

Used logic chips:
* [NE555](https://github.com/stnolting/74xx_discrete_clock/blob/master/datasheets/ne555.pdf): The one and only keeper of time :sunglasses:

### Pictures

![clock module top](https://github.com/stnolting/74xx_discrete_clock/blob/master/img/clock_module_top.jpg)

![clock module bottom](https://github.com/stnolting/74xx_discrete_clock/blob/master/img/clock_module_bottom.jpg)



## Assembled Clock

When assembled and powered up, the clock draws ~95mA @ 7V (~70mA @ 6V).

### Pictures

![rear view](https://github.com/stnolting/74xx_discrete_clock/blob/master/img/final_rear.jpg)

![top view](https://github.com/stnolting/74xx_discrete_clock/blob/master/img/final_top.jpg)

![bird's view](https://github.com/stnolting/74xx_discrete_clock/blob/master/img/final_sky.jpg)



## Accuracy

I've tried my best to _calibrate_ the NE555 to generate a 1 Hz clock. My oscilloscope was not precise enough for that, so I
used a simple microcontroller setup based on the [NEO430](https://github.com/stnolting/neo430) to compare the NE555 clock
against a Quartz-based reference clock. The deviation in _ms_ is printed via the processor's UART. After some fine tuning
of the potentiometer I finally got a "good" 1 Hz clock from the NE555 and sealed the configuration with some nail polish.

![calibration of the NE5555](https://github.com/stnolting/74xx_discrete_clock/blob/master/img/clock_calibration.jpg)

Well, it's obvious that the NE555 is not the most precise clock generator on the block. The actual frequency is impacted by the voltage fluctuations
and especially by the temperature of the time-defining components. I tested the clock on a warm summer day and after 24 hours I got a deviation of approx.
+1800s or 30 minutes. Not good, not bad, but pretty moderate. So I think I will need to replace the NE555 by a more stable (Quartz) oscillator.



## Copyright

This is a hobby project released under the BSD 3-Clause license. No copyright infringement intended.

Schematics made with EAGLE 9.6.0 free.


Made with :coffee: in Hannover, Germany.

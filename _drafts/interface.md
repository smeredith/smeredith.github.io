---
layout: post
title: Radio Interface
---

I have designed and built a radio interface based on the [Masters Communications DRA-30](https://www.masterscommunications.com/products/radio-adapter/dra/dra30.html).
My interfaces adds
- a serial port for PTT control via RTS and/or CAT/CI-V control of the radio,
- a VOX circuit,
- a GPS,
- an integrated USB hub so only one cable is required between the computer and the interface,
- a USB-C connector in place of the USB type B connector on the DRA-30.

The interface uses the original DB-9 connector on the DRA-30 and adds the serial port TX and RX as TTL-level signals.
It also adds 5V to one of the pins to support converting the TTL signals to real RS232 level signals.

![interface in case](interface-in-case.jpg)
![connected boards](connected-boards.jpg)
![side-by-side boards](side-by-side-boards.jpg)

I have mixed feelings about the DB-9.
On the one hand, it's an ancient and clunky connector that extends a good distance out the back of the interface.
On the other hand, it is easy to wire up whatever cable configuration is needed for any radio, including space for a capacitor and resistor that some HTs require, or a circuit to convert the UART TTL signals to conforming RS232 signals.
Multiple cables eliminate the need for something like the set of jumpers that the SignaLink uses to configure the pins.

![db-9 circuit](db-9-circuit.jpg)

The interface addes a circuit board that sits on top of the DRA-30, making electrical connections to it via header recepticles on the bottom that connect to jumpers on the DRA-30.
Some of those jumpers come with the DRA-30, and some I had to solder on.
The add-on board sits very low on the DRA-30 with the goal of allowing it to fit inside the original DRA-30 case (when the tall stacked USB connector for the GPS is not installed.)
The dimensions of the board are slightly smaller than those of the DRA-30, allowing space for the ridges on the case lid to fit.

## DRA-30 Modifications

I removed the blue "COMM OK" LED and replaced it with a jumper.
My circuit uses this signal to decided if it's safe to trigger PTT.
The blue LED is replicated on the add-on board.

I added 

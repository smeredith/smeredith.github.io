---
layout: post
title: Radio Interface
---

I have designed and built a radio interface based on the [Masters Communications DRA-30](https://www.masterscommunications.com/products/radio-adapter/dra/dra30.html).
My interfaces adds
- a serial port for PTT control via RTS and/or CAT/CI-V control of the radio with TX and RX on the DB-9,
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
Some of those headers come with the DRA-30, and some I had to solder on.
The add-on board sits very low on the DRA-30 with the goal of allowing it to fit inside the original DRA-30 case (when the tall stacked USB connector for the GPS is not installed.)
The dimensions of the board are slightly smaller than those of the DRA-30, allowing space for the ridges on the case lid to fit.

## DRA-30 Modifications

It's easier to make these modifications while you are building the board instead of desoldering things.

I did not install the R-PTT resistors.

I did not install JU1 and JU2 headers because I don't need COS or CTCSS and want these DB-9 pins for the serial port TX and RX instead.

I removed the USB type B connector and replaced it with a 4-pin square header.
This is so the DRA-30 can connect to the USB hub chip on the add-on board instead of to the computer.
The pin spacing is not quite right for a standard header, but I was able to manipulate the pins enough to make it fit.

I removed the blue "COMM OK" LED and replaced it with a 2-pin header.
Like the DRA-30, my circuit use this signal to decided if it's safe to trigger PTT.
The blue LED is replicated on my add-on board.

I added a 2-pin header in holes marked "2" and "4" to get access to pins 2 and 4 on the DB-9.
The holes happen to be the right distance appart to accept a standard 2-pin header.
I could have used the headers labeled "JU1" and "JU2" for this but chose not to because of the convenient spacing of the "2" and "4" holes.

I added a 1-pin jumper to the hole marked "7" to get access to pin 7 on the DB-9.
I put 5V on this pin to power a serial port level converter inside the DB-9 case if needed.



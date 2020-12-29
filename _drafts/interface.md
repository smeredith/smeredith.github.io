---
layout: post
title: Radio Interface
---

I have designed and built a radio interface based on the [Masters Communications DRA-30](https://www.masterscommunications.com/products/radio-adapter/dra/dra30.html).
My interfaces adds:
- a serial port for PTT control via RTS and/or CAT/CI-V control of the radio with TX and RX on the DB-9,
- a VOX circuit with adjustable sensitivity and tail hang time,
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

The add-on board is a circuit board that sits on top of the DRA-30, making electrical connections to it via header recepticles on the bottom that connect to header pins on the DRA-30.
Some of those headers come with the DRA-30, and some I had to solder on.
The add-on board sits very low on the DRA-30 with the goal of allowing it to fit inside the original DRA-30 case (when the tall stacked USB connector for the GPS is not installed.)

## DRA-30 Modifications

It's easier to make these modifications while you are building the board instead of desoldering things.

I did not install the R-PTT resistors.

I did not install JU1 and JU2 headers because I don't need COS or CTCSS and want these DB-9 pins for the serial port TX and RX instead.

I removed the USB type B connector and replaced it with a 4-pin square header.
This is so the DRA-30 can connect to the USB hub chip on the add-on board instead of directly to the computer.
The pin spacing is not quite right for a standard header, but I was able to manipulate the pins enough to make it fit.

I added a 2-pin header in holes marked "2" and "4" to get access to pins 2 and 4 on the DB-9.
The holes happen to be the right distance apart to accept a standard 2-pin header.
I could have used the headers labeled "JU1" and "JU2" for this but chose not to because of the convenient spacing of the "2" and "4" holes.

I added a 1-pin jumper to the hole marked "7" to get access to pin 7 on the DB-9.
I put 5V on this pin to power a serial port level converter inside the DB-9 case if needed.

I snipped about 1mm off the header pins to allow the board to sit lower on the DRA-30 so that it would fit inside the original case.
I should have made the dimensions of the board slightly smaller so that it would fit inside the ridges on the lid of the case.
This would have made snipping the header pins unnecessary, and I will make this change on the next revision of the board.

## Project Motivation

The DRA-30 uses a GPIO on the Cmedia CM119A sound card chip for PTT.
This works fine for Vara FM and SoundModem.
But I wanted to use Fldigi and Direwolf on Windows, but neither supported this method of PTT.
So the motivation for this project was adding a serial port so that the software could assert RTS to trigger PTT.
Most software supports this scheme.

I also wanted VOX as a last resort in case I wanted to use some software that doesn't support either of the methods described above.
As of yet I haven't needed it, but the circuitry is implemented and it works.

## Microcontroller

There is also an Attiny84 microcontroller on the board.
This simplifies the PTT implementation.
The microcontroller listens to the following inputs:
- RTS signal from the serial port chip,
- VOX circuit audio detect level,
- RTS DIP switch position,
- VOX DIP switch position,
- VOX sensitivity trimpot setting,
- VOX tail time trimpot setting.

If the RTS DIP switch is OFF, then the serial port is ignored.
Otherwise, if RTS goes low, PTT is asserted.

If the VOX DIP switch is OFF, the VOX circuitry input is ignored.
Otherwise, if the audio detect signal is above the threshold specified by the VOX sensitivity pot, PTT is asserted.
PTT is not released until a time specified by the tail time trimpot after the audio detect level drops off.

In my next board revision, I will add the COMM OK signal from the DRA-30 555 timer as an input to the microcontroller as a requirement to assert PTT.

## PTT

The microcontroller triggers a MOSFET to ground the PTT line on the DB-9.
The SMT MOSFET can sink 300mA from the radio's PTT line to ground, which is plenty for my setup.
I can also solder in a through-hole BS170 should I need more current, up to 500mA.

A red LED lights when PTT is asserted.

## UART

I chose the FT232R for the serial port chip.
I like this chip because I can configure it via a utility program if I ever need to invert the TX or RX signal.
It also solves a problem with Windows asserting RTS when at inopportune times.

Windows will assert RTS several times when the serial port is detected, or some other USB serial port is added or removed from the system.
If your radio is connected to the interface when this happens, PTT will assert and your radio will transmit a few short pulses.
This is a pain and you have to manage it manually by turning on the radio last or connecting it after Windows has finished its thing.
But the FT chip can be configured to prevent this via the device manager: under the advanced properties settings, check "Disable Modem Ctrl At Startup" and uncheck "Serial Enumerator".
This is a huge convenience.

## USB Hub

The UART is a USB device.
Without a hub, I would need two USB connections to the computer.
So I built the USB hub into the add-on board--it's a chip and a handful of capacitors and resistors.
Now the hub is the only thing connected to the computer, and the serial port, DRA-30, and GPS all use ports on the hub.
There is one USB port left over on the hub for future expansion.

## GPS

Two of the USB hub ports are connected to a stacked double USB connector on the add-on board.
The GPS is an unmodified dongle from u-blox that plugs into that connector.
It appears to the computer as a serial port device.
By default, it spits out NMEA sentences once per second at 9600 baud.
This is what I want and I didn't have to configure a thing on it.
This is useful for APRS, Winlink position reports, and for syncing the computer's time if a network is not available.



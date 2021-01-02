---
layout: post
title: Radio Interface
---

I have designed and built a radio interface add-on board for use with the [Masters Communications DRA-30](https://www.masterscommunications.com/products/radio-adapter/dra/dra30.html).
My add-on board adds:
- a serial port for PTT control via RTS and/or CAT/CI-V control of the radio with TX and RX on the DB-9,
- a VOX circuit with adjustable sensitivity and tail hang time,
- a GPS,
- an integrated USB hub so only one cable is required between the computer and the interface,
- an extra USB port on the integrated hub,
- 5V on one pin of the DB-9.

The add-on board adds serial port TX and RX as TTL-level signals to the original DB-9 connector on the DRA-30.
It also adds 5V to one of the lines to support converting the TTL signals to real RS-232 level signals downstream of the interface.
This configuration is in no way standard and custom cables are required for every radio.

![interface in case](interface-in-case.jpg)
![connected boards](connected-boards.jpg)
![side-by-side boards](side-by-side-boards.jpg)

The add-on board is a circuit board that sits on top of the DRA-30, making electrical connections to it via header receptacles on the bottom that connect to header pins on the DRA-30.
Some of those headers come with the DRA-30, and some I had to solder on.
The add-on board sits very low on the DRA-30 allowing it to fit inside the original DRA-30 case (when the tall stacked USB connector for the GPS is not installed.)
A USB-C connector replaces of the USB type B connector on the DRA-30.
Access to the DRA-30 level adjustment trimpots and LEDs is available via cutouts in the add-on board PCB.

## Project Motivation

The DRA-30 uses a [GPIO on the Cmedia CM119A sound card chip for PTT](https://www.masterscommunications.com/products/radio-adapter/faq/hardware-ptt.html).
This works fine for Vara FM and SoundModem.
But I wanted to use Fldigi and Direwolf on Windows, but neither supported this method of PTT.
There is [a software solution involving com0com and CAT7200 that allows a virtual com port to trigger the GPIO output on the Cmedia chip](http://www.masterscommunications.com/products/radio-adapter/pdf/fldigi_Using_C-Media_Interface.pdf).
Due to issues with how the com port driver is signed, installing this solution requires weakening the security of Windows.
I'm not willing to do that.
So the motivation for this project was adding a serial port so that the software could assert RTS to trigger PTT.
Most software supports this scheme.

I also wanted VOX in order to use some software that doesn't support either of the methods described above.
As of yet I haven't needed it, but the feature is implemented and it works really well.
It certainly easier to use than the RTS method.

Simply buying a DRA-65 or DRA-70, which include VOX, would have been a perfectly fine solution for the PTT problem.
Or add a downstream VOX circuit after the DB-9, although the audio levels of the DRA-30 are too low to drive [the VOX-10](http://www.masterscommunications.com/products/radio-adapter/vox10/vox10.html).
But I already had the DRA-30, and now I have a USB-C connector, a GPS, and a serial port as well.
Plus it was fun and educational to make.

## DB-9

I have mixed feelings about the DB-9 (DE-9 for the pedantic.)
On the one hand, it's an ancient and clunky connector that extends a good distance out the back of the interface.
On the other hand, it is easy to wire up whatever cable configuration is needed for any radio, including space for a capacitor and resistor that some HTs require, or a circuit to convert the UART TTL signals to conforming RS-232 signals.
A cable for each radio eliminates the need for something like the set of jumpers that the SignaLink uses to configure the pins of the radio connector.

![db-9 circuit](db-9-circuit.jpg)

## DRA-30 Modifications

It's easier to make these modifications while you are building the board instead of desoldering things.

I did not install the R-PTT resistors.

I did not install JU1 and JU2 headers because I don't need COS or CTCSS and want these DB-9 lines for the serial port TX and RX instead.

I removed the USB type B connector and replaced it with a 4-pin square header.
This is so the DRA-30 can connect to the USB hub chip on the add-on board instead of directly to the computer.
The pin spacing is not quite right for a standard header, but I was able to manipulate the pins enough to make it fit.

I added a 2-pin header in holes marked "2" and "4" to get access to pins 2 and 4 on the DB-9.
The holes happen to be the right distance apart to accept a standard 2-pin header.
I could have used the headers labeled "JU1" and "JU2" for this but chose not to because of the convenient spacing of the "2" and "4" holes.
This header could be omitted if the serial port is only needed for PTT and TX and RX are not needed on the DB-9.

I added a 1-pin header to the hole marked "7" to get access to pin 7 on the DB-9.
I put 5V on this pin to power a serial port level converter inside the DB-9 housing if needed.
This header could be omitted if 5V is not needed.

I snipped about 1mm off the header pins to allow the board to sit lower on the DRA-30 so that it would fit inside the original case.
I should have made the dimensions of the board slightly smaller so that it would fit inside the ridges on the lid of the case.
This would have made snipping the header pins unnecessary, and I will make this change on the next revision of the board.

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
Otherwise, if RTS goes low, PTT is triggered.

If the VOX DIP switch is OFF, the VOX circuitry input is ignored.
Otherwise, if the audio detect signal is above the threshold specified by the VOX sensitivity pot, PTT is triggered.
PTT is not released until a time specified by the tail time trimpot after the audio detect level drops off.

## VOX

The VOX design is inspired by [this post](http://kb9rlw.blogspot.com/2016/08/cheap-and-easy-to-build-digital-modes.html).
The left audio channel is used as the input to the VOX circuit.
While both left and right audio channels are routed the the DB-9, I build my radio cables to send the right audio channel to my radios.
In order for VOX to work, I configure my software to send audio to both channels.
This allows me to adjust the left audio trimpots on DRA-30 to its highest level to increase VOX reliability while adjusting the right audio level to the appropriate level for your radio.
In practice, I have both level maxed out, and Windows audio levels at 100%.
A DRA-45 might have been a better starting point since it has higher output levels, but the DRA-30 provides just enough.

Fldigi allows you put a constant tone on one channel to support modes that rise and fall in audio signal level that would otherwise cause the PTT to drop without a long tail hang time.
For my interface, that would be the left channel.
The Fldigi "Soundcard/right channel" settings, check the following boxes:
- [x] Reverse right and left channels.
- [x] PTT tone on right audio channel.

## PTT

The microcontroller triggers a MOSFET to ground the PTT line on the DB-9.
The SMT MOSFET can sink 300mA from the radio's PTT line to ground, which is plenty for my setup.
I can also solder in a through-hole BS170 should I need more current, up to 500mA.

[This article](http://www.masterscommunications.com/products/radio-adapter/faq/hardware-ptt.html) states that the original DRA interface design also used a MOSFET but they switched to a transistor due to RFI affecting the MOSFET's reliability.
I will stick with my MOSFET for now and see what happens.

A red LED lights when PTT is asserted.

The PTT will be held for a maximum of 5 minutes each time it is triggered.
I pulled this number out of the air, but if it proves to be too short I can change the microcontroller firmware.
In a future revision, I would like to add an LED to indicate this timeout condition.

In my next board revision, I will add the COMM OK signal from the DRA-30 555 timer as an input to the microcontroller as a additional requirement to assert PTT.

## Serial Port

I chose the FTDI FT232R USB-to-serial port chip.
I like this chip because I can configure it via a utility program if I ever need to invert the TX or RX signal.
It also solves a problem with Windows asserting RTS when at inopportune times.

Windows will assert RTS several times when the serial port is powered up, and when some other USB serial port is added or removed from the system.
If your radio is connected to the interface when this happens, PTT will trigger and your radio will transmit a few short pulses.
This is a pain and you have to manage it manually by turning on the radio last or connecting it after Windows has finished doing its thing.
But the FTDI chip can be configured to prevent this via the device manager: under the advanced properties settings, check "Disable Modem Ctrl At Startup" and uncheck "Serial Enumerator".
This is a huge convenience.

There TX and RX LEDs on the board that are handy for troubleshooting.

## USB Hub

The serial port is a USB device.
Without a hub, I would need two USB connections to the computer or an external hub.
So I built the USB hub into the add-on board--it's a chip and a handful of capacitors and resistors.
Now the hub is the only thing connected to the computer, and the serial port, DRA-30, and GPS all use ports on the hub.
There is one USB port left over on the hub for future expansion.
This is available via the stacked USB connector: there should be enough clearance to plug in a USB cable or a very flat PCB.

## GPS

Two of the USB hub ports are connected to a stacked double USB connector on the add-on board.
The GPS is an unmodified dongle from u-blox that plugs into the top port of that connector.
It appears to the computer as another serial port device.
By default, it spits out NMEA sentences at a rate of once per second at 9600 baud.
This is what I want and I didn't have to configure it in any way.
This is useful for APRS, Winlink position reports, and for accurately syncing the computer's time if a network is not available.

## Case

You can use the original DRA case if you don't need the GPS and stacked USB connector.
Since that case is translucent, you can see the LEDs from the outside.

The larger case I found has ventilation slits in the top that allow me to see the state of the LEDs.
But I have to be looking at it from the right angle to see them.
I would like to mount the LEDs on the outside so they are easier to see.
I have yet to decide if this change is worth it.

## Final Thoughts

I have been using this interface and several prototype versions for several months with a Yaesu FT-530 HT and a Kenwood TM-V71A dual-band mobile.
I have used SoundModem, Direwolf, Vara FM, Winlink, APRS, and Fldigi.
It is serving me well, but I already have ideas for the next version:
- use COMM OK as a prerequisite to trigger PTT,
- make the dimensions of the board slightly smaller,
- route DTR to the microcontroller for future use,
- mount the indicator LEDs on the outside of the case,
- add an LED to the microcontroller to indicate PTT timeout.

If I allow the microcontroller access to the serial port TX and RX lines, I could implement a configuration interface via the serial port.
Eg, [AT commands](https://en.wikipedia.org/wiki/Hayes_command_set).
Then I could eliminate the DIP switches and/or VOX trimpots.
Plus it would enable a 4th way to trigger PTT: a serial port command.
Like CAT control, but to the interface instead of the radio.
I don't know if this is a good idea or not.

Additionally, that would allow the interface to translate from RTS or VOX to a CAT command if such a thing were ever needed.
The scenario would be a radio that only uses CAT for PTT and software that doesn't support CAT control.
It's and interesting capability at least.

If I were to start this project over from scratch, I would start with the DRA-45 or DRA-65, depending on whether or not I think the VOX controls on my board are worth anything.
If not, I would use the DRA-65 which already has VOX.
If so, I would use the DRA-45 plus my own VOX circuit with the level trimpots.

Since I'm already building a PCB and circuit, another option is to just start with a bare Cmedia chip, duplicating the DRA's audio functionality.
Some of the circuity on the DRA boards is boilerplate to support the Cmedia chip: the crystal and bunch of capacitors.
I wouldn't need the 555 and related components or the transistors because LED and GPIO output would be connected to inputs on the microcontroller, which would implement the heartbeat protection in firmware.
The interesting bits are the audio paths.
I'd borrow those as-is.

Would a more powerful microcontroller and I2S audo from a Cmedia chip simplify VOX?
This would eliminate all the discrete components of the VOX circuit, but at the expense of a more complicated microcontroller.
Maybe at that point would be better to go all in with something like [this setup with a Teensy board.](http://www.kk5jy.net/AnyRig-v1/).

Maybe a microcontroller is a bad idea in an RFI environment.
I guess I will find out.


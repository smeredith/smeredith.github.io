# Teensy Radio Interface

I assembled a radio interface using a [Teensy 3.2](https://www.pjrc.com/store/teensy32.html) and [audio board](https://www.pjrc.com/store/teensy3_audio.html) from PJRC.
A minimal implementation only requires those two modules plus an additional resistor and transitor for PTT and a connector for your radio.
I wanted something a little more complex, so my build also has a GPS and a serial port to program and control my radio.

## Overview

My interface connects to a computer with a micro-USB connector.
To the computer it looks like a USB sound card and two serial ports.
The first serial port is connected to the radio and can be used for CAT control or programming.
The blue and green "TX" and "RX" LEDs light up whenever a byte is sent or received.
This serial port can also be used for PTT via one of its control lines (RTS or DTR.)
The second serial port is connected to the GPS.

The outgoing left audio channel from the computer is connected to the radio.
The outgoing right audio channel from the computer is used for VOX, if enabled.
Incoming audio from radio is routed to both channels on the computer.

I found insipration from the [AnyRig](http://www.kk5jy.net/AnyRig-v1/) interface, which is more flexible but more complicated.
Because I only need it to support the one radio I own, I was able to keep it simple.
For example, for my implementation, I don't need audio isolation, an opto-isolated PTT switch, or CW keying.

## PTT

There are three ways to trigger PTT:
- RTS on the first serial port (see note below)
- DTR on the first serial port
- VOX.

Note: there is a [known bug on Windows with RTS.](https://forum.pjrc.com/threads/65829-Serial-rts()-on-Teensy-3-2?p=266761&viewfull=1#post266761).
I use DTR.
RTS should work on Linux.

The red "PTT" LED lights when PTT is triggered.

I can configure the PTT method via either one of the serial ports.
For example, to enable DTR, I send the command "ATDTRON".
To disable it, I send "ATDTROFF".

There is a 5 minute PTT timeout.
If PTT is held longer than that, it is released and the "STATUS" LED flashes.
This is automatically reset once the PTT trigger is released.

## GPS

The GPS module (center) continuously sends NMEA sentences out the second serial port.
This can be used to set the computer's time or get position information for APRS.

To disable the GPS, I can send a command via either serial port: "ATGPSOFF".
It will remain off until I send "ATGPSON".

## Headphone Jack

The headphone jack can be used to monitor both sides of a digital conversation, and the level can be controlled from the computer's audio control.
Audio from the radio is routed the the right channel.
Audio from the computer is routed to the left channel.

## Radio Connectors

The mini-DIN connectors match those on the radio so that I can use standard cables.
The labels "PC" and "DATA" don't make much sense, but they match the labels on the radio.
"DATA" (6-pin) carries audio and PTT.
"PC" (8-pin) carries the serial port data.
Note that I use a MAX3232 to get the right voltage levels for the radio.

## Case

I am using a clear case so that I can see the LED indicators without having to mount them externally.
The PTT LED is really bright.
The serial port RX and TX LEDs are much dimmer as to be less distracting.
The Teensy 3.2 has a small orange LED to indicate power.
The GPS module has a small green LED to indicate power, and a dim orange LED that flashes once per second when it has a positon fix.

## Power Consumption

With the GPS off, the device draws 39mA.
With the GPS on, it draws between 70mA and 78mA.

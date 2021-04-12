# Teensy Radio Interface

I assembled a radio interface using a Teensy 3.2 and audio board from PJRC.
A minimal implementation only requires an additional resistor and transitor for PTT and a connector for your radio.
I wanted something a little more complex, so my build also has a GPS and a serial port to program and control my radio.

## Overview

The interface connects to a computer with a micro-USB connector.
To the computer it looks like a sound card and two serial ports.
The first serial port is connected to the radio and can be used for CAT control or programming.
The blue and green "TX" and "RX" LEDs light up whenever a byte is sent or received.
This serial port can also be used for PTT via one of its control lines (RTS or DTR.)
The second serial port is connected to the GPS.

The left audio channel of the sound card is connected to the radio.
The right audio channel is used for VOX, if enabled.


## PTT

There are three ways to trigger PTT:
- RTS on the first serial port
- DTR on the first serial port
- VOX.

The red "PTT" LED lights when PTT is triggered.

I can configure the PTT method via either one of the serial ports.
For example, to enable DTR, I send the command "ATDTRON".
To disable it, I send "ATDTROFF".

There is a 5 minute PTT timeout.
If PTT is held longer than that, it is released and the "STATUS" LED flashes.
This is automatically reset once the PTT trigger is released.

## GPS

The GPS continuously sends NMEA sentences out the second serial port.
This can be used to set the computer's time or get position information for APRS.

To disable the GPS, I can send a command via either serial port: "ATGPSOFF".
It will remain off until I send "ATGPSON".

## Power Consumption

With the GPS off, the device draws 39mA.
With the GPS on, it draws between 70mA and 78mA.

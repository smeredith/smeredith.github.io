---
layout: post
title: Using a Networked Brother HL-3170CDW Printer with FreeBSD
---

I had some trouble finding the information I needed to use a Brother HL-3170CDW with FreeBSD over a network.
Here is what I learned:

I didn't need to install any software from Brother.
I didn't set any options in the printer itself, except those needed to get on the network.
I added the printer using KDE's "Add Printer" option in System Settings.

Connection string: "ipp://\<ip-addr\>/ipp/port1".

You'll need to substitute the IP address of your own printer, but "port1" is the literal string you need: don't substitute anything for that.

Driver: "Generic PostScript Printer".

Find the properties dialog and check "Duplexer: installed."

Color printing does work.
So does double-sided printing.

If that doesn't work, you can select the "Generic PDF Printer" driver.
Again, you'll get color and double-sided printing.

You can also select the "Generic PCL 6/PCL XL Printer", but you get black and white.


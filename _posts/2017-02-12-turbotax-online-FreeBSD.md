---
layout: post
title: Unix Users Pay Taxes Too
---

For some reason, TurboTax online rejects browsers from FreeBSD.
It tells you that your browser is out of date, not that you are using an unsupported OS.
To do my taxes, I had to fool TurboTax into thinking I was browsing from a Windows machine.

I use Firefox, so changing the user agent string was pretty easy.

1. Navigate to "about:config".
1. Add a new key "general.useragent.override" with value "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:18.0) Gecko/20100101 Firefox/51.0.1".
1. Do your taxes.
1. Reset the key.

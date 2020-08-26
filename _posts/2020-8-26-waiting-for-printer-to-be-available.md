---
layout: post
title: Waiting for printer to be available..
date: 2020-8-28 21:36:00
tags: [frontpage]
categories: [electronics]
img: pi-printerusb-disconnect.png
---

[My wireless printer]({% post_url 2020-8-18-print-server-on-raspberry-pi-zero %}) worked fine for 10 days and then stopped working. 
Print jobs are queued but are not printed and even CUPS doesn't detect the printer anymore. After looking at the logs at `/var/log/messages`,
it appears that the printer was constantly getting disconnected.

```
Aug 26 23:38:25 raspberrypi kernel: [  172.606211] usblp0: removed
Aug 26 23:38:34 raspberrypi kernel: [  180.837856] usblp 1-1:1.0: usblp0: USB Bidirectional printer dev 2 if 0 alt 0 proto 2 vid 0x03F0 pid 0x2B17
Aug 26 23:38:34 raspberrypi kernel: [  181.174488] usblp0: removed
Aug 26 23:38:42 raspberrypi kernel: [  189.214852] usblp 1-1:1.0: usblp0: USB Bidirectional printer dev 2 if 0 alt 0 proto 2 vid 0x03F0 pid 0x2B17
Aug 26 23:38:43 raspberrypi kernel: [  189.795780] usblp0: removed
```

I don't know why and how, but the following two commands fixed the problem.

```
## lpstat -p
## lpadmin -p <insert printer name found from above command> -o usb-unidir-default=true 

pi@raspberrypi:~ $ lpstat -p
printer HP_LaserJet_1020 now printing HP_LaserJet_1020-59.  enabled since Wed 26 Aug 2020 11:39:49 PM IST
        Waiting for printer to become available.
pi@raspberrypi:~ $ lpadmin -p HP_LaserJet_1020 -o usb-unidir-default=true
```

Found these from the following thread - https://bugzilla.redhat.com/show_bug.cgi?id=873123
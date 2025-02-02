---
layout: post
title:  "Inside DuetDisplay and Unattributed Open-Source"
date:   2014-12-18 13:12:43
categories: re
---

<link rel="canonical" href="https://ich.deanmcnamee.com/re/2014/12/18/DuetDisplay.html" />

I read about Duet a few days before its release.  There were some interesting
claims being suggested &mdash; "ex-Apple engineer" is able to run a display over the
lightning connector, with zero latency.  The marketing made it sound like this
Apple-insiderness allowed them to do something new, perhaps a hardware level / 
accelerated connection with the iOS device to run it as a "real" display at 60
FPS.  So how does it work?

Overview
========

The Duet driver creates an additional display, a memory backed framebuffer.
The pixel contents of this framebuffer display are captured and compressed
(with the help of CocoaSplit) and sent to the iOS device through the iTunes TCP
over USB transport mechanism implemented by PeerTalk.  So in the end this is
quite like a VNC over TCP over UDP, so I find the claims of "zero lag" and 60
FPS to be a bit dishonest.  This is not something like a DisplayLink
connection.

Open-Source Libraries
=====================

Before Duet was released for iOS, the Desktop Duet.app was available to
download.  Taking a look at the binary revealed code from a few open-source
libraries:

- [PeerTalk](https://github.com/rsms/peertalk)
- [CocoaSplit](https://github.com/zakk4223/CocoaSplit)
- [GPUImage](https://github.com/BradLarson/GPUImage)

![Credits?](/images/DuetDisplay-credits-400.png)

Credits? I am not a lawyer and generally not a psychopath about software
licensing.  However, having spent much of my life on open-source software,
I feel that when someone gives you something to use, commercially, free of
charge, with attribution as the only obligation, that's a pretty good deal.
You really have a responsibility to do the right (and legal) thing.  As far as
I've seen, none of the above libraries are attributed anywhere in Duet.
Additionally it seems [CocoaSplit is licensed under the GPL](https://github.com/zakk4223/CocoaSplit/blob/master/CocoaSplit/en.lproj/Credits.rtf).  Uh oh.


PeerTalk
--------

PeerTalk uses the iTunes [usbmux](https://theiphonewiki.com/wiki/Usbmux) system
to relay TCP connections across the iOS USB connection.  I am surprised that
Apple has approved this in the AppStore, as it uses a reversed engineered
interface to the usbmuxd socket.  The good news is that if Apple has approved
Duet, it means if all is fair anyone else should be able to use usbmux also.

![scheduleReadPacketWithCallback](/images/DuetDisplay-peertalk-700.png)

Disassembly of "DuetUSBChannel" as compared with PTUSBChannel [scheduleReadPacketWithCallback](https://github.com/rsms/peertalk/blob/master/peertalk/PTUSBHub.m#L516).  It seems the main 
difference between PeerTalk and the version in Duet, is the prefix has been renamed
from PT to Duet.  Why?

CocoaSplit
----------

CocoaSplit is designed for capturing and streaming a display.

![420v.fgsh](/images/DuetDisplay-420v-fgsh-700.png)
420v fragement shader included in Duet.app, as compared to [the shader in CocoaSplit](https://github.com/zakk4223/CocoaSplit/blob/master/CocoaSplit/420v.fgsh).  Now, this is quite similar to
[one of the examples from Apple](https://developer.apple.com/library/ios/samplecode/GLCameraRipple/Listings/GLCameraRipple_Shaders_Shader_fsh.html), however notice that the variables are renamed
and it matches the whitespace and naming from CocoaSplit identically.

![PreviewView](/images/DuetDisplay-PreviewView-700.png)

Duet's PreviewView, as compared with [the same from CocoaSplit](https://github.com/zakk4223/CocoaSplit/blob/master/CocoaSplit/PreviewView.m#L60).

GPUImage
--------

GPUImage is a general purpose library of image processing via OpenGL.

![GPUImage](/images/DuetDisplay-GPUImage-400.png)

[GPUImageFramebuffer](https://github.com/BradLarson/GPUImage/blob/master/framework/Source/GPUImageFramebuffer.m) as seen in Duet.app.

Driver
======

I haven't looked very deeply at the driver, but it is implementing a
virtual screen framebuffer, similar to what is done by
[EWProxyFramebuffer](https://github.com/mkernel/EWProxyFramebuffer).
I'm not suggesting that any of the code from EWProxyFramebuffer was used.
However, if you were to try to implement something similar that would be a good
starting point.

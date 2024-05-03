---
layout: post
title: "Sawppy: Raspberry Pi 4B"
description: >
    Replacing the Arduino Nano-based joystick controller with Raspberry Pi 4B
    over WiFi.
category: Star-the-Rover
tags: [Robotics, Rover, 3D-Printing, Sawppy, raspberry-pi]
author: Nicholas Bering
---

Yesterday, we put a little work into tightening up all the joints on Star (our
family's Sawppy Rover build). After a reasonably long test drive of about 30
minutes, there were not discern-able issues with loose set screws on the wheel
and steering shafts.

Not having to tune up drive parts afforded me some time in the evening to
replace the Arduino Nano-based Joystick control board we'd been using. The new
Raspberry Pi 4B/2GB board had just arrived this week, so this was the perfect
opportunity.

## Powering it Up

Powering the Raspberry Pi after installation in the Rover was pretty simple. I
already have a DC/DC Buck Converter powering the Arduino board. It takes the
LiPo battery power from the 2S battery pack (ranging 7.4-8.4V), and steps it
down to a voltage acceptable to the control circuitry.

I just needed to make sure the output was set to 5V for the Raspberry Pi, and
connect it to GPIO pins 1 (5V), and 3 (GND) on the Raspberry Pi. It booted as
soon as I turned on the power, and there were no under-volt issues reported in
the system logs.

## Initial Setup

Before powering the board up on the rover, I needed to do some initial setup at
my desk. It would have been difficult to get a monitor and keyboard hooked up
to the rover chassis so I could run the first commands to enable SSH, join the
WiFi network, etc.

I'd hoped to power it at my desk from my MacBook's charger, but because the
[USB-C connector on the RPi 4 is non-conforming][1], it didn't work with any of
the USB-C to USB-C cables I have. So I had to rob a wall wart power supply from
another running Raspberry Pi in my office to get it started.

Once I'd sorted out power, everything else was a breeze. Using the `raspi-config`
command, I set the keyboard locale, WiFi settings, changed the hostname to the
somewhat whimsical name "star-rover", and so on.

The only other minor hiccup I had was that I tried to configure it to connect to
my 5GHz network, but that doesn't appear to be supported. But since my router is
Dual-Band, I just used the regular 4Ghz SSID and everything was good to go.

## Connecting the BusLinker Board

The last thing I needed to do for physical hardware setup was connect to the
Micro-USB port on the LewanSoul BusLinker board.

It's using a CH431 USB-UART chip onboard, which is supported by the Raspberry Pi
OS without any additional drivers.

The only challenge I had here, was actually finding a short enough USB cable
that wouldn't be awkwardly-long.

I have plenty of little Micro USB cables around, but many came with devices that
only need it to charge a battery, and a lot of these only have 2 wires attached
to the connector to make them cheaper. Very few of these actually have any
marking to indicate that they don't support data, so it took some trial-and-error
to pick the right cable.

But once I had the right one, the BusLinker connecting showed up in `dmesg`, and
`lsusb -t` so I knew all was good to go.

## Next Steps

I have some chores to do today before I can pick up where I left off, but I plan
to install the [original interface][2] that was used by Sawppy Rover.

[1]: https://hackaday.com/2019/07/16/exploring-the-raspberry-pi-4-usb-c-issue-in-depth/
[2]: https://github.com/Roger-random/Sawppy_Rover/blob/main/docs/SGVHAK%20Rover%20Software.md

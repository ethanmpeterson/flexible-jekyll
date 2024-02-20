---
layout: post
title: "STM32 FSAE Steering PCB"
date: 2021-05-01 14:30:20 +0300
description: An STM32 combined steering and dashboard PCB for a formula car.
img: steering/thumb.jpeg # Add image post (optional)
fig-caption: The PCB assembled into a functioning formula steering wheel!
tags: [CAN Bus, PCB Assembly, Altium Designer, Car Dashboard, LCD Display]
---
<script src="https://viewer.altium.com/client/static/js/embed.js"></script>

# Introduction
The Steering PCB (hereby referred to as SPCB) was one of my final projects
before graduating and ending my time on the [Queen's Racing Formula SAE
Team](https://www.instagram.com/queensracing/?hl=en). The SPCB serves as a
driver dashboard and gear-shifting interface fully packaged into the steering
wheel of the racecar. This design attempts to mirror the steering wheels used in
formula one where the display and related electronics are packaged into the
steering wheel so that the driver can always see the display without obstruction
from steering wheel movements.

The SPCB builds on the work done in the [Debugger]() and [Mock ECU]() projects
by employing an STM32 microcontroller. In the past, the team's dashboard has
been Arduino based. The goal of SPCB is to upgrade the microcontroller to STM32
and provide a truly re-usable design that allows the team to continue to build
out more sophisticated firmware in the coming seasons.

STM32 provides two advantages; speed and native CAN buses. Unlike the Arduino's
based on ATMEGA328P that have been used in the past, the STM32F4 used in the
SPCB has two native CAN controllers. The native CAN controllers open a variety
of possibilities for multiple CAN buses in the vehicle, gatewaying, interrupt
driven message transmitting and more. Moreover, the STM32 is signifcantly faster
and is able to drive the steering wheel's display. STM32 and other ARM based
microcontrollers are also very commonly used in industry. As a result, there is
a wealth of resources for bringing up new tools and improving the team's
firmware development workflow to an industry representative standard. In this
first implementation, the team leveraged the pre-existing arduino toolchain to
quickly prepare firmware for the car's first drive. However, in future, the team
can create their own hardware abstraction layer for the microcontroller and
integrate an RTOS without the limitations of the Arduino ecosystem.

The SPCB breaks out IO for the following driver interfaces.
- A high-brightness LCD display for the driver to consume real-time information
  about the vehicle via ECU CAN data.
- High-brightness RGB neopixels as an easily readable RPM meter telling the
  driver when to shift (modeled after today's F1 cars).
- 4 buttons to control which information is on the screen and configure the display.
- 2 button inputs for paddle shifting. (The steering PCB is responsible for
  relaying shifting commands to the vehicle's ECU).

The following subsections explain how the functionality listed above was
implemented at both the hardware and firmware level.

# Hardware Design
The schematic design of the SPCB is broken down into several sub-circuits in a
multi-sheet design on Altium. The following sections break down each component
of the SPCB schematic design.

## Power Distribution
The SPCB manages the following three power domains.
- 12V bus voltage from the car's battery.
- 5V power for the neopixel LEDs and LCD display.
- 3.3V logic power for the microcontroller and LCD SPI interface.

There are two LDO voltage regulators that provide the 5 and 3.3V rails
respectively. The 5V regulator uses the 12V bus voltage as input voltage. The 5V
output is used to feed the 3.3V regulator. There is no dedicated control of
either power rail and both will power up as soon as a 12V bus voltage is
applied. A dedicated LED is provided on each power rail as a visual indication
of board power. The power schematic is located in the `Power.SchDoc` file
[here](https://github.com/qfsae/pcb/tree/master/Steering).

## Neopixel Control
[Neopixels](https://learn.adafruit.com/adafruit-neopixel-uberguide/the-magic-of-neopixels)
are used as the RPM indicator lights on the SPCB. Neopixels take their RGB color
and brightness commands via a one-wire data interface. This interface is
bi-directional and uses 5V logic to match the 5V supply voltage.

The 5V logic of the neopixels does not match the 3.3V logic of the STM32
microcontroller commanding the neopixels. This mismatch can damage the STM32
with overvoltage and prevent the neopixels from functioning correctly. The
solution is to use a logic level translator. The translator steps the STM32
control signal up from 3.3 to 5V. Conversely, it steps any signals driven by the
neopixels down from 5 to 3.3V allowing both devices to communicate seamlessly.

The level translator and neopixel schematic is located in `Neopixel.SchDoc`
[here](https://github.com/qfsae/pcb/tree/master/Steering).

## LCD Control
The microcontroller communicates with the LCD via SPI. Rather than sending a
full framebuffer, individual SPI commands are sent to an FT813 display
controller. As a result, the PCB only needs to provide a SPI interface to the
LCD. The schematics are located in `MCU.SchDoc` and `Connectors.SchDoc`.

## Debug Interface
The SPCB implements both JTAG and USART debug interfaces to be used in
conjunction with the custom [Queen's Formula Debugger](../debugger). In future
revisions, this debug interface can be integrated directly into the PCB.
However, this was not done on the first revision due to the chip shortage and
difficulty acquiring STM32 chips specifically.

## CAN Bus Design

- Switchable termination, microcontroller controlled IO

## Project Viewer
Embedded below is the Altium project view of the SPCB project. The embedded viewer
contains all the schematics that constitute the multi-sheet design as well as
the PCB layout, BOM and 3D renders. The complete Altium CAD files can be found 
[here](https://github.com/qfsae/pcb/tree/master/Steering).

<iframe src="https://personal-viewer.365.altium.com/client/index.html?feature=embed&source=6934B02C-5ED8-45D2-84E8-5A2D8FEC4D79&activeView=PCB" width="1280" height="720" style="overflow:hidden;border:none;width:100%;height:720px;" scrolling="no" allowfullscreen="true" onload="window.top.scrollTo(0,0);"></iframe>

# Software Design
- Basic bring-up and define file for all the pinouts.
- Bring-up of the display
- Jacob's CAN decoder library
- Video showing all that info piped out onto a display.
- Shifting over CAN and improvements to that behavior.
- Brightness limitation in firmware to avoid browning out the 5V feed to the
  display. Suggest use of a buck regulator like the one in PMBOARD project (link
  it)

# Gallery
- Photos and videos of the board in action
- instagram embeds and links

# Conclusion
- Strong way to end time on the electrical team with a working dashboard design
  that can be re-used along with a fully documented vehicle harness.

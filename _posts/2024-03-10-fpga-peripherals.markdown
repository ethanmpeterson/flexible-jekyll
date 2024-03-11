---
layout: post
title: "FPGA Project: A Peripherals Library Written in SystemVerilog"
date: 2024-03-10 14:30:20 +0300
description: A variety of SystemVerilog abstractions for common communication protocols.
img: peripherals/thumb.png # Add image post (optional)
fig-caption: Arty A7 FPGA kit used for development.
tags: [FPGA, SystemVerilog, SPI, UART, MDIO, MII, Ethernet]
---

# Introduction
After completing the [TeachEE](../teachee) project, I began to further explore
FPGAs in my spare time after work. The result was the
[`peripherals`](https://github.com/ethanmpeterson/peripherals) library. This
library implements a variety of communications peripherals typically found in
microcontrollers on an FPGA. The library is implemented in SystemVerilog and
provides a standardized AXI interfaces for using these protocols on an FPGA.

At the time of writing, I have implemented the following:
- [A SPI Master](https://github.com/ethanmpeterson/peripherals/blob/449dad196a008e544701788d95b37d0976b4c1e2/rtl/src/spi/spi_master.sv)
- [An MDIO Master](https://github.com/ethanmpeterson/peripherals/blob/449dad196a008e544701788d95b37d0976b4c1e2/rtl/src/ethernet/mdio/mdio_master.sv)
- [A UART Peripheral](https://github.com/ethanmpeterson/peripherals/blob/449dad196a008e544701788d95b37d0976b4c1e2/rtl/src/uart/uart.sv)
- [UDP Transmitter](https://github.com/ethanmpeterson/peripherals/blob/449dad196a008e544701788d95b37d0976b4c1e2/rtl/src/udp/udp_tx_example.sv)
- [UDP Receiver](https://github.com/ethanmpeterson/peripherals/blob/449dad196a008e544701788d95b37d0976b4c1e2/rtl/src/udp/udp_rx_example.sv)
- [UDP Loopback Module](https://github.com/ethanmpeterson/peripherals/blob/449dad196a008e544701788d95b37d0976b4c1e2/rtl/src/udp/udp_echo_example.sv)

My goals behind creating this library were to:
- Improve my FPGA development skillset.
- Design custom state machines to implement many of the peripherals we take for
  granted on a microcontroller.
- Provide a reference for my own future projects to quickly integrate these
  protocols in an FPGA development environment.
- Utilize Ethernet and begin taking advantage of higher-speed communication
  protocols (i.e UDP).

This library was made strictly to learn how to implement these protocols at the
hardware level. While I have validated all the modules in simulation and
on-bench, they are likely not as thorough or capable as some of the other
mainstream open source options available. They do serve as a great introduction
to these protocols and systems nonetheless.

# The Peripherals Repository
The [`peripherals`](https://github.com/ethanmpeterson/peripherals) repository
contains the source modules for each peripherals in addition to VUnit
testbenches, bus functional models (BFMs), and usage examples.

All the modules are regression tested by VUnit testbench. Each testbench is run
on every commit using this [GitHub Actions
job](https://github.com/ethanmpeterson/peripherals/blob/449dad196a008e544701788d95b37d0976b4c1e2/.github/workflows/rtl.yml)

This GitHub actions infrastructure is taken from my project template
[here](https://github.com/ethanmpeterson/rtl-project), which was in-turn
influenced by the [GitLab template from Alex
Lao](https://gitlab.com/lao.alex.512/hdl_project_template).

# Modules
The following subsections provide a brief overview of each peripheral available
in the library. All the modules use an AXI stream or AXI Lite interface. Below
are some recommended resources for AXI.
- [AXI Introduction Part 1: How AXI works and AXI-Lite transaction
  example](https://www.youtube.com/watch?v=p5RIVEuxUds)
- [AXI Stream Explained](https://www.youtube.com/watch?v=GyYmSZZor1s)

## UART
TODO

## SPI
TODO

## Ethernet
TODO

## MDIO
TODO

# Demos

## Ethernet UDP Echo
TODO

## MDIO Link Light Blink
TODO


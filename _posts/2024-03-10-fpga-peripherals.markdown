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


- After doing TeachEE, wanted to do more FPGA design work.
- Built out an RTL project template based off the Github actions and VUNIT
  infratrsucture I set up for TeachEE. Which was in turn inspired to Alex's
  gitlab template
- Specifically, was curious about getting FPGA talking over ethernet and bringing up some basic peripherals
  - SPI
  - MDIO
  - UART
- Wanted to extend knowledge of AXI and start using AXI Lite instead of just AXI stream like in TeachEE
- This library might be a handy reference to those getting started with FPGAs
  and designing similar state machines.
- However, I would not recommend for production use. There are better reference
  designs and IPs online that are properly validated. I brought these for
  learning purposes only and will likely modify them when they are needed in
  another project.


- The hardware is an ARTY A7 FPGA devkit. I chose this one because it is quite
  popular online and has plenty of documentation.
  - And critically, it contains a 10/100 ethernet PHY.

# The Peripherals Repository
- `peripherals` is a monorepo containing various peripherals I brought up on the
  Arty A7 with corresponding VUNIT validation tests and examples.
- The repo also contains a full Xilinx Vivado project and pin assignments /
  timing constraints specific to the A7 board.
- If you also have an A7, it should be plug and play.
- Reference the templates from my github and Alex's

# Modules
The peripherals library at the time of writing has SPI, UART, MDIO, and ethernet
abstractions available. These modules are written to employ AXI streams or AXI
Lite. As a result, they all share a common interface that is commonly used in
FPGA datapaths. To learn more about AXI, I recommend the following resources:
- Link 1 (Stacy's video on AXI lite)
- Link 2 ("Dedicated Xilinx Page on AXI lite")
- Link 3 ("Alex's readme from verilog axis")

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


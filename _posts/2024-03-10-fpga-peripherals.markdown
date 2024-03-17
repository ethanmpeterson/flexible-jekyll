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
The UART peripheral implementation can be found [here](https://github.com/ethanmpeterson/peripherals/tree/449dad196a008e544701788d95b37d0976b4c1e2/rtl/src/uart). The module interface is as-follows:
```verilog
module uart #(
    parameter TX_FIFO_DEPTH = 32,
    parameter RX_FIFO_DEPTH = 32,

    // Equates to a 115200 bps rate on 100 MHz clock.
    parameter CLKS_PER_BIT = 868
) (

    // asynchronous RX signal
    input var logic rxd,

    output var logic txd,

    axis_interface.Sink tx_stream,
    axis_interface.Source rx_stream
);
```

Two AXI stream interfaces are used to transmit and receive UART bytes. Both
stream interfaces are connected to a FIFO queue specified by the `FIFO_DEPTH`
parameters. This allows the parent module to queue bytes for transmit
asynchronously to the underlying UART transmitter and avoid the need to
continuously poll received bytes.

The FIFOs are wrapped in an `axis_interface` interface type to group the
relevant AXI stream signals together. The original FIFO module is provided by
the `verilog-axis` [library](https://github.com/alexforencich/verilog-axis).

Instead of generating a low-speed clock to send UART data, the
transmitter/receiver state machines use the `CLKS_PER_BIT` parameter to send
data at the correct baud rate. Using the main high-speed clock and counting
cycles is preferred for the following reasons:

- Full control of when the UART RX line is sampled.
- Avoids creating an additional clock domain and resulting metastability issues.

There is however one metastability concern in the UART receiver. Since the RX
line is asynchronous to the FPGA clock, it must be synchronized before its data
is processed to avoid incorrect sampling and data loss. This synchronization to
the FPGA's clock domain is implemented with a [Two-Flip-Flop (2FF)
Synchronizer](https://www.icdesigntips.com/2020/12/two-ff-synchronizer-explained.html)
[module](https://github.com/ethanmpeterson/peripherals/blob/449dad196a008e544701788d95b37d0976b4c1e2/rtl/src/metastability-tools/sync_slow_signal.sv).

## SPI
The peripherals repository implements a simple [SPI
Master](https://github.com/ethanmpeterson/peripherals/blob/449dad196a008e544701788d95b37d0976b4c1e2/rtl/src/spi/spi_master.sv). The SPI module interface definition is as follows:

```verilog
module spi_master #(
    parameter TRANSFER_WIDTH = 8,

    // default for a 100 MHz clock
    // produces 1 MHz SPI
    parameter CLKS_PER_HALF_BIT = 50,

    parameter CPOL = 0, // sck idle state
    parameter CPHA = 0 // sampling edge (0 for rising, 1 for falling)
) (
    input var logic miso_en,
    spi_interface.Master spi_bus,

    // it is assumed that these two streams share the same clock
    axis_interface.Sink mosi_stream,
    axis_interface.Source miso_stream
);
```
### Parameters
The SPI master module currently only supports fixed data transfer sizes as
defined by the `TRANSFER_WIDTH` parameter. This simplifies some of the logic
around when the CS line is asserted and de-asserted. However, a future
improvement would be to utilize the `tlast` signals in the AXI streams to
determine when CS should be asserted. Alternatively, a future iteration could
give full control of the CS line to the parent module.

Similarly to the UART module, the SPI master also uses the main FPGA system
clock and counts cycles to produce a low-speed SPI clock as defined by the
`CLKS_PER_HALF_BIT` parameter. This approach is beneficial in the SPI use-case
because it allows the module to have precise control over when the MOSI line
transitions and when the MISO line is sampled.

For the MOSI line, it is important to update its state at the correct time
relative to the SCK clock signal so that setup and hold times are respected.
Similarly, the MISO line must be sampled at the correct time. Another benefit is
that FPGA pins can have some jitter when they change states, if the module
operated directly on the low-speed SPI SCK clock, this jitter could lead to data
integrity issues. The drawback of the cycle counting approach is the increased
LUT usage in the FPGA.

The last parameters in the SPI master module are `CPOL` and `CPHA` which define
the SPI mode and clock polarity.

### Main Interface
The SPI master module interface consists of `miso_en`, `spi_bus`, `mosi_stream`,
and `miso_stream`. Each signal / interface is summarized below.

- `miso_en` is used to enable and disable miso functionality in the SPI master
module. This is useful for SPI devices that do not send data back to the master.
- `spi_bus` is a SystemVerilog interface grouping all the SPI signals together.
  (MOSI, MISO, SCK, CS).
- `mosi_stream` is an AXI stream where the parent module can load data to be
  sent to the SPI slave.
- `miso_stream` is an AXI stream output that the parent module may use to read
  data returned by the SPI slave.

### Example Usage
The SPI master peripheral was designed to be wrapped by a parent module that
contains all the device-specific behavior. While testing the SPI master, I wrote
a module which reads and writes registers via [SPI on an ADXL345
IMU](https://github.com/ethanmpeterson/peripherals/blob/449dad196a008e544701788d95b37d0976b4c1e2/rtl/src/examples/adxl345.sv).

## Ethernet
- Majority ethernet infra provided by the AF library
- Made a transmitter, receiver and loopback for UDP.
- Main set up is tying together the modules and configuring the PHY timing constraints

## MDIO
- Used for configuring the PHY
- Explain I2C Similarity
- Configures registers in the PHY. Might be needed to configure sleep, duplex modes, link speed etc.
- Needed for a complete ethernet HDL stack.

# Demos

## Ethernet UDP Echo
TODO

## MDIO Link Light Blink
TODO

# Conclusion
- Modules were a great learning experience that I will continue to draw on.
- Will likely pull this repo into my other projects as a submodule and refactor these modules as I learn more about FPGA development. There are already things I would do differently.
- Great to gain a better understanding about how these protocols function at the HW level.
- Looking forward to sharing other FPGA projects!

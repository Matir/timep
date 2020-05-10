# Test Interface for Multiple Embedded Protocols

This is an Open Source Hardware board based around the FTDI FT2232H chip to
provide breakouts, buffering, and level conversion for a number of common
embedded hardware interfaces.  At present, this includes:

* SPI
* I2C
* JTAG
* SWD
* UART

It's intended to be easy to use and work with open source software, including
tools like OpenOCD and Flashrom.

## Usage

### Hardware Hookup

The hardware has dedicated headers for a variety of electrical interfaces.
Connect the desired interface to you target.  The pinout for SPI, I2C and UART
are printed on the board, SWD follows the ARM SWD standard, and JTAG is the ARM
20 pin JTAG interface.  (Electrically, however, it should work with most any
processor supporting JTAG.)

You may only use one of the SPI, I2C, SWD, and JTAG interfaces at any given
time.  The UART is handled by dedicated pins in the FT2232H, so can be used
simultaneously with one of the other interfaces.

There are 3 switches on the board.  The mode switch selects how the reference
power (for the interface) and the target are connected:

* **Isolated**: The interface is powered by the on-board voltage regulators, and the
  target must be powered by its own power supplies.  The power rails are not
  connected to one another.  You will almost always want to use this mode.
* **Bridged**: The interface is powered by the on-board voltage regulators which
  will also supply power to the target.  **The target must not be separately
  powered.**  This mode is useful for e.g., programming SPI/I2C flash out of
  circuit, or powering very small circuits -- maximum stable load is around 200mA.
* **Target**: The interface is partially powered from the target.
  (Specifically, the reference voltage for the level shifters.)  In theory, this
  can be used for cases where the target voltage is not 5v, 3.3v, or 1.8v.  The
  target voltage must still remain under 5V!

Note that the *ground* of your computer is connected to the *ground* of your
target when you connect with this.  In almost all cases, this is not a problem,
but if both are powered via the wall ("mains"), there is a risk of noise due to
ground loops, etc.

The voltage switch allows you to select the I/O voltage used in `Isolated` or
`Bridged` mode.  It does nothing (well, except change an LED) in `Target` mode.
5V is supplied directly from the USB input, and there are onboard regulators
for 3.3v and 1.8v.

The remaining switch selects between JTAG/SPI and I2C/SWD.  I2C and SWD use a
bidirectional data line, so require special handling.

### Software

I've tested the board with `OpenOCD` for JTAG/SWD use and `flashrom` for SPI/I2C
flash.  I'm planning to make a demo target board with a handful of interfaces on
it and provide that here in the future.

With `OpenOCD`, use the configuration file in `configs`.  You will also need a
target configuration.  Time permitting, I will post a tutorial on the use of
TIMEP and OpenOCD to debug targets.

With `flashrom`:

```
flashrom -p ft2232_spi:type=2232H,port=B,divisor=4
```

Use of `port=B` is critical, port `A` is wired to the UART, so will not work as
you intend.  :)

### Protections

Some efforts have been made to protect against mistakes that will destroy your
hardware, but there is still some risk in using this if connected improperly.

Included protection in current revision:

* Current limiting resistors on I/O lines.
* 5v+ tolerant I/O.
* Warnings in the README.  :)

Shorting power to ground may damage the target, the TIMEP interface, or the USB
port on your computer (if running in 5V mode).  This is unlikely, but be careful
with your connections.

Using the wrong voltage may damage components not rated for that voltage.  Any
voltage up to 5v is tolerated on the interface board I/O pins, but beyond that
may result in damage.

Additional protection that could be considered:

* Polyfuses on power/ground rails to computer.
* Clamping diodes on USB lines.
* Clamping diodes on I/O lines.

These have not been added because I have been trying to keep BOM cost down to
get these in the hands of as many hardware hackers as possible.

## License

Copyright 2020 Google, LLC

Maintainer: David Tomaschik <david@systemoverlord.com>

Please see [CONTRIBUTORS.md](CONTRIBUTORS.md) for credit to all contributors.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

This is not an officially supported Google product.

# Minnow - USB-C DAM Tool

![./media/minnow-pcb.jpg](./media/minnow-pcb.jpg) # NEW Image soon!

DAMLink is a tool for using DAM (Debug Accessory Mode), providing an interface to SWD or JTAG and/or Serial interface from the device. It expands upon [this concept](https://github.com/BitterAndReal/SWD-over-USB-C) to include a USB-UART and some utility for use within test rigs. This fork bases on the Minnow project, with some improvments from my side and changes to fit into our application. Above all, the extension of the use of the DAM, in that a signal can be routed via all pins, e.g. SPI / I2C / USART, offers the advantage in the application that you have the possibility to develop and debug USB accessories by connecting a logic analyzer. 

* Enables and interfaces USB DAM configured in image below; Full SWD or JTAG over USB-C.
* Provides board designer the option of using RX+ for NRST/RXD,  RX- for SWO/TXD, TX+ - for SWDIO/IO0 and TX- - for SWCLK/IO1 - for full SWD debugging or Serial interface analysis.
* Four configurable GPIO on FT230 for test rig control of UUT: power enable; RX pin control; reset.
* TagConnect TC2050 and STDC14 header to debugger (ST-Link V3, Segger J-Link or equal).
* USB pass-through or FT230 USB UART to device.
* VTARGET reference from device or external.
* Maintains USB-C rotational symmetry.
* Alternate Mode 3.1+ can still be used in the main application with normal hosts.

![./media/swd_dam_pinout.png](./media/swd_dam_pinout.png)
[Original reference](https://github.com/BitterAndReal/SWD-over-USB-C/blob/main/images/SWD%20over%20USB-C%20Pinout-01.png) modified to include option of UART.

# Status and Obtaining

[The latest release](https://github.com/tuna-f1sh/minnow/releases/latest) is full tested and I have PCBAs for purchase:

* [Purchase from my shop](https://shop.jbrengineering.co.uk/product/minnow-usb-c-debug-accessory-mode-tool-dam/).
* [Purchase from Tindie](https://www.tindie.com/products/jbrengineering/minnow-usb-c-debug-accessory-mode-tool-dam/).
* Since it's a development tool, it uses 0603 so can be easily assembled/modified by hand.

# Usage

![Minnow R2 connected to example DAM board](./media/minnow-dam-board.jpg)
__Minnow connected to the USB DAM example test board - this would be integrated to a target in practice__

Here is a quick video showing basic usage: https://youtu.be/QQiKsJ13bL0

**Note** the target device must be designed to enable DAM mode following the USB specification and Minnow DAM USB-C connections. I have made an example board that does this, see below.

## Cable

If a cable is required between Minnow and the device (TS), the cable needs to be a **complete USB-C extension cable** with all [Alternate Mode](https://en.wikipedia.org/wiki/USB-C#Alternate_Mode) wires. USB 3.1+, DisplayPort, Thunderbolt and HDMI rated cables _should_ include these. Here are a couple: [usb-c-extension-cable-for-raspberry-pi-4](https://thepihut.com/products/usb-c-extension-cable-for-raspberry-pi-4) or [Tripp Lite U421-20N-G2](https://www.digikey.ch/en/products/detail/tripp-lite/U421-20N-G2/16161593).

USB-C extension cables are not [technically specification compliant](https://hackaday.com/2022/12/27/all-about-usb-c-illegal-adapters/) but one is required for DAM because no compliant male-male cable will supply both CC1 and CC2 between the the DTS and TS. See [USB Type-C](https://www.usb.org/sites/default/files/USB%20Type-C%20Spec%20R2.0%20-%20August%202019.pdf): B.2.3.1.

![highlighted USB-C Spec B.2.3.1](./media/usb-c-dam-cable.png)

I learnt this the hard way: R1 Minnow had a receptacle for both the host and device and no cable worked to enable USB DAM using the CC pull-ups with logic AND. As the specification says, this is why a direct plug is required for USB DAM mode and why Minnow R2 now has one.

The cable to the host can be a USB 2.0.

## Example Device 

I've included an example design for the device end: './example-dev/usb-dam.kicad\_pro' [pdf](./example-dev/usb-dam.pdf). It can be used for testing and as a foundation for a project with DAM. There are clearly alternative design choices that can be made based on the requirements of the device but it is a good starting point. The layout was done in haste as a means to test the Minnow board!

Essentially the target must detect when the CC lines are pulled up with the values specified in the USB specification (500mA @5V 10kΩ on CC1 and 22kΩ on CC2). Because the CC lines might also have 5k1 pull-downs, a potential divider is created:

* CC1: Host 22kΩ pull-up to 5V and device 5.1kΩ pull-down = 0.9V
* CC2: Host 10kΩ pull-up to 5V and device 5.1kΩ pull-down = 1.7V

The example board uses two op-amps configured as non-inverting amplifiers to buffer this to a logic AND, which enables a 4-channel switch to connect the debug lines to the Alternate Mode pins.

## FAQ

### What's the default configuration? 

USB host to device; device powered; full SWD on debug headers; VCCIO from VTARGET.

### VCCIO selection

By default, VCCIO for the FT230 is supplied from VTARGET (< 10 uA current demand from logic gates) to ensure the same logic level as the target. VCCIO controls the logic level for the GPIO and UART. Cut JP5 1-2 and link 2-3 to use JP6 source - default 3V3 1-2, 2-3 VEXT for external source (1V8 for example). **If using a VCCIO source less than 3V3, one should take the LEDs out of circuit by removing R1 and R2 as they are pulled to 3V3.**

### Why aren't the UART act LEDs blinking?

Logic level is sourced from VCCIO; VTARGET from the device by default. If no device is attached, there is no logic level and so nothing to sink the LEDs or send UART for that matter. Consider this a feature not a bug since they don't blink if nothing is output/listening 🙂. To change this, VCCIO can be self-supplied with JP5 2-3 - see 'VCCIO Selection'.

### Why doesn't the FT230 USB UART work?

One must switch SW1:1 ON to enable USB -> FT230 rather than device. Also be aware that the default configuration VCCIO is supplied from the device and without this there is no logic on the UART/GPIO pins. See 'VCCIO Selection'. It's the reason I didn't include a GPIO to the device power since it would not work without an external VCCIO source.

### How can I control the device power with the FT230

First switch the VCCIO source from VTARGET - see 'VCCIO Selection' - as logic will be required without the device. Add the resistor R15 (680R or anthing to protect GPIO) to link CB3 to the load switch. It's active LOW.

### The device is powered but no DAM pins are working

The cable to the device is probably not a USB 3.1+ (or a bad one) and does not include the alternate mode pins. Use a cable tester to continuity test the cable or find a better one! See 'Cable'.

### How can I configure/use the FT230?

See [ftx-prog](https://github.com/richardeoin/ftx-prog), the main [FTDI utilities page](https://ftdichip.com/utilities/) or the [pyftdi module](https://eblot.github.io/pyftdi/tools.html). The GPIO pins can be changed at runtime or non-volatile with the EEPROM - see './power-gpio.py' as an example of toggling power to the device.

I personally use the 'ftconf.py' script as part of _pyftdi_ to provision new devices using './provision.py'.

# Useful Links

* [Minnow Project](https://github.com/tuna-f1sh/minnow)
* [SWD over USB-C](https://github.com/BitterAndReal/SWD-over-USB-C)
* [USB Type-C](https://www.usb.org/sites/default/files/USB%20Type-C%20Spec%20R2.0%20-%20August%202019.pdf)
* [ARM JTAG/SWD Interface](https://developer.arm.com/documentation/101636/0100/Debug-and-Trace/JTAG-SWD-Interface)
* [FT230X datasheet](https://www.ftdichip.com/Support/Documents/DataSheets/ICs/DS_FT230X.pdf)

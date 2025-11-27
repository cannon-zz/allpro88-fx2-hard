# Cypress FX2 Based Controller for Logical Devices' ALLPRO88 Software-Driven Device Programmer

## Overview

Thanks to the skilled work of kevtris reverse engineering one of these units and his kind generous sharing of his documents, I was able to design and put together this controller to get my own unit going.

kevtris, in addition to needing a control solution, also wasn't satisfied with the squealing power supplies, and so to solve all of the issues at once he devised a fancy card that replaces several of the power supplies and provides an FPGA-based custom CPU design for controlling the unit.

The approach here is different.  This is only a controller, no power supplies are replaced, and the controller is a simple EZ-USB FX2LP based circuit.

You can build your own easily with some pin header socket connectors, a protoboard, three resistors, and one of the "USB logic analyzer" boards you can find on AliExpress.  The "logic analyzer boards", as they're called, are actually just Cypress' FX2LP development board reference design.  The boards get shipped with a logic analyzer firmware which seems to have been ripped off some other product or another, but what's in the EEPROM is irrelevant, it gets replaced with this project's firmware.  Those boards are breakout boards providing access to all of the pins that are needed to implement this circuit.

What's here is the schematic of the circuit to build.  It's also a PCB layout that implements it.  If, instead of using this PCB, you go the route of using a USB logic analyzer board as-is, there is a small difference in the power management that needs to be considered, which will be mentioned below.

For the firmware that goes with this controller, see https://github.com/cannon-zz/allpro88-fx2-soft.

## Circuit Description

### Overview

The FX2LP chip provides an 8051 micro-controller running at a speedy 48 MHz.  The chip includes a USB device interface, which we use to get commands from the host and respond to queries.  It has an I2C master interface used mainly to extract a firmware image at boot from an external EEPROM, but that is available to be used for other stuff if one wishes.  The 56 pin FX2LP version used here also provides 3 8-bit general-purpose I/O ports, and 3 additional I/O pins intended to be used as control lines.

The chip runs at 3.3 V but all of the I/O lines are 5 V tolerant and will tolerate voltages on them even if the chip is off, making interfacing it to legacy 5 V circuitry very simple.

The ALLPRO88's original interface was a 36 pin ribbon cable connecting it to an ISA card.  The 36 lines consist of an 8 bit bi-directional data bus, a 12 bit address bus, three active-low control lines (!READ, !WRITE, and !RESET), and then a whole pile of grounds for signal integrity.  All signals are 5 V TTL logic compatible.

What we do is remove the cable, plug the FX2LP into the ALLPRO88's motherboard where the cable used to be, use two of the 3 control lines to drive the !READ, and !WRITE, lines, one of the 8 bit I/O ports to talk to the data bus, 1.5 of the other two 8 bit I/O ports to talk to the address bus, and finally one of the spare GPIO pins to control the !RESET line.  We route a USB connection to the hole on the ALLPRO88's back panel where the 36 pin connector used to be.  We siphon 5 V from a DIP socket on the ALLPRO88's motherboard, and use a 3.3 V linear regulator to power the FX2LP directly from the ALLPRO88.

### ALLPRO88 Bus Interface

Because of the design of the FX2LP's GPIO scripting system, the data bus must be connected to port B.  The !READ and !WRITE control lines must be connected to the FX2LP's programmable control lines.  Because the !RESET line is never toggled as part of I/O transactions we can connect it to a GPIO port, and use the third programmable control line for a status light that blinks when I/O is occurring.  That leaves the address bus being connected to port A and half of port D.

The FX2LP is 5 V tolerant, so no level shifting circuitry is required.

The ALLPRO88's motherboard has a pair of resistor array chips acting as bus terminators for the 36 conductor interface cable.  They're two orange DIP packages next to the pin header for the cable.  These need to be removed from the motherboard.  At one time I believed they put too great a load on the FX2LP's I/O pins.  I'm not sure about that anymore, but what is definitely true is that (i) they are not needed because there is no bus to terminate, only a few centimetres of PCB traces connect the FX2LP to the chips on the motherboard;  and (ii) they're too close to the pin header onto which the interface board mounts and they interfere with the board if they're left installed.

![Bus termination resistors interfering with controller PCB.](photos/resistor_arrays_need_to_go.jpg)

### Power

5 V and GND are available on the usual pins of those DIP resistor array packages, so after removing them that provides a convenient source of power for the interface board.  I suggest soldering a pair of wires into the appropriate holes and running them to a connector.  Powering the FX2 from the ALLPRO88 and not from the USB bus is necessary because doing so ensures the FX2's firmware runs through the correct hardware reset sequence for the ALLPRO88 when the system is turned on.  Without at least the !RESET line being held in the asserted state, the ALLPRO88 powers up with its control registers set to random values and it can damage itself.

![Recommended power connection](photos/power_siphon.jpg)

If you choose to build this using a USB logic analyzer board as-is, those boards are powered by the USB bus and some modifications are needed to power them from the ALLPRO88.  In every example I've seen, those boards include a surface-mount fuse in series with a through-hole push-button switch that feed the USB connector's VBUS pin (+5 V) to the input of a 3-pin 3.3 V linear regulator.  Remove the through-hole switch, and that gives you some empty through-holes to which jumper wires can be soldered.  The hole that was the VBUS side of the switch needs to be wired to the WAKEUP pin on the development board's headers through a resistor and capacitor network as in this circuit.  The hole that was the other of the switch, the side connected to the 3.3 V regulator's input, can be where a wire from the ALPPRO88's 5 V supply gets soldered.  If you're lucky, your board's switch will have been mounted on the same 2.54 mm pitch layout grid as the pin headers, and it's holes will be aligned with holes on the protoboard to which you're attaching it, allowing pins and pin sockets to be used to conveniently jumper the holes the switch used to be in to the protoboard and do all this wiring there.

## Assembly

![Top view of partially assembled PCB](photos/controller_1.jpg)

### Parts Source

If you have a hot air soldering station, the cheapest way to build one of these is to order a USB logic analyzer and salvage the parts from it.  Almost all of the required parts can be obtained from one of those boards, they can be had for well under $10 delivered, whereas the FX2LP part, alone, is more than $20 from Mouser (with delivery extra).  Without a hot air station removing the parts without damaging them will be very difficult.  In that case just buy them all new.  (the price difference doesn't come close to justifying buying a hot air station, but if you were thinking of getting one anyway maybe this is the time).

### Soldering Tips

Install the FX2LP first.  Hand-soldering it requires patience, and the more space you have to work the more patient you'll be, so it's best if there's not yet other stuff on the board.  Get a magnifying glass and check each pin of the chip one-by-one to make sure solder has flowed onto it and onto the pad, and hasn't bridged between pins.  Solder flux is your friend.  Everything else is easy to install.

### ALLPRO88 Connection

Two footprints are provided for a female pin header connector for the connection to the ALLPRO88 motherboard.  One option is a TE Connectivity 532956-6.  This is not cheap, but matches the style of the connectors used on the other boards in the system.  An alternative footprint is available for a standard, cheaper, right-angle female header connector as you'd find when doing a search for such things.

![Controller mounted](photos/controller_2.jpg)

### USB Connection

There are two options for the USB connection to the PCB:  a footprint for a USB connector, and a footprint for soldered-in USB cable.  This is not the connection that the cable from the host PC makes, this is for an internal connection within the ALLPRO88 from the interface board to a USB connector on the rear panel (and that's where the cable from the host connects to).  The directly-soldered-cable option is less likely to become a point of failure in the future, but it means the interface board cannot be removed from the chassis without unscrewing the USB connector plate from the rear panel.  That can become inconvenient.  I initially soldered a cable directly to the board, then later decided I preferred having a connector I can pull a cable out of.

For the rear panel, I bought a USB 2.0 type B break-out board.  The shell of a USB 2.0 type B connector is almost exactly the height of the shell of the DB36 connector removed from the rear panel, and fits nicely into the space left by the connector.  I prefer the larger USB connectors, they're more resilient.  We're not sending this stupid thing into space, use a big connector.  To install it in the hole, I bought a piece of 90 degree aluminum angle extrusion at a local hobby shop.  I cut it to a suitable length with a hack saw, and then used a drill, a small file, and lots of patience to carve out holes and notches for the USB connector and its break-out board.  I reused the original screws from the DB36 connector to mount the aluminum angle into the ALLPRO88 case.

![Rear panel USB connector assembly detail](photos/rear_panel_usb_port_1.jpg)
![Rear panel USB connector assembly detail](photos/rear_panel_usb_port_2.jpg)
![Rear panel USB connector assembly detail](photos/rear_panel_usb_port_3.jpg)
![Rear panel USB connector assembly detail](photos/rear_panel_usb_port_4.jpg)
![Rear panel USB connector assembly detail](photos/rear_panel_usb_port_5.jpg)

Just a little blue paint, and it would look like it was always that way.

### Other Connections

I2C and GPIO headers.  These can be left unpopulated.  It's hard to find connectors that fit into the pin pitch I chose.  That's my bad.  They're not used for anything, they're just there in case somebody wants to play with them in the future.  There are three spare GPIO lines on port D that get routed to J8, along with 3.3 V and GND.  The FX2's I2C clock and data lines, and 3.3 V and GND, are routed to J1.  The ALLPRO88's socket module connectors have many unused pins on them, and one could jumper these signals from the controller board to unused pins on the socket module connectors on the ALLPRO88's motherboard, thereby making them available to custom socket modules.  A little I2C display could be added to show status messages, for example.

## Smoke Test and Firmware Upload

DO NOT connect the module to the ALLPRO88 before writing the proper firmware to the EEPROM.  Apply power to the board and see if it looks OK.  If you don't have a bench power supply for this, find an old 5 V USB wall wart charger and sacrifice it.  Don't try powering the board from the ALLPRO88 because, as mentioned above, the ALLPRO88 needs a hardware reset at start-up otherwise it can damage itself, and for that it needs this controller to be installed.  If the board hasn't caught fire, and the 3.3 V test point shows the correct voltage, plug it into your computer with a USB cable and see what it shows up as.  If you salvaged parts from a USB logic analyzer board, or you are using one of those boards as is, it should show up as that, otherwise it'll probably show up as an FX2 chip in its unprogrammed state.  Either way that tells you it's working.  Write the firmware to the EEPROM, then power cycle the board and it should report itself as an ALLPRO88 USB interface.

NOW you can connect it into the ALLPRO88's motherboard.  With the controller installed, turn on the ALLPRO88.  The "IDLE" light on the socket module should be on, and the "BUSY" light on the socket module should briefly blink.  When the controller is turned on, it enables the main power supply and runs through a channel scan to determine which pin driver modules are installed.  That process is the BUSY light blinking.  If you don't see the light blink once, briefly, or the IDLE light is not on, probably something's wrong.

![Controller installed](photos/controller_3.jpg)

If, when you apply power, the BUSY light comes on and stays on, turn off the ALLPRO88.  That almost certainly indicates the !RESET line is not being driven properly.  That needs to be diagnosed and fixed before moving on.  When its interface bus is floating, the ALLPRO88 powers up with the main power supplies active.  I don't know why, it doesn't seem wise, it can potentially damage the circuitry, but that's the design.  !RESET must be asserted to clear that state.  The controller circuit has a pull-down resistor on !RESET to ensure it's held in the asserted state until the FX2 firmware has booted and can take control, so if your BUSY light is on and won't turn off it means that the pull-down resistor is not doing what it's supposed to do and also that the FX2 firmware is not starting and doing what it needs to do.  Both are serious problems that need to be rectified.

## Service Suggestions

If you are in the process of getting an ALLPRO88 up and running, whether using this controller to do so or some other trick, here are some hints for things to check before closing it up.

 * Check the heat sinks in the power supplies.  The heatsink for the FET in the VADJ power supply in one of my units was not mounted properly, the bolt missed the hole and the heatsink was just sort of sqeezed there between the transistor and the PCB and was sort of free to wiggle around.  It was sufficiently out of place that it was shorting one of the transistor's pins to a pin on a neighbouring component.  Luckily those two are actually connected in the circuit but who knows what mischief it had gotten up to in the past.

 * The electrolytic capacitors in the power supplies should be replaced.  They predate the age of the capacitor plague, so you won't find a disaster waiting for you, but they're old and doing duty in switching power supplies.  Most of mine had begun to leak stuff onto the PCBs, but nothing serious yet.

 * When replacing them, note that three of the original capacitors are not suitable parts, and need to be replaced with capacitors having different ratings:

   * The UNREG power supply's C7, an electrolytic mounted between the input connector from the transformer and the switching regulator, is an "Elite" brand (?) 470 uF 25 V capacitor.  This capacitor is in parallel with a bunch of other capacitors all rated for 50 V and, yes, this part of the circuit can be expected to see voltages over 25 V ... so ... I don't know what's going on.  The schematic clearly shows that this *is* the capacitor that should be there, and shows that its ratings don't match those of its partners in that circuit.  This capacitor is obviously different from everything else in the power supply, and my guess is that they wanted a lower ESR for this capacitor right next to the regulator and after discussing with the manufacturer decided that they could use this specific capacitor in this off-label application.  Don't try it.  I think any modern, quality, low ESR capacitor will significantly outperform the original part, anyway, so just get a good capacitor rated for the same voltage as the others.

   * There are two electrolytic capacitors providing input and output filtering for an LM79M05CT linear regulator on the ANALOGUE 1 board.  The input capacitor is 2.2 uF and the output capacitor is 1 uF, both rated for 50 V.  If you check the datasheet for that regulator, you'll see that it does recommend those capacities but it specifically says they *must* be tantalums.  If electrolytics are used, the capacitances should be at least 20x higher.  The datasheet suggests 25 uF for the output and nothing less than 50 uF for the input if electrolytics are used.  I guess the engineers missed that remark in the datasheet.  Maybe there's enough capacitance elsewhere in the circuit on this regulator's input and output rails that it doesn't really matter, but I went with what the datasheet says.  I had a 6.8 uF solid tantalum in my parts bin that I used for the input capacitor, and a 1 uF multi-layer ceramic chip on hand that I used for the output.

 * You might observe the 5 V rail supplying all the logic chips in the system to be quite low at the far end of the unit, farthest from the 5 V power supply.  In mine I think I was observing 4.8 V or something on the last channel driver module.  Still in spec for TTL chips, but only just barely.  I recommend replacing the voltage feed-back sense resistors in the 5 V supply to increase its output voltage slightly.  I installed a 1.8 kOhm + 5.6 kOhm pair, both 1% tolerance.  That adds about 0.1 V, it sets the regulator's nominal output to 5.15 V (I measured 5.14 V in mine), and that gives me voltages on the logic chips ranging from 5.1 V closest to the power supply to 4.9 V farthest.

 * If you care, the efficiency of the switching power supplies can be significantly improved, reducing the heat produced in the unit, simply by installing new diodes and transistors.  I damaged the -9 V supply in one of my units, and learned about the improvements when picking out replacement parts.  The -9 V supply, as well as some others, use 1N5822 diodes and RFP12P08 P-channel FETs.  I have found that modern diodes labelled 1N5822 are probably something else, and have *much* lower forward bias voltages than the limit given in the datasheet.  Likewise, a TJ11A10M3 FET can replacement for an RFP12P08 and has only 1/3 the on resistance, dissipating 3x less power in itself than the original transistor.  The latter were available locally but are already obsolete parts, so probably whatever newer thing you can find will be even better.

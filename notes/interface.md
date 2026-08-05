# Comments on FX2 <--> ALLPRO 88 Bus Interface Electrical Characterisits

the address and control lines are wired into the inputs of SN74LS244N bus
driver chips, and the data bus into an SN74LS245N bi-directional bus
driver.  those chips are guaranteed to recognize anything over 2 V as a
logic high level, so they should provide the required level shifting from
the FX2's 3.3 V logic outputs to 5 V logic inside the programmer.  the
FX2's documentation says it has 5 V tolerant inputs, so the 5 V output of
the 245N on the data bus during read operations is acceptable.  no level
shifting is required to connect the FX2 directly to the ALLPRO88's
interface.  from the ALLPRO88's service manual, the /RD line controls the
direction of the 245N, so be careful not to pull /RD low while driving the
data bus.

NOTE:  I measure 200 Ohm between every I/O line and both +5 V and GND
inside the programmer.  I don't understand this.  there are Vishay MDP1605
331/471G resistor arrays on the board beside the ribbon cable pin header
which I assume are terminating the cable.  they should have 330 Ohm / 470
Ohm 2% resistors in them, one to +5 V and one to GND.  I'm fairly sure the
330 Ohm resistor goes to +5 and the 470 Ohm to GND, but I don't know that
for a fact.  Neither should be only 200 Ohm, that's for sure.  in any case,
there are termination resistors to both the positive supply rail and
ground, so regardless of what the values are there are a number of
consequences:

1.  with the FX2 chip powered down, all I/O lines should be pulled to
approximately 2.5 V by these resistors.  I had previously believed that
this is a problem for the FX2, because I got the impression from something
that its inputs must not be driven when the chip is powered off, i.e., they
must not be raised to a potential above the supply voltage.  however, more
recently I've rechecked the documentation and I don't see that restriction,
and anyway that would make them not 5 V tolerant if it was true (the chip
runs on 3.3 V).  the only restriction is that the GPIO lines must not have
more than 5 V to ground placed on them, but there is no statement about the
chip being powered when this happens.  I now believe it is safe to power
the programmer before powering the FX2 chip.

2.  when an FX2 output pin is pulled low, assuming there is a 330 Ohm
resistor between it and a +5 V rail, then 15 mA of current will flow.  when
pulled high, to about 3 V, much less current will flow, about 0.3 mA
assuming a 470 Ohm resistor from that node to GND and the 330 Ohm to +5 V.
the chip can only source or sink a maximum of 4 mA on any GPIO pin, so it
should have no trouble pulling the programmer's inputs to logic high but
will not be able to pull them to logic low levels.  it might even damage it
to try to hold a pin low.  to work with an unmodified ALLPRO88 programmer,
buffer circuits will be needed.  alternatively, the termination resistors
could be removed from the ALLPRO's motherboard altogether, maybe replaced
with something comfortably above 1.3 kOhm.  the Vishay datasheet says there
are resistor arrays in all kinds of values, but neither digikey, nor
mouser, nor marutsu sells the MDP1605 configuration in higher than a 680
Ohm / 680 Ohm variant, which would still not be high enough (would need to
sink 7 mA to pull a pin low).

I removed the termination resistors from my unit.  I replaced them with
sockets, so they can be re-installed or removed again easily.  they are not
needed in my case, anyway, because I have connected the FX2 board directly
to the pin header on the ALLPRO's motherboard, inside the unit, so there's
no ribbon cable inductance or capacitance, and only a short physical signal
path between the FX2's GPIO pins and the ALLPRO's bus interface chips.
this also means I don't have to worry about the order in which I apply
power to things.  the 74LS series bus driver chips tolerate normal input
voltages even without power supplied to the chips, so the FX2 can be
powered on and driving the ALLPRO's inputs without damaging them even when
the ALLPRO is powered off.

ultimately, I ended up using one of the empty resistor pack sockets to get
+5 V and GND from the motherboard into a custom FX2 based controller board,
which gave me another reason to want the resistor arrays removed.
therefore, I've never tried putting them back in to see if actually an
unmodified system would work.

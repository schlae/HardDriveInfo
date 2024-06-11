# Hard Drive Repair Information

There's not a whole lot here right now. I'll add more information as I
collect it.

| Manufacturer | Model     | Control PCB | Stepper Type | Avg Pos Time | Media        |
|--------------|-----------|-------------|--------------|--------------|--------------|
| Seagate      | ST-412    | ASSY 20201  | 4-Wire       |         85ms | Oxide, 00253 |
| Seagate      | ST-419    | ASSY 20225  | 4-Wire       |         85ms | Oxide, 00252 |
| Seagate      | ST-225    | ASSY 20301  | 4-Wire       |         65ms | Oxide Coated |
| Seagate      | ST-238    | ASSY 20527  | 4-Wire       |         65ms | Oxide Coated |
| Seagate      | ST-238R   | ASSY 20527  | 4-Wire       |         65ms | Oxide Coated |
| Seagate      | ST-225    | ASSY 20301  | 4-Wire       |         65ms | Oxide Coated |
| Seagate      | ST-251    | ASSY 20629  | 10-Wire      |         40ms | Thin Film    |
| Seagate      | ST-251-1  | ASSY 20938  | 10-Wire      |         30ms | Thin Film    |
| Seagate      | ST-277R-1 | ASSY 21020, 20938-300 | 10-Wire      |         28ms | Thin Film    |

## Driver boards

### 20629, 20938, 21020 "251 CONTROL"

Unlike the earlier ST-225 and ST-412 generations, drives with this board use 5-phase steppers and can seek nearly twice as fast as the ST-412. A lot of discrete logic and analog circuitry has been integrated into several LSI chips.

The PCB schematic and layout for 20938 are available, see st251/20938. This board is used in the ST-251-1 as well as the ST-277R (-300 version).

The other boards are very similar to each other showing a clear design progression.

20629: Uses L293 H-bridge chips instead of the 11721-501. Firmware is in a separate EPROM chip.

## Chips

| Part Number | Nicknames     | Where Used | Description |
|-------------|---------------|------------|-------------|
| 11721-501   |               | 20938 | Similar to the L293 H-bridge driver |
| 10189-521   | SSI257        | 20629, 20938 | Steering diodes and head preamp, NE592 equivalent. |
| 10188-501   | SSI257.2      | 20629, 20938 | Head center tap driver, write current selector. |
| 11744-502   | RING DETECTOR | 20629, 20938 | Stepper motor seek settling chip - adaptive ringout. |
| 11695-502   |               | 20629 | Unknown |
| 10223-502   | RETURN        | 20629, 20938 | Retracts stepper motor on power-down. |
| 11743-501   |               | 20938 | Controls H-Bridge enables based on phase state. |
| 11743-521   |               | 20938-300 | Controls H-Bridge enables based on phase state. |
| 11782-501   | STEP SEQR     | 20938 | 5-Phase stepper motor controller. |
| 11791       |               | 20938 | Spindle motor driver. Similar to the HA13406W. |
| 10206-501   | SSI296        | 20629, 20938 | Read channel and write amplifier. |
| 11647-502   | STEP LOGIC    | 20629, 20938 | MFM interface logic. |
| 80118-502   | R1512-12      | 20938 | Rockwell R6518(?) 6502-core microcontroller. |

### 10223-502 "RETURN"

This device moves the heads to the landing zone when it detects a power loss.

**Pin Description**

| Pin | Name  | Direction | Description                                     |
|-----|-------|-----------|-------------------------------------------------|
| 15  | MOTA+ | 3-state   | Stepper motor winding driver. Only drives high. |
| 3   | MOTD+ | 3-state   | Stepper motor winding driver. Only drives high. |
| 1   | MOTE+ | 3-state   | Stepper motor winding driver. Only drives high. |
| 14  | MOTA- | 3-state   | Stepper motor winding driver. Only drives low.  |
| 2   | MOTD- | 3-state   | Stepper motor winding driver. Only drives low.  |
| 16  | MOTE- | 3-state   | Stepper motor winding driver. Only drives low.  |
| 6   | CLK   | Input     | Step clock input. Comparator driven, can be tied directly to spindle motor coil. |
| 5   | VCC   | Power In  | Main chip power supply. Connect to 12V reservoir capacitors to maintain power during power failure. |
| 4   | VC    | Power In  | Stepper high-side drive supply. Connect to VCC through schottky diode. |
| 10  | VREF  | Output    | Voltage reference output, about 2.625V. |
| 11  | 5V    | Input     | 5V rail monitoring input. Monitored with an undervoltage comparator (3.33V falling, 4.2V rising threshold). |
| 12  | 12V   | Input     | 12V rail monitoring input. Monitored with an undervoltage comparator (8.05V falling, 10.1V rising threshold). |
| 9   | RETRACT# | Output | Active low retraction output signal. Asserts low when either 5V or 12V supply fall below their respective thresholds. Push pull relative to 5V (pin 11) |
| 8   | CT    |           | Timer capacitor connection. Sets the time delay between power failure and spindle motor braking. |
| 7   | DRV   | Open drain | Spindle brake MOSFET driver output. Normally pulled low, this pin goes high impedance when power fails and the timer expires. |
| 13  | GND   | Ground | Connect to ground. |

**Functional Description**

This device monitors the 5V and 12V input pins. When either pin falls below their respective undervoltage thresholds, this begins retraction mode.
The retract pin asserts low and the motor winding drivers activate, driving either high, low, or high impedance depending on the 5-phase step table.
The step speed is controlled by the clock input, typically tied to one of the coils on the spindle motor. The input has a 2-level comparator,
requiring a signal amplitude that falls below 2.5V and rises above 3.6V.

When retraction mode is entered, the CT pin gets driven with a 10uA current source so the capacitor begins to charge. Once it reaches 2.44V, the MOSFET brake driver pin
goes from the low state to the high impedance state. External pullup circuitry then drives the brake MOSFETs to stop the spindle motion.

Calculate the delay time as T = C * 2.44V/10e-6. Using a 6.8uF capacitor, the delay time is about 1.66s. This is the amount of time given to allow the stepper motor
to retract the heads up against the hard stop before the spindle brake is activated.

The step sequence is as follows:

| Phase | 1   | 2   | 3   | 4   | 5   |
|-------|-----|-----|-----|-----|-----|
| MOTA+ | z   | z   | z   | H   | H   |
| MOTA- | z   | z   | z   | L   | L   |
| MOTD+ | z   | H   | H   | z   | z   |
| MOTD- | z   | L   | L   | z   | z   |
| MOTE+ | H   | H   | z   | z   | H   |
| MOTE- | L   | L   | z   | z   | L   |

This is designed for a 5-phase stepper motor (10 wire) but it does not drive phases B and C.


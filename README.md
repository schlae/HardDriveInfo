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

### 11647-502 "STEP LOGIC"

This device interfaces with the MFM control cable.

**Pin Description**

| Pin | Name    | Direction | Description                                      |
|-----|---------|-----------|--------------------------------------------------|
| 30  |  VCC    | Power In  | Main 5V supply input.                            |
| 13  |  GND    | Ground    | Ground connection.                               |
| 2   | BIAS    |           | Tie through a 10K resistor to ground.            |
| 4   | DRV\_SEL# | Input   | Active low drive select signal from MFM.         |
| 9   | DRV\_SEL\_OUT# | O.C. | Buffered, open collector drive select output. |
| 6   | READY\_IN# | Input | Active low ready signal from MCU. |
| 7   | READY\_OUT# | O.C. | "Drive is ready" signal to MFM bus. Gated by DRV\_SEL#. |
| 5   | TRK0\_IN# | Input | Active low track 0 signal from MCU. Asserts low when current track is 0. |
| 8   | TRK0\_OUT# | O.C. | Track 0 signal to MFM bus. Gated by DRV\_SEL#. |
| 3   | WRT\_FAULT# | Input | Write fault signal from write driver circuitry. |
| 10  | WRT\_FAULT\_OUT# | O.C. | Buffered version of WRT\_FAULT# delayed by 6 clock cycles, to MFM bus. Gated by DRV\_SEL#. |
| 37  | CLK     | Input     | System clock input, nominally 2MHz. |
| 34  | HALL    | Input     | RPM signal from spindle motor driver. |
| 12  | INDEX\_OUT# | O.C. | Index signal to MFM bus. Driven by a one-shot from HALL (but divided by two). About 1213 clock pulses wide. Gated by DRV\_SEL#. |
| 42  | MCU\_INDEX | Output | Same as INDEX\_OUT# but inverted and sent to the MCU. |
| 33  | STEP#   | Input     | Step signal from MFM bus. Causes the track to change by +/-1. |
| 38  | MCUSTEP# | Output   | Step signal to MCU. Same as STEP# but gated by DRV\_SEL#. |
| 32  | DIR#    | Input     | Stepper direction signal from MFM bus. 5V on this pin moves the head towards the spindle. 0V towards the edge of the platter. |
| 41  | LATCHED\_DIR# | Output | Stepper direction signal derived from DIR#, gated by DRV\_SEL#, latched on falling edge of STEP#. |
| 43  | MCU\_SEEK\_DONE# | Input | MCU pulses this line low to indicate it has arrived at the desired track. |
| 11  | SEEK\_COMPLETE# | Output | Seek completed signal to MFM bus. Goes low when MCU\_SEEK\_DONE# pulsed, goes high when STEP#, RESET# or DC\_UNSAFE# pulsed. |
| 31 | DC\_UNSAFE# | Input | Input from power supply voltage monitors. |
| 35 | RESET# | Input | Reset input from MCU. Clears the SEEK\_COMPLETE# latch. |
| 36 | READ\_DATA | Input | Data stream from read channel. |
| 39 | INDEX\_TRACK# | O.C. | Connects to MCU PA2. Detects index MFM data pattern on READ\_DATA. |
| 40 | INDEX\_SYNC | Input | Signal from MCU to reset the HALL divider. |
| 44 | WRT\_OK | Output | Asserts high when it is OK to write. Requires SEEK\_COMPLETE#, READY\_IN#, and DRV\_SEL# to be asserted. |
| 19-29 | No Connect | N.C. | Not internally connected. |
| 1, 14-19 | Unknown |      | Unknown function. Typically not connected. |

**Functional Description**

Several signals are simply buffered to the MFM bus and active only when the drive select line is valid:

* DRV\_SEL\_OUT#
* READY\_OUT#
* TRK0\_OUT#
* WRT\_FAULT\_OUT#
* INDEX\_OUT#
* SEEK\_COMPLETE#

Other signals are returned from the MFM bus only when the drive select line is valid:

* STEP#
* DIR#

The INDEX\_OUT# signal is passed through a one-shot timer run off the HALL input divided by two. On every other HALL pulse, it goes low for about 1213 cycles of the CLK input. This is about 600us. The INDEX\_SYNC# signal, when held low, prevents the INDEX\_OUT# signal from asserting. On the rising edge of INDEX\_SYNC#, the HALL input divider is resynchronized and the first pulse on HALL will generate an index output pulse. The second pulse is skipped, but the third makes it through, and so forth.

The WRT\_FAULT\_OUT# is the WRT\_FAULT# input but delayed by 6 clock cycles.

The stepper direction signal going into the MCU is a latched version of the one present on the MFM bus, latched when STEP is pulsed.

There is an SR latch driving the SEEK\_COMPLETE# signal. This signal deasserts when STEP# asserts, RESET# asserts, or DC\_UNSAFE# asserts. It asserts when the MCU\_SEEK\_DONE# signal asserts.

The INDEX\_TRACK# signal is somewhat complex. The internal circuit isn't well understood but it looks at the data coming from the read channel and detects a special signal recorded on track -2. This is "2F" or 00101111 which occurs, repeating for 4ms, at the index location. The rest of the track contains a 1.75MHz signal which does not trigger this pin. For CLK=2MHz, this pin seems to go low when READ\_DATA falls between 1.958MHz and 2.038MHz.

The INDEX\_TRACK# signal seems to depend on internal counters. If READ\_DATA pulses at least once per CLK high/low, in 9 READ\_DATA pulses, then INDEX\_TRACK# asserts low on the 9th READ\_DATA rising edge. This triggers another one-shot device that ensures that INDEX\_TRACK# continues to be asserted for 5 rising CLK edges after READ\_DATA stops pulsing. DC\_UNSAFE# resets both the READ\_DATA pulse detector and the INDEX\_TRACK# one-shot.


### 10188-501 "SSI257.2"

This device drives the center tap connections of the R/W heads, and also provides two outputs to drive the write current resistor divider.

**Pin Description**

| Pin | Name    | Direction | Description                                      |
|-----|---------|-----------|--------------------------------------------------|
| 18  |  5V     | Power In  | Main 5V supply input.                            |
| 11  |  GND    | Ground    | Ground connection.                               |
| 17  | HDSEL1# | Input     | Head 1 select input. Active low.                 |
| 16  | HDSEL2# | Input     | Head select bit 2 input. Active low.             |
| 15  | HDSEL4# | Input     | Head select bit 3 input. Active low.             |
| 12  | WRT\_FAULT\_IN| Input | Goes high if there is a write fault, which disables all center tap outputs. |
| 20  | VHS     | Input     | Head center tap resistor divider supply. Provides termination voltage. |
| 1   | VCTAP   | Input     | Head center tap active voltage. The selected head's center tap gets driven to this voltage - 0.25V. |
| 9   | HDC1    | Output    | Head 1 center tap output. |
| 8   | HDC2    | Output    | Head 2 center tap output. |
| 7   | HDC3    | Output    | Head 3 center tap output. |
| 6   | HDC4    | Output    | Head 4 center tap output. |
| 5   | HDC5    | Output    | Head 5 center tap output. |
| 4   | HDC6    | Output    | Head 6 center tap output. |
| 10  |  12V    | Power In  | 12V supply used for write current drive outputs. |
| 2   | RIWR1   | Input     | Write current drive 1 input. |
| 19  | IWR1    | Output    | Write current output drive 1. |
| 13  | RIWR2   | Input     | Write current drive 2 input. |
| 14  | IWR2    | Output    | Write current output drive 2. |

**Functional Description**

The center tap driver chip drives the center tap of each head coil depending on the head select input state. During a write fault
(WRT\_FAULT\_IN = 1) all heads are deselected.

| WRT\_FAULT\_IN  | HDSEL4# | HDSEL2# | HDSEL1# | HDC1 | HDC2 | HDC3 | HDC4 | HDC5 | HDC6 |
|-----------------|---------|---------|---------|------|------|------|------|------|------|
|               0 |       1 |       1 |       1 | VCTAP|   VT |   VT |   VT |   VT |   VT |
|               0 |       1 |       1 |       0 |   VT | VCTAP|   VT |   VT |   VT |   VT |
|               0 |       1 |       0 |       1 |   VT |   VT | VCTAP|   VT |   VT |   VT |
|               0 |       1 |       0 |       0 |   VT |   VT |   VT | VCTAP|   VT |   VT |
|               0 |       0 |       1 |       1 |   VT |   VT |   VT |   VT | VCTAP|   VT |
|               0 |       0 |       1 |       0 |   VT |   VT |   VT |   VT |   VT | VCTAP|
|               1 |       x |       x |       x |   VT |   VT |   VT |   VT |   VT |   VT |

(VT is 0.86 * VHS. VCTAP in the table is actually VCTAP - 0.25V.)

Unselected heads are driven to VHS * 0.86. There may be an internal voltage divider but this is buffered with a low (0.3 ohm) impedance driver before being switched onto the head output.

The voltage provided to VCTAP must be higher than the voltage provided to VHS.

The chip also includes a very simple two-transistor driver for the write current selection. This is used for up to 4-zone recording.

When RIWR1 is driven high, IWR1 is driven to +12V. Otherwise, IWR1 floats.
When RIWR2 is driven high, IWR2 is driven to +12V. Otherwise, IWR2 floats.

### 10189-521 "SSI257"

This is the head select steering matrix and read preamplifier.

**Pin Description**

| Pin | Name    | Direction | Description                                      |
|-----|---------|-----------|--------------------------------------------------|
| 12  |  12V    | Power In  | Main 12V supply input.                           |
| 11  |  GND    | Ground    | Ground connection.  |
| 1   |  HD1+   |           | Connection to head coil 1 (positive). |
| 2   |  HD1-   |           | Connection to head coil 1 (negative). |
| 3   |  HD2+   |           | Connection to head coil 2 (positive). |
| 4   |  HD2-   |           | Connection to head coil 2 (negative). |
| 20  |  HD3+   |           | Connection to head coil 3 (positive). |
| 19  |  HD3-   |           | Connection to head coil 3 (negative). |
| 18  |  HD4+   |           | Connection to head coil 4 (positive). |
| 17  |  HD4-   |           | Connection to head coil 4 (negative). |
| 16  |  HD5+   |           | Connection to head coil 5 (positive). |
| 15  |  HD5-   |           | Connection to head coil 5 (negative). |
| 14  |  HD6+   |           | Connection to head coil 6 (positive). |
| 13  |  HD6-   |           | Connection to head coil 6 (negative). |
| 8   |  DI+    | Input     | Write data input (positive).|
| 7   |  DI-    | Input     | Write data input (negative).|
| 10  |  DO+    | Output    | Read data output (positive).|
| 9   |  DO-    | Output    | Read data output (negative).|
| 6   | GAIN+   |           | Gain connection. |
| 5   | GAIN-   |           | Gain connection. |

**Functional Description**

The currently selected read/write head has the common (center tap) winding brought to a voltage higher than the center tap windings
of all the other heads. This forward biases internal steering diodes, while reverse biasing the internal diodes connected to deselected heads.

A differential signal placed on the DI inputs appears on the connection of the currently selected head. This is used to write track data. Leave the inputs high to avoid writing data--an output only sinks current when a DI input sinks current.

Differential signals from the currently selected read/write head gets amplified by the internal read preamp and driven out to the DO outputs. A gain network (R, C, or RLC) can be attached to the two GAIN pins. Lower impedances increase the gain, as shown approximately in this table.

|  R  | Gain |
|-----|------|
| 10K | 24   |
|  1K | 28   |
| 100 | 40   |

The preamp is similar to the NE592 video amplifier chip. See the datasheet for examples of different gain networks.

### 11782-501 "STEP SEQR"

The step sequencer generates the 5-phase signals that drive the stepper motor power driver chips.

**Pin Description**

| Pin | Name  | Direction | Description                                     |
|-----|-------|-----------|-------------------------------------------------|
| 13, 20 | VCC | Power In | 5V power rail. |
| 10, 19 | GND | Ground   | Connect to ground. |
| 1   | MTA+  | Output    | Motor phase A high drive |
| 2   | MTA-  | Output    | Motor phase A low drive |
| 4   | MTB+  | Output    | Motor phase B high drive |
| 5   | MTB-  | Output    | Motor phase B low drive |
| 6   | MTC+  | Output    | Motor phase C high drive |
| 7   | MTC-  | Output    | Motor phase C low drive |
| 8   | MTD+  | Output    | Motor phase D high drive |
| 9   | MTD-  | Output    | Motor phase D low drive |
| 11  | MTE+  | Output    | Motor phase E high drive |
| 12  | MTE-  | Output    | Motor phase E low drive |
| 14  | RESET# | Input    | Active low reset input. Drive low to go to sequence 0. |
| 15  | CLK   | Input     | Clock input. Sequence changes on rising edge. |
| 3   | RETRACT# | Input  | Active low retract input. Drive low to set all phase outputs low. |
| 17  | DIRECTION | Input | Direction pin. The sequence tables all assume this pin is high. |
| 16  | HALFSTEP# | Input | Active low half-step input. Activating this pin turns on half-step mode. |
| 18  | HALFSTEP2# | Input | Active low alternative half-step mode input. When asserted while HALFSTEP# is low, turns on alternate half-step mode. Ignored when HALFSTEP# is deasserted.|

**Detailed Description**

This device is basically a sequence controller chip that drives the phase signals going to the stepper motor driver chips.

They follow these phase tables.

For each phase, the states below correspond with the following output states. Phase 0 corresponds with the phase after a reset. When DIRECTION=0, proceed from phase 0, 1, 2...9 and wrap back to 0. When DIRECTION=1, proceed from phase 0, 9, 8, 7...1.

| Phase | + Pin State | - Pin State |
|-------|-------------|-------------|
|   z   |         Low |        Low  |
|   H   |        High |        Low  |
|   L   |         Low |       High  |


Std: Standard step mode (HALFSTEP#=1, HALFSTEP2#=x, DIRECTION=0)

Half: Half step mode (HALFSTEP#=0, HALFSTEP2#=1, DIRECTION=0)

AltHalf: Alternate half step mode (HALFSTEP#=0, HALFSTEP2#=0, DIRECTION=0)

| Std   | 0 | . | 1 | . | 2 | . | 3 | . | 4 | . | 5 | . | 6 | . | 7 | . | 8 | . | 9 | . |
|-------|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|    A  | z |   | L |   | L |   | L |   | L |   | z |   | H |   | H |   | H |   | H |   |
|    B  | L |   | z |   | H |   | H |   | H |   | H |   | z |   | L |   | L |   | L |   |
|    C  | H |   | H |   | z |   | L |   | L |   | L |   | L |   | z |   | H |   | H |   |
|    D  | L |   | L |   | L |   | z |   | H |   | H |   | H |   | H |   | z |   | L |   |
|    E  | H |   | H |   | H |   | H |   | z |   | L |   | L |   | L |   | L |   | z |   |
| **Half** | **0** | **1** | **2** | **3** | **4** | **5** | **6** | **7** | **8** | **9** | **10**| **11**| **12**| **13**| **14**| **15**| **16**| **17**| **18**| **19**|
|    A  | z | z | L | L | L | L | L | L | L | z | z | z | H | H | H | H | H | H | H | z |
|    B  | L | z | z | z | H | H | H | H | H | H | H | z | z | z | L | L | L | L | L | L |
|    C  | H | H | H | z | z | z | L | L | L | L | L | L | L | z | z | z | H | H | H | H |
|    D  | L | L | L | L | L | z | z | z | H | H | H | H | H | H | H | z | z | z | L | L |
|    E  | H | H | H | H | H | H | H | z | z | z | L | L | L | L | L | L | L | z | z | z |
| **AltHalf** | **0** | **1** | **2** | **3** | **4** | **5** | **6** | **7** | **8** | **9** | **10**| **11**| **12**| **13**| **14**| **15**| **16**| **17**| **18**| **19**|
|    A  | z | L | L | L | L | L | L | L | L | L | z | H | H | H | H | H | H | H | H | H |
|    B  | L | L | z | H | H | H | H | H | H | H | H | H | z | L | L | L | L | L | L | L |
|    C  | H | H | H | H | z | L | L | L | L | L | L | L | L | L | z | H | H | H | H | H |
|    D  | L | L | L | L | L | L | z | L | H | H | H | H | H | H | H | H | z | L | L | L |
|    E  | H | H | H | H | H | H | H | H | z | L | L | L | L | L | L | L | L | L | z | H |

### 11743-501 Stepper drive enable control

This device controls the H-bridge enable lines for each phase of the stepper motor.

**Pin Description**

| Pin | Name  | Direction | Description                                     |
|-----|-------|-----------|-------------------------------------------------|
| 28  | VCC   | Power     | +5V power input. |
| 1, 17  | GND | Ground   | Connect to ground. |
| 16  | DIS#  | Input     | Active low disable input. Pull low to make all outputs go low. |
| 15  | ALLON | Input     | Forces all motor phase output enables high. |
| 2   | S1    | Input     | Output mask select mux input. |
| 3   | S2    | Input     | Output mask select mux input. |
| 4   | S3    | Input     | Output mask select mux input. |
| 5   | +A    | Input     | Motor phase A high drive. |
| 6   | -A    | Input     | Motor phase A low drive. |
| 7   | +B    | Input     | Motor phase B high drive. |
| 8   | -B    | Input     | Motor phase B low drive. |
| 9   | +C    | Input     | Motor phase C high drive. |
| 10  | -C    | Input     | Motor phase C low drive. |
| 11  | +D    | Input     | Motor phase D high drive. |
| 12  | -D    | Input     | Motor phase D low drive. |
| 13  | +E    | Input     | Motor phase E high drive. |
| 14  | -E    | Input     | Motor phase E low drive. |
| 26  | EN\_A | Output    | Phase A output enable. |
| 24  | EN\_B | Output    | Phase B output enable. |
| 22  | EN\_C | Output    | Phase C output enable. |
| 20  | EN\_D | Output    | Phase D output enable. |
| 18  | EN\_E | Output    | Phase E output enable. |
| 19, 21, 23, 25, 27 | NC | No connect. | Leave unconnected. |

**Detailed Description**

The power driver chips for the stepper motor phase windings are typically L293 or similar totem-pole devices. That means that each phase connection can be driven high or low depending on the input state, but the output driver can also be disabled so it pulls neither high nor low. However, the step sequencer chip, 11782-501, can only drive both phase outputs (+ and -) low to indicate that the driver should be disabled. If both coil drivers are driven low, the coil is effectively short-circuited and will brake the motor, so this chip detects that and disables both coil drivers.

If the ALLON line is low, then each phase output enable EN\_x will track the corresponding + and - inputs by XORing them together. So if both inputs are low or both are high, then the phase output enable will go low. If one input is high and the other is low, then the phase output enable goes high.

If the ALLON line is high, then all phase output enables go high regardless of the motor phase inputs.

The output mask select mux inputs, S1, S2, and S3, can selectively force individual phase output enables low according to this table:

| S3 | S2 | S1 | EN\_A | EN\_B | EN\_C | EN\_D | EN\_E |
|----|----|----|-------|-------|-------|-------|-------|
|  0 |  0 |  0 |       |       |       |       |       |
|  0 |  0 |  1 |       |       |       |  low  |       |
|  0 |  1 |  0 |       |  low  |       |       |       |
|  0 |  1 |  1 |  low  |  low  |       |       |       |
|  1 |  0 |  0 |  low  |       |       |       |       |
|  1 |  0 |  1 |       |       |       |       |  low  |
|  1 |  1 |  0 |       |       |  low  |       |       |
|  1 |  1 |  1 |  low  |       |  low  |       |       |

Empty boxes follow the state prescribed by the ALLON line and the +/- motor phase inputs.

Typical firmware leaves all three select mux inputs low, but one section brings S1 high, possible for stepper motor voltage calibration.



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


# Hard Drive Repair Information

This site contains extensive information about the control boards used in some 1980s hard drives made by Seagate.

Full schematics and even KiCad layouts are available for a number of different boards. My hope is that this will

1. Aid anyone attempting to repair one of these drives. Loading the schematic and layout in KiCad will let you click on a part on the layout and highlight it in schematic.
2. Inform anyone studying the history of small hard drives. For example, this information shows how Seagate integrated various functions into custom chips.

Below is a table linking specific drive models with the part numbers of their matching control boards.

| Manufacturer | Model     | Interface | Control PCB | Stepper Type | Avg Pos Time | Media        | Media Size OD/ID |
|--------------|-----------|-----------|-------------|--------------|--------------|--------------|------------------|
| Seagate      | ST-412    | MFM       | ASSY 20201  | 4-Wire       |         85ms | Oxide, 00253 | 130/40mm |
| Seagate      | ST-419    | MFM       | ASSY 20225  | 4-Wire       |         85ms | Oxide, 00252 | 130/40mm |
| Seagate      | ST-225    | MFM       | ASSY 20301, 20527 | 4-Wire |         65ms | Oxide Coated | 130/40mm |
| Seagate      | ST-238    | MFM       | ASSY 20527  | 4-Wire       |         65ms | Oxide Coated | 130/40mm |
| Seagate      | ST-238R   | MFM/RLL   | ASSY 20527  | 4-Wire       |         65ms | Oxide Coated | 130/40mm |
| Seagate      | ST-251    | MFM       | ASSY 20629  | 10-Wire      |         40ms | Thin Film    | 130/40mm |
| Seagate      | ST-251-1  | MFM       | ASSY 20938  | 10-Wire      |         30ms | Thin Film    | 130/40mm |
| Seagate      | ST-277R-1 | MFM/RLL   | ASSY 21020, 20938-300 | 10-Wire |    28ms | Thin Film    | 130/40mm |
| Seagate      | ST-296N   | SCSI      | ASSY 20741  | 10-Wire      |         28ms | Thin Film    | 130/40mm |
| Seagate      | ST-125    | MFM       | ASSY 20867  | 6-Wire       |         30ms | Thin Film    | 95/25mm  |
| Seagate      | ST-157R   | MFM/RLL   | ASSY 20829  | 6-Wire       |         30ms | Thin Film    | 95/25mm  |
| Seagate      | ST-157A-1 | AT-IDE    | 20948       |              |         28ms | Thin Film    | 95/25mm  |

## Assembly Information

The control boards are typically held to the drive by three screws. Two of the screws are insulated from the PCB ground with plastic washers, but one of them makes a solid ground connection to the chassis. This is (as far as I can tell) always the screw closest to the power connector.

Early ST-225 boards have an 18-pin header going to the read/write heads, but the flex cable has a 16-pin connector. It should be plugged in offset to the pin 1 end of the header, allowing pins 17 and 18 on the header to hang outside the connector.

## Driver boards

### 20301 "225 CONTROL", 20527

This ST-225 control board integrates the functions of both circuit boards in the ST-412/419 generation. Some functions
have been integrated in LSI chips.

The PCB schematic and layout for 20301 are available, see st225/20301.

The PCB schematic and layout for 20527 are available, see st225/20527.

There are several variations of the 20301 board: 

* 1986 example: 74273, 2716, RP6 are not stuffed.

* 1987 example: 74273, 2716, RP5, RP6, RP7, RP8, RP9, RP10, C18, C31, CR4, 7445(8C, 8B) are not stuffed. Q9 is replaced with a 620 ohm resistor between the emitter and base terminals. This disables the fancy pulldown resistors on the stepper motor circuit (probably a cost reduction).

Unlike the EPROM chip on the ST-251 boards, the 2716 chip seems to be used to store tables for controlling the stepper motor.

**20301 to 20527 changes**

* A better LC power filter has been added to the 12V and 5V power inputs.
* The two 7445 chips driving the stepper motor have been depopulated and all but one resistor array have been removed. Presumably a cost reduction.
* The circuit driving the stepper motor chip's SET pin has been depopulated and jumpered with a resistor, probably a cost reduction.
* The footprint for the 2716 EPROM and the 74273 latch (to support external firmware) have been removed completely.
* The transistor driving the START#/RUN connection to the spindle driver has been swapped with a spare 7406.
* Several discrete resistors in the spindle driver circuit have been incorporated into resistor arrays, including a fully custom array, the 12497-001 which contains a 3.3K, 470, 560, and 2.2K resistor.
* The reference clock circuit driving the spindle motor control IC has changed, presumably to put less of a load on the 2MHz MCU crystal.
* The transistor array and resistor arrays used to drive the read/write head center tap connections (as well as the two write current driver transistors) have been consolidated into LSI 10014-002.
* Read/write channel SSI280 has been expanded and modified (now called 10206-002) with 3 additional one-shot timers in the pulse detector circuit as well as a more complex 2-part final differential signal filter. Compare this new design to the later ST-251.
* New one-shot devices have been added to the MCU's read channel. This read channel allows the MCU to detect the index and guard band tracks recorded at manufacture time. Presumably these changes make the MCU's detection algorithms more tolerant of defects on these tracks. It is not known if drives with these boards have a different pattern recorded on the index and guard band tracks.
* The reset signal pulse, derived from the DC\_UNSAFE# signal comparators in the 10206-002, has been shortened significantly.
* The hall sensor signal to the MCU is generated with a real one-shot chip instead of two chained flip flops with RC filters.

**A note about 20527 to ST-251 boards, like the 20629**

There appears to be a clear evolution between the 20527 ST-225 board and the early ST-251 board, the 20629.

* The read channel is nearly identical, with the preamp and first stage filter being exactly the same. The second stage filter has been adjusted and the one-shots are different.
* Support for two additional read/write heads has been added.
* The spindle motor is now a three-phase BLDC (instead of 2-phase) with three hall effect sensors (instead of one). A single chip 3-phase motor driver has been bolted onto the existing speed control chip.
* The stepper motor is now a really wild 5-phase custom design with a slew of new control chips, including a ring detector LSI that is used to optimize the motor voltage, and an auto-retract chip that takes care of parking the heads during a power-down. All this design effort almost *halves* the average seek time.
* A new LSI incorporates the MCU read channel one-shot devices along with a bunch of discrete buffers.
* The MCU's external ROM chip is back. Presumably this was used extensively for internal firmware development and left in the shipping product until mask ROM MCUs became available.

### 20629, 20938, 21020 "251 CONTROL"

Unlike the earlier ST-225 and ST-412 generations, drives with this board use 5-phase steppers and can seek nearly twice as fast as the ST-412. A lot of discrete logic and analog circuitry has been integrated into several LSI chips.

The PCB schematic and layout for 20629 are available, see st251/20629. This board is used in the ST-251 (plain).

The PCB schematic and layout for 20938 are available, see st251/20938. This board is used in the ST-251-1 as well as the ST-277R (-300 version).

The PCB schematic and layout for 21020 are available, see st251/21020. This board is used in the ST-277R-1.


**20629 to 20938 changes**

Changes to the stepper motor circuits:

* The L293 bridge chips have been replaced with custom drivers (11721-501).
* Another custom chip, the 11743-501, has been added to control the enable state of each stepper phase.
* The DC\_UNSAFE# signal has been split and no longer controls the stepper motor enable. A new RETRACT# signal, generated by the 10223-502 chip's power good monitoring comparators, does this instead. (DC\_UNSAFE# is generated by power good comparators in the 10206-501 read/write circuit.)
* The stepper sequencer output enable (pin 3) is now driven by the RETRACT# signal instead of being pulled up.
* The stepper sequencer has been moved from the regular 5V rail to the spindown keepalive rail (+5VP).
* The stepper driver chips now get 5V from the regular 5V rail instead of a filtered rail (+5F).
* The ring detector LSI has had a jumper installed in series with the reset input (not populated by default)
* 1K series resistors have been added between the stepper windings and the ring detector LSI.
* The digitally-controlled stepper motor voltage rail now has more bulk capacitance (C63, with series resistor for stability).

Changes affecting the spindle motor drive circuit:

* The spindle motor driver, the speed control LSI (11692-502) and the index comparator LM339 have been combined in a single custom spindle driver IC, the 11791.
* The braking bridge rectifier (UC3610) has been removed and two additional MOSFETs have been added for braking.

Changes affecting the read/write circuit:

* The read/write head center tap driver filter now gets power from the regular 5V rail instead of sharing it with the stepper driver chips.
* Write termination resistor R21 has been moved to be near 11E (the receive side) for signal integrity.

Some changes imply changes to the drive firmware. This means that firmware meant for the previous revision will no longer work on this board:

* The MCU has been swapped with a mask ROM chip (80118-502) and the EPROM and address latch (6B) are no longer populated
* The MCU port PA7 is now tied through a resistor (R33) to ground. It no longer drives the stepper motor ALL\_PHASES\_ON signal. Presumably newer firmware can check for this strapping resistor to detect which board revision it's running on.
* The ALL\_PHASES\_ON signal is now driven by bit 7 of the external IO port (the 'LS273).
* The INDEX\_SYNC# signal to sync up the hall state machine is now triggered by the output port 'LS273 bit 0 instead of being triggered by a write to 0x0105.
* A write to 0x0105 now pulses the stepper sequencer 7C RESET# pin (14), instead of it being controlled by bit 0 of the output port 'LS273.
* The MCU port PB4 no longer goes to the debug test points. Instead, it's been routed (along with PB5 and PB6) to the new 11743-501 stepper phase control chip inputs S1, S2, and S3.

**20938 to 21020 changes**

The electrical changes here are fairly minor. The layout has some moderate changes to improve routing and add the (few) new parts.

* The spindle motor driver IC is now the 11738-002 which has a slightly different package.
* The spindle motor IC's reference resistor now has two possible values, set by a transistor driven by a toggle flip flop (1F section A) by writing to address 0x0106. The default is 13K (19.6K || 38.3K) and, when toggled, it is just 38.3K. Presumably this is for power savings during operation by running the spindle motor at a reduced current.
* The MCU has a new part number, 80118-505. Presumably this is new firmware that knows how to reduce the spindle driver power.
* The MCU's clock crystal is now just a single through-hole footprint instead of a dual SMT/PTH one.
* The steering diode IC is now the 10189-502 (not the 10189-521).
* The H-bridge enable controller is now the 11468 instead of the 11743-501.

## Firmware

The firmware from the 20629 board has been dumped. The label is marked "ST 251 / LSIL18". The firmware has been fed through Ghidra and a good chunk of it has been commented and labeled. See [the repository](st251/firmware/ST251_commented.txt). Some parts of it are not understood.

## Chips

| Part Number | Nicknames     | Where Used | Description |
|-------------|---------------|------------|-------------|
| 10014-002   |               | 20527 | Head center tap driver, write current selector. |
| 10188-501   | SSI257.2, V10096BQZ | 20629, 20938, 21020, 20867, 20829, 20948, 20741 | Head center tap driver, write current selector. |
| 10189-521   | SSI257        | 20629, 20938, 20829 | Steering diodes and head preamp, NE592 equivalent. |
| 10189-501   |               | 20867 | Steering diodes and head preamp, NE592 equivalent. |
| 10189-502   |               | 21020, 20741  | Steering diodes and head preamp, NE592 equivalent. |
| SSI 280     |               | 20301 | Read channel and write amplifier. |
| 10206-501   | SSI296        | 20629, 20938, 21020, 20867, 20741 | Read channel and write amplifier. |
| 10206-502   |               | 20829, 20948 | Read channel and write amplifier. |
| 10206-002   |               | 20527 | Read channel and write amplifier. |
| 10223-501   | RETURN        | 20867 | Retracts stepper motor on power-down. |
| 10223-502   | RETURN        | 20629, 20938, 21020, 20829, 20948, 20741 | Retracts stepper motor on power-down. |
| 11468       |               | 21020 | Controls H-Bridge enables based on phase state. |
| 11642-001   |               | 20301, 20527  | Steering diodes and head preamp, NE592 equivalent. See 10189.|
| 11647-502   | STEP LOGIC    | 20629, 20938, 21020, 20867, 20829 | MFM interface logic. |
| 11665-001   |               | 20301, 20527 | Stepper motor control and driver chip (4 phase). |
| 11695-002   |               | 20301, 20257 | Spindle speed control chip. |
| 11695-502   | SPEED CONTROL | 20629 | Spindle speed control chip. |
| 11721-501   |               | 20938, 21020, 20741 | Similar to the L293 H-bridge driver |
| 11738-002   |               | 21020 | Similar to 11791 but in a different package. |
| 11738-502   |               | 20741 | Similar to 11791 but in a different package. |
| 11741-502   |               | 20948, 20741 | SSI 2,7 RLL codec/data separator. Similar to 32D5321. |
| 11743-501   |               | 20938, 20741 | Controls H-Bridge enables based on phase state. |
| 11743-521   |               | 20938-300 | Controls H-Bridge enables based on phase state. |
| 11744-501   | RING DETECTOR | 20867 | Stepper motor seek settling chip - adaptive ringout. |
| 11744-502   | RING DETECTOR | 20629, 20938, 21020, 20741 | Stepper motor seek settling chip - adaptive ringout. |
| 11747-501   |               | 20948, 20741 | Unknown. |
| 11782-501   | STEP SEQR     | 20938, 21020, 20867, 20741 | 5-Phase stepper motor controller. |
| 11789-502   |               | 20829 | Unknown. Maybe a stepper motor controller with built-in driver transistors. |
| 11789-504   |               | 20829, 20948 | Unknown. Maybe a stepper motor controller with built-in driver transistors. |
| 11791       |               | 20938, 20829, 20948 | Spindle motor driver. Similar to the HA13406W. |
| IP3M05AW    |               | 20867, 20829 | Spindle motor driver. Similar to 11791 but in a different package. |
| 12654-501   |               | 20948 | AT-IDE interface. |
| 80007-001   | R10L7-11      | 20301, 20527 | Rockwell R6500 6502-core microcontroller. |
| R6518AJ     | R1113-18      | 20629 | Rockwell R6518 6502-core microcontroller. Requires external ROM. |
| 80118-502   | R1512-12      | 20938 | Rockwell R6518(?) 6502-core microcontroller. |
| 80118-505   | R1818-11      | 21020 | Rockwell R6518(?) 6502-core microcontroller. |
| 80049-501   | SCN8051HCCA44 | 20867 | Intel 8051 microcontroller. |
| 80074-504   | SAB8052A-N    | 20741 | Intel 8052 microcontroller. |
| 80084-508   | SAB8052A-N    | 20829 | Intel 8052 microcontroller. |
| 80127-506   | SCN8052HCCA44 | 20948 | Intel 8052 microcontroller. |

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


### 10188-501 "SSI257.2", 10014-002

These devices drive the center tap connections of the R/W heads, and also provides two outputs to drive the write current resistor divider.

The 10188-501 comes in an SOIC package while the 10014-002 comes in a DIP package.

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

### 10189-521 "SSI257", 11642-001

This is the head select steering matrix and read preamplifier.

The 10189-521 comes in an SOIC package while the 11642-001 comes in a DIP package.

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

The preamp is similar to the NE592 video amplifier chip. See the datasheet for examples of different gain networks. Typically a capacitor is used here to create a differentiator (high pass) function. This aids the peak detection function in the read channel (see the next section.)

### SSI 280 Read channel and write preamplifier

This device incorporates the read channel and the write amplifier circuitry.

**Pin Description**

| Pin | Name | Direction | Description |
|-----|------|-----------|-------------|
| 28  |  12V  | Power In  | 12V power rail. |
| 19  |  5V   | Power In  | 5V power rail. |
| 5, 20 | GND   | Ground    | Connect to ground. |
| 10  | WRT\_OK | Input   | Write OK input. Assert when seek completed, ready, and drive selected. |
| 11  | WRITE\_CUR |      | Write current set point. |
| 12  | WDO+  | Output    | Data out to the head steering matrix (positive) |
| 13  | WDO-  | Output    | Data out to the head steering matrix (negative) |
| 15  | WDATA\_IN+ | Input | Write data in (positive) |
| 16  | WDATA\_IN- | Input | Write data in (negative) |
| 23  | WRT\_GATE# | Input | Active low write enable. Connect to MFM bus. |
| 24  | WRT\_FAULT# | Output | Active low write fault indicator. Asserted if a fault occurs during a write, may deassert during reads. |
| 25  | DC\_UNSAFE# | Output | Active low DC unsafe indicator. Asserts if 5V or 12V rail fall below minimum thresholds. |
| 26  | VHS   |           | Head selected monitoring connection to the head steering matrix. |
| 27  | VCTAP | Output    | Head center tap driver. Driven to 12V during a write and about 5.3V during a read. |
| 1   | DI+   | Input     | Preamp signal input (positive). |
| 2   | DI-   | Input     | Preamp signal input (negative). |
| 3   | -GAIN |           | Preamp gain pin 1. |
| 4   | +GAIN |           | Preamp gain pin 2. |
| 6   | DO+   | Output    | Preamp signal output (positive). |
| 7   | DO-   | Output    | Preamp signal output (negative). |
| 8   | +C    | Input     | Signal comparator input+. |
| 9   | -C    | Input     | Signal comparator input-. |
| 14  | DRV\_SEL | Input  | Active low drive select input. Enables the read channel differential outputs. |
| 21  | +RDO  | O.C.      | Single-ended open-collector read data output. |
| 17  | +RD   | Output    | Digital data output (positive). |
| 18  | -RD   | Output    | Digital data output (negative). |
| 22  | TMG1  |           | One-shot timer RC pin. |

The functionality of this device is similar to the 10206-501 but with a simpler pulse generation section including a single one-shot.

### 10206-501 "SSI296", 10206-002 Read channel and write amplifier

These devices incorporate both the read channel and the write amplifier circuitry.

The 10206-501 comes in the 44-pin PLCC package while the 10206-002 comes in the 40-pin DIP package.

**Pin Description**

| PLCC Pin | DIP Pin | Name  | Direction | Description                                     |
|----------|---------|-------|-----------|-------------------------------------------------|
| 44       | 40      |  12V  | Power In  | 12V power rail. |
| 29       | 27      |  5V   | Power In  | 5V power rail. |
| 5, 30    | 5, 26   | GND   | Ground    | Connect to ground. |
| 28       | 28      | GND   | Ground    | Connect to ground. |
| 15       | 13      | WRT\_OK | Input   | Write OK input. Assert when seek completed, ready, and drive selected. |
| 16       | 14      | WRITE\_CUR |      | Write current set point. |
| 19       | 17      | WDO+  | Output    | Data out to the head steering matrix (positive) |
| 20       | 16      | WDO-  | Output    | Data out to the head steering matrix (negative) |
| 22       | 20      | WDATA\_IN+ | Input | Write data in (positive) |
| 23       | 19      | WDATA\_IN- | Input | Write data in (negative) |
| 37       | 35      | WRT\_GATE# | Input | Active low write enable. Connect to MFM bus. |
| 39       | 36      | WRT\_FAULT# | Output | Active low write fault indicator. Asserted if a fault occurs during a write, may deassert during reads. |
| 40       | 37      | DC\_UNSAFE# | Output | Active low DC unsafe indicator. Asserts if 5V or 12V rail fall below minimum thresholds. |
| 42       | 38      | VHS   |           | Head selected monitoring connection to the head steering matrix. |
| 43       | 39      | VCTAP | Output    | Head center tap driver. Driven to 12V during a write and about 5.3V during a read. |
| 1        | 1       | DI+   | Input     | Preamp signal input (positive). |
| 2        | 2       | DI-   | Input     | Preamp signal input (negative). |
| 3        | 3       | -GAIN |           | Preamp gain pin 1. |
| 4        | 4       | +GAIN |           | Preamp gain pin 2. |
| 8        | 7       | DO+   | Output    | Preamp signal output (positive). |
| 9        | 8       | DO-   | Output    | Preamp signal output (negative). |
| 10       | 9       | +C1   | Input     | Signal comparator input 1+. |
| 11       | 10      | -C1   | Input     | Signal comparator input 1-. |
| 12       | 11      | +C2   | Input     | Signal comparator input 2+. |
| 13       | 12      | -C2   | Input     | Signal comparator input 2-. |
| 21       | 18      | DRV\_SEL | Input  | Active low drive select input. Enables the read channel differential outputs. |
| 24       | 21      | +RDO  | O.C.      | Single-ended open-collector read data output. |
| 25       | 22      | +RD   | Output    | Digital data output (positive). |
| 26       | 23      | -RD   | Output    | Digital data output (negative). |
| 31       | 29      | OA    |           | Digital data output amplitude control. Pull up for max or connect a resistor to ground to set another level. |
| 32       | 30      | TMG4  |           | One-shot timer RC pin. Controls the width of the output pulse on RD for a detected flux transition. |
| 33       | 31      | TMG3  |           | One-shot timer RC pin. Edge delay 2b. Triggered by edge delay 2. |
| 34       | 32      | TMG2  |           | One-shot timer RC pin. Edge delay 2. Triggered by input comparator on pins 12 and 13. |
| 35       | 33      | TMG1  |           | One-shot timer RC pin. Edge delay 1. Triggered by input comparator on pins 10 and 11. |
| 6, 7, 14, 17, 18, 27, 36, 38, 41 | 6, 15, 24, 25, 34 | NC | No connect | Not connected. |

**Detailed Description**

The write drive circuit takes differential digital data from the WDATA\_IN pins and outputs it (with a current set by the WRITE\_CUR pin) on the WDO pins during write mode. Write mode is entered when DC\_UNSAFE# is deasserted, WRT\_OK is asserted, and WRT\_GATE# is asserted (DRV\_SEL# is ignored).

VHS senses when the voltage in the heads exceeds a limit of about 3.5V, and when this happens, WRT\_FAULT# asserts.

VCTAP is driven to 12V in write mode and otherwise goes to 5.3V. It is controlled by WRT\_GATE# alone, for some reason.

The power supply monitor asserts DC\_UNSAFE# when the 5V rail falls below 4.2V (rising threshold is 4.23) or when the 12V rail falls below 9.54V (rising threshold is 9.67V).

The preamp section is very similar to the NE592. A RLC network is typically connected across the gain terminals to provide a bandpass filter characteristic. This shapes the differentiated signal coming from the head preamp (located in the head matrix IC).

Typically the circuit on the drive logic board splits this diffential output signal into two sets of signals with different filters applied. Each set of signals has a differential comparator input that each drives a one-shot timer.

Comparator C1 drives one-shot TMG1. Comparator C2 drives one-shot TMG2, and TMG2 drives TMG3. In order to generate an output pulse, the logic is not well understood but it seems that comparator C1 must trigger after comparator C2 (due to an external phase shift network).

If the conditions are met to generate an output pulse, the fourth one-shot trigger, TMG4, triggers for its fixed pulse width time.

![Raw waveforms of each one-shot pin](photos/10206-501-one-shot.png)

Each one-shot pin works in stages:

1. The pin is pulled up to (mostly) 5V. This discharges the timing capacitor tied between the pin and 5V.
2. The pin is released and allowed to discharge through the external pulldown resistor to 90% of 5V. This triggers the output of the one-shot. By using the narrow threshold (only 10% of the total RC curve) the timing result is approximately linear.
3. Some pins discharge the timing capacitor again to 5V, others hover at the 90% of 5V mark.

The timing value can be approximated as T = 0.1 * R * C.

In the scope shot above, all the pins have had 10K resistors installed and the following capacitor values

* Pin 32: 270pF
* Pin 33: 470pF
* Pin 34: 680pF
* Pin 35: 2200pF

These are much larger than the usual values used by the hard drive, which is typically 47pF, to make the scope shot easier to interpret.

Typical values (on the ST-251) are:

* TMG1: 82ns - comparator C1
* TMG2: 47ns
* TMG3: 69ns - total time for comparator C2 is 116ns
* TMG4: 42ns - output data pulse width


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

### 11744-502 "RING DETECTOR"

This device serves two purposes:

* Provided a digitally-controlled linear regulated power source to the stepper motor
* Detect back-EMF "ringing" cycles on the stepper motor windings

**Pin Description**

| Pin | Name  | Direction | Description                                     |
|-----|-------|-----------|-------------------------------------------------|
| 6   | +12V  | Power     | 12V power input. |
| 13  | GND   | Ground    | Connect to ground. |
| 7   | VREF  | Power     | 2.5V reference voltage input. |
| 4   | CS#   | Input | Active low chip select input. |
| 3   | SET\_CLK | Input | Clock input for SPI shift register. Normally high but latches on the rising edge. |
| 2   | SET\_DATA | Input | Data input for SPI shift register. |
| 1   | SYS\_CLK | Input | System clock input, used for one-shot timing. |
| 8   | DAC\_SET |        | DAC output voltage from 0 to VREF/2. Add a resistor to ground to set the current. |
| 9   | DRV | Output | Drives external pass transistors for linear regulator. |
| 10  | COMP |       | Compensation node for linear regulator. |
| 11  | FB   | Input | Feedback node for linear regulator. |
| 12  | RING\_SET| Input | Ring detector threshold voltage setpoint. |
| 18  | A+   | Input | Motor phase A+ input. |
| 23  | A-   | Input | Motor phase A- input. |
| 17  | B+   | Input | Motor phase B+ input. |
| 22  | B-   | Input | Motor phase B- input. |
| 16  | C+   | Input | Motor phase C+ input. |
| 21  | C-   | Input | Motor phase C- input. |
| 15  | D+   | Input | Motor phase D+ input. |
| 20  | D-   | Input | Motor phase D- input. |
| 14  | E+   | Input | Motor phase E+ input. |
| 19  | E-   | Input | Motor phase E- input. |
| 24  | RING\_DET | O.C. | Ring detector comparator output. |
| 5   | NC   |       | Not connected. |


**Functional Description**

The device has a linear regulator with a 7-bit DAC controlling the voltage. The topology is simple. The output voltage is monitored by an external resistor divider tied to FB and compared directly with the VREF input. The error amplifier output connects to the COMP pin for external compensation purposes. The DRV pin is inverted and connects to an external PNP pass transistor (or an arrangement of an NPN and a PNP).

Communications to the device occurs over a simple 8-bit SPI bus (no data output). Deassert the active-low CS# line and clock data, LSB first, into the device (CLK normally high, rising edge triggered), then release CS#. The data format is as follows:

| 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 | Description |
|---|---|---|---|---|---|---|---|-------------|
| 1 | n | n | n | n | n | n | n | Update DAC with the code n. |
| 0 | x | x | x | x | m | m | m | Update the ring mux selector with code m. x bits are ignored. |

The built-in DAC is a 7-bit device that drives a voltage from 0 to VREF/2 onto the DAC\_SET pin. An external resistor connected to ground turns this into a current that is drawn from the FB pin. The higher this current, the more the output voltage increases.

The nominal output voltage is Vref * (1 + Rtop/Rbot). With the values in the ST-251 schematic, this is 3.75V, assuming the DAC is set to 0.

The DAC increases the output voltage by Rtop * (VDAC / RDAC\_SET). VDAC is just (n/127) * (VREF/2). For example, a DAC code of 0x2A results in a DAC output voltage of 0.413V. Using the values in the ST-251 schematic, this is a current of 255uA pulled from the FB node. With an upper resistor of 10K, this corresponds with a voltage increase of 2.55V. The total output voltage is then 6.3V.

The ring detector is a window comparator that examines one of five possible differential inputs, A-E. The input is selected using the SPI bus with the MSB set to 1 and the value 0-4 loaded into the LSBs.

The window threshold is controlled by the voltage applied to the RING\_SET pin. It seems to be a percentage of that voltage. The output of the window comparator goes to a digitally-timed counter state machine which is incompletely understood but clocked by the SYS\_CLK input. The output of this is passed along to the RING\_DET output (which must be pulled up externally). In typical operation, this pin toggles on every edge of the ringing signal generated by the back EMF of the stepper motor winding.


# License
This work is licensed under a Creative Commons Attribution-ShareAlike 4.0
International License. See [https://creativecommons.org/licenses/by-sa/4.0/](https://creativecommons.org/licenses/by-sa/4.0/).




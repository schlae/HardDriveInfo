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
| Seagate      | ST-225N   | SCSI      | ASSY 20427  | 4-Wire       |         65ms | Oxide Coated | 130/40mm |
| Seagate      | ST-238    | MFM       | ASSY 20527  | 4-Wire       |         65ms | Oxide Coated | 130/40mm |
| Seagate      | ST-238R   | MFM/RLL   | ASSY 20527  | 4-Wire       |         65ms | Oxide Coated | 130/40mm |
| Seagate      | ST-251    | MFM       | ASSY 20629  | 10-Wire      |         40ms | Thin Film    | 130/40mm |
| Seagate      | ST-251-1  | MFM       | ASSY 20938  | 10-Wire      |         30ms | Thin Film    | 130/40mm |
| Seagate      | ST-251N   | SCSI      | ASSY 20603  | 10-Wire      |         40ms | Thin Film    | 130/40mm |
| Seagate      | ST-277R-1 | MFM/RLL   | ASSY 21020, 20938-300 | 10-Wire |    28ms | Thin Film    | 130/40mm |
| Seagate      | ST-296N   | SCSI      | ASSY 20741  | 10-Wire      |         28ms | Thin Film    | 130/40mm |
| Seagate      | ST-125    | MFM       | ASSY 20867  | 6-Wire       |         30ms | Thin Film    | 95/25mm  |
| Seagate      | ST-157R   | MFM/RLL   | ASSY 20829  | 6-Wire       |         30ms | Thin Film    | 95/25mm  |
| Seagate      | ST-157A-1 | AT-IDE    | 20948       |              |         28ms | Thin Film    | 95/25mm  |

## Assembly Information

The control boards are typically held to the drive by three screws. Two of the screws are insulated from the PCB ground with plastic washers, but one of them makes a solid ground connection to the chassis. This is (as far as I can tell) always the screw closest to the power connector.

Early ST-225 boards have an 18-pin header going to the read/write heads, but the flex cable has a 16-pin connector. It should be plugged in offset to the pin 1 end of the header, allowing pins 17 and 18 on the header to hang outside the connector.

## Platter Layout

### ST-225

For a stepper-motor hard drive, the motor phase is rather important because the rotor will "clock" to the phase you drive it with. The firmware uses this while searching for the index tracks so it can jump 8 tracks at a time, since it knows the inner index track is aligned to phase 5 and the outer to phase 7.

The "Track" column indicates the track as visible to the user (the controller card in the PC). From this layout, you can see that the data area contains 615 total tracks. The landing zone is specified at track 670 which is safely outside the data area and away from the inner index marker track.

The "Firmware Track" column is offset by 2 because this is how the drive's firmware numbers them; it limits the MFM controller from track 2 to track 672. This implies that there is no protection against the controller overwriting the inner index track -- you must enter the drive parameters in the BIOS correctly! The outer index track is inaccessible during normal operation.

| Track | Firmware Track | Description          | Motor Phase |
|-------|----------------|----------------------|-------------|
|    -1 |              1 | Index marker track   | 7 |
|     0 |              2 | Outermost data track | 6 |
|   ... |            ... | Data                 | ... |
|   614 |            616 | Innermost data track | 0 |
|   615 |            617 | DC Erased            | 7 |
|   616 |            618 | DC Erased            | 6 |
|   617 |            619 | Index marker track   | 5 |
|   618 |            620 | DC Erased            | 4 |
|   ... |            621 | ...                  | 3 |
|   670 |            672 | Landing zone         | 0 |
|  ~694 |           ~696 | Inner hard stop      | - |

*Note: Motor phase count is 0-7 since the motor is half-stepped.*

The index marker track contains the following repeating information:

| 0 - 4ms        | 4 - 16.67ms |
|----------------|-------------|
| 1.75MHz DC=17% | 5MHz DC=50% |

When the MCU PA7 is set as an input (logic 1), the drive will hold the hall-effect signal divider in reset as long as the 1.75MHz marker signal is present.

The index marker tracks are present only on head 0; none of the other surfaces contain this information.

### ST-225N (SCSI)

This drive has firmware and bad sector information recorded on tracks -1 and -2, presumably multiple copies located on each surface. Each sector is 256 bytes.

| Track | Head | Sector  | Description                       | Motor Phase |
|-------|------|---------|-----------------------------------|-------------|
|    -2 |   0  | 0 - 31  | Primary Operating System (copy 1) | 8 |
|    -2 |   1  | 0 - 31  | Primary Operating System (copy 2) | 8 |
|    -2 |   2  | 0 - 31  | Primary Operating System (copy 3) | 8 |
|    -2 |   3  | 0 - 31  | Primary Operating System (copy 4) | 8 |
|    -1 |   0  | 0       | Seagate copyright notice          | 7 |
|    -1 |   0  | 0 - 12  | Program overlay #1                | 7 |
|    -1 |   0  | 13 - 15 | Drive params?                     | 7 |
|    -1 |   0  | 16 - 17 | Defect map                        | 7 |
|    -1 |   0  | 19      | Drive serial number               | 7 |
|    -1 |   1  | 0 - 31  | Copy of track -1 head 0           | 7 |
|    -1 |   2  | 0 - 12  | Program overlay #2                | 7 |
|    -1 |   2  | 13 - 15 | Mfg defect map? Drive params?     | 7 |
|    -1 |   2  | 16 - 28 | Program overlay #3                | 7 |
|    -1 |   2  | 29      | Drive params? Identical to sector 13?     | 7 |
|    -1 |   3  | 0 - 31  | Copy of track -1 head 2           | 7 |
|     0 |      |         | Outermost data track | 6 |
|   ... |      |         | Data                 | ... |
|   614 |      |         | Innermost data track | 0 |
|   ... |      |         | ...                  |   |
|   670 |      |         | Landing zone         | 0 |
|  ~694 |      |         | Inner hard stop      | - |

*Note: Motor phase count is 0-7 since the motor is half-stepped.*

Data on track -2 is the primary operating system loaded in the 8051 at 0x4000, length of 0x1CFF. The MCU's ROM just contains enough code to load this operating system into memory and jump to it. All the ROM functions are reimplemented/duplicated in the firmware stored on track -2. 

Data on track -1 head 0, 2, and 3 is extra code loaded in the 8051 at 0x5000 (length = 0xD00) as an overlay that is swapped in and out as needed. RAM from 0x5D00 to 0x5FFF just contains data tables. Overlay code can call functions in the base OS using a dispatch function at 0x4BA1.

Binary dumps are included in the st225n/firmware directory. To calculate the file offset for a particular sector, multiple the sector number by 256.

Unused sectors contain the data 0x6C (01101100).

The defect map at track -1 heads 0/1 sectors 16-17 uses the following format:

Header:

| Byte | Content |
|------|---------|
|    0 | 0xBE    |
|    1 | 0xED    |
|    2 | Number of defects |
|    3 | Reserved (0xFE) |
|    4 | Reserved (0xED) |

Following the header is a list of defects, where each defect has the following format:

| Byte | Content |
|------|---------|
|  0   | Cylinder number, MSB |
|  1   | Cylinder number, LSB |
|  2   | Head number |
|  3   | Bytes |
|  4   | From index |

The last entry in the table contains 0xFF.


### ST-251

| Track | Description        | Motor Phase |
|-------|--------------------|-------------|
|    -4 | (End of outer crash stop) |   |
|    -3 | DC Erased                 | 7 |
|    -2 | Index marker track        | 8 |
|    -1 | DC Erased                 | 9 |
|     0 | Outermost data track      | 0 |
|   ... | Data                      |   |
|   819 | Innermost data track      | 9 |
|   820 | DC Erased                 | 0 |
|   821 | DC Erased                 | 1 |
|   822 | DC Erased                 | 2 |
|   823 | 1.75MHz                   | 3 |
|   824 | DC Erased                 | 4 |
|   825 | DC Erased                 | 5 |
|   826 | 1.75MHz                   | 6 |
|   827 | DC erased on odd tracks to inner crash stop. | |
|   ... | After track 827, 1.75MHz on all tracks with motor phases 0 and 6 to inner crash stop. All other even tracks DC erased. | |
|   910 | Landing zone              | |

Index marker track:

| 0 - 4ms        | 4 - 16.67ms |
|----------------|-------------|
| 2F             | 1.75MHz     |


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

### 20427 "225N INTELLIGENT"

The PCB schematic and layout for 20427 are available, see st225n/20427. This board is used in the ST-225N.

This board has a read/write channel similar to the plain ST-225, but the rest of it is quite different. All of the chips are now surface mount, including the Seagate custom devices.

The stepper motor driver has been replaced with a simple off-the-shelf MC3479.

An on-board data separator (DP8455) has been added, along with an Adaptec chipset for SCSI:

* AIC-250L - MFM encoder/decoder
* AIC-010L - Programmable mass storage controller
* AIC-300 - Dual port buffer controller

The 6500/1 microcontroller has been replaced with an 8051. The 8051 has much more intelligence than the old design which
only implemented seek buffering and the index track search. The firmware hasn't been completely analyzed yet but it

* Programs and manages the Adaptec chipset to implement the SCSI interface
* Handles the stepper motor seek
* Loads extra firmware into its dedicated SRAM chip from "hidden" tracks on the drive surface

Some of the custom chips seem to share functionality with the ST-251, which may have been under development at the same time.

* The 11647-501 was used on the ST-251 as the MFM interface, but on this board it performs housekeeping functions and is wired differently.
* The 11695-502 has been repackaged into SMD (unlike the older 11695-002) and was also used on the ST-251 to control the spindle driver.

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

### 20603 ST-251N Control Board

The ST-251N is based on the ST-251 platform. It uses a SCSI controller design similar to that of the ST-225N but with several major improvements:

* Encoding is now RLL 2,7 for improved data density, using the DP8462 + AIC-270L instead of the old DP8455 + AIC250L combination.
* Discrete SCSI bus buffers 2x 74LS240, 74LS373, and 4x 74F38 have been replaced by an LSI, the 11734-501.
* The Adaptec AIC-300L controller has been upgraded to the AIC-301L controller.


### 20741 ST-296N Control Board

The ST-296N is based on the ST-251 platform. It's very similar to the ST-251N SCSI design but it has a different 2,7 RLL encoder/decoder. 

The PCB schematic and layout for 20741 are available, see st296n/20741.

This board is very similar to the ST-251N board 20603. Changes are as follows:

* A new 2,7 RLL encoder/decoder IC, the 11741-502, has been added to replace the old DP8462 + AIC-270L combination.
* A new RLL enc/dec window control register has been added to the 8051 at 0xE000.
* The SCSI buffer has been doubled, from 1K to 2K bytes.
* A new halfstep mode has been added to head actuator. It is controlled by bit 2 of 8051 register 0xA000.

Minor changes include:

* The read channel filter components have been adjusted slightly.
* The read channel pulse networks have been adjusted slightly.
* Spindle motor RC balancing network has been incorporated into the board (was a bodge).
* Several diodes are now surface mount.
* The write current now has a jumper to allow external control over write current (at the factory).
* Write path test points have been added to allow the factory to write to the platters.
* The head retract signal has been disconnected from the 10223-502 chip. It uses the DC\_UNSAFE comparator from the 10206-501 chip instead.
* The SCSI ID header has been increased to 10 pins by adding two no-connect pins.

## Firmware

### ST-251

The firmware from the 20629 board has been dumped. The label is marked "ST 251 / LSIL18". The firmware has been fed through Ghidra and a good chunk of it has been commented and labeled. See [the repository](st251/firmware/ST251_commented.txt). Some parts of it are not understood.

### ST-225

The firmware from the ST-225's ROM microcontroller, 80007-001, has been dumped, disassembled, and commented. See st225/firmware.

Fun facts:

* Some (but not all) ST-225 logic boards have an array of resistors and two 7445 chips. These are connected to the stepper motor and are used to adjust
the voltage on individual windings to subtly shift the position of the stepper motor (microstepping). This is used in recovery mode.
* Early ST-225 logic boards have a latch and EPROM chip. These are not populated. They are not used to store firmware. The microcontroller's internal
ROM contains code that uses this external EPROM as a lookup table of 1 byte per track. Each contains two 4-bit values used to microstep the head position
(two values because the value can be different if you are stepping inwards to the track or stepping outwards to the track). This may have been used to provide
a custom "defect map" for a single-platter drive, using the stepper to avoid bad areas on the platter.
* An undocumented "pause mode" appears in the firmware. By pulling PA4 low (pin 34 of the MCU), it enters a state where it allows all GPIO pins to go high/float. Presumable this was useful for debugging.

### ST-225N

TBD, stay tuned.

### ST-251N

TBD, stay tuned.

### ST-296N

TBD, stay tuned

## Chips

Click the part number for more information on that particular device.

| Part Number | Nicknames     | Where Used | Description |
|-------------|---------------|------------|-------------|
| [10014-002](chips/10188-501.md)   |               | 20527 | Head center tap driver, write current selector. |
| [10188-501](chips/10188-501.md)   | SSI257.2, V10096BQZ | 20603, 20629, 20938, 21020, 20867, 20829, 20948, 20741, 20427 | Head center tap driver, write current selector. |
| [10189-521](chips/10189-521.md)   | SSI257        | 20629, 20938, 20829 | Steering diodes and head preamp, NE592 equivalent. |
| [10189-501](chips/10189-521.md)   |               | 20867 | Steering diodes and head preamp, NE592 equivalent. |
| [10189-502](chips/10189-521.md)   |               | 21020, 20603, 20741  | Steering diodes and head preamp, NE592 equivalent. |
| [SSI 280](chips/SSI280.md)     |               | 20301 | Read channel and write amplifier. |
| [SSI 280.2](chips/SSI280.md)   |               | 20427 | Read channel and write amplifier. |
| [10206-501](chips/10206-501.md)   | SSI296        | 20629, 20938, 21020, 20603, 20867, 20741 | Read channel and write amplifier. |
| [10206-502](chips/10206-501.md)   |               | 20829, 20948 | Read channel and write amplifier. |
| [10206-002](chips/10206-002.md)   |               | 20527 | Read channel and write amplifier. |
| [10210-501](chips/11782-501.md)   |               | 20603 | 5-Phase stepper motor controller. |
| [10223-501](chips/10223-502.md)   | RETURN        | 20867 | Retracts stepper motor on power-down. |
| [10223-502](chips/10223-502.md)   | RETURN        | 20603, 20629, 20938, 21020, 20829, 20948, 20741 | Retracts stepper motor on power-down. |
| 11468       |               | 21020 | Controls H-Bridge enables based on phase state. |
| [11642-001](chips/10189-521.md)   |               | 20301, 20527  | Steering diodes and head preamp, NE592 equivalent. See 10189.|
| [11642-501](chips/10189-521.md)   |               | 20427 | Steering diodes and head preamp, NE592 equivalent. |
| [11647-501](chips/11647-502.md)   |               | 20427 | Miscellaneous logic. Seems similar to 11647-502 but used in another mode. |
| [11647-502](chips/11647-502.md)   | STEP LOGIC    | 20629, 20938, 21020, 20867, 20829 | MFM interface logic. |
| [11665-001](chips/11665-001.md)   |               | 20301, 20527 | Stepper motor control and driver chip (4 phase). |
| [11695-002](chips/11695-002.md)   |               | 20301, 20257 | Spindle speed control chip. |
| [11695-502](chips/11695-002.md)   | SPEED CONTROL | 20629, 20427 | Spindle speed control chip. |
| 11721-501   |               | 20603, 20938, 21020, 20741 | Similar to the L293 H-bridge driver |
| 11734-501   |               | 20603, 20741 | SCSI bus transceiver. |
| 11738-002   |               | 21020 | Similar to 11791 but in a different package. |
| 11738-502   | IP3M05AW      | 20603, 20741 | Similar to 11791 but in a different package. |
| 11741-502   |               | 20948, 20741 | SSI 2,7 RLL codec/data separator. Similar to 32D5321. |
| [11743-501](chips/11743-501.md)   |               | 20603, 20938, 20741 | Controls H-Bridge enables based on phase state. |
| [11743-521](chips/11743-521.md)   |               | 20938-300 | Controls H-Bridge enables based on phase state. |
| [11744-501](chips/11744-502.md)   | RING DETECTOR | 20867 | Stepper motor seek settling chip - adaptive ringout. |
| [11744-502](chips/11744-502.md)   | RING DETECTOR | 20603, 20629, 20938, 21020, 20741 | Stepper motor seek settling chip - adaptive ringout. |
| [11747-501](chips/11747-501.md)   |               | 20603, 20948, 20741 | Index pulse divider and 5-phase stepper motor low-side driver. |
| [11782-501](chips/11782-501.md)   | STEP SEQR     | 20938, 21020, 20867, 20741 | 5-Phase stepper motor controller. |
| 11789-502   |               | 20829 | Unknown. Maybe a stepper motor controller with built-in driver transistors. |
| 11789-504   |               | 20829, 20948 | Unknown. Maybe a stepper motor controller with built-in driver transistors. |
| 11791       |               | 20938, 20829, 20948 | Spindle motor driver. Similar to the HA13406W. |
| IP3M05AW    |               | 20867, 20829 | Spindle motor driver. Similar to 11791 but in a different package. |
| 12654-501   |               | 20948 | AT-IDE interface. |
| 80007-001   | R10L7-11      | 20301, 20527 | Rockwell R6500 6502-core microcontroller. |
| R6518AJ     | R1113-18      | 20629 | Rockwell R6518 6502-core microcontroller. Requires external ROM. |
| 80118-502   | R1512-12      | 20938 | Rockwell R6518(?) 6502-core microcontroller. |
| 80118-505   | R1818-11      | 21020 | Rockwell R6518(?) 6502-core microcontroller. |
| 80044-401   | SAB8051A-N    | 20603 | Intel 8051 microcontroller. |
| 80049-501   | SCN8051HCCA44 | 20867 | Intel 8051 microcontroller. |
| 80074-504   | SAB8052A-N    | 20741 | Intel 8052 microcontroller. |
| 80084-508   | SAB8052A-N    | 20829 | Intel 8052 microcontroller. |
| 80127-506   | SCN8052HCCA44 | 20948 | Intel 8052 microcontroller. |


## More Information

A large amount of Seagate drive information can be found on [Bitsavers](https://bitsavers.org/pdf/seagate/).

Some operational clues for the ST-225 are available in the [ST-225 OEM Manual](https://www.minuszerodegrees.net/manuals/Seagate/Seagate%20ST225%20-%20OEM%20Manual%20-%20Oct85.pdf).


## License
This work is licensed under a Creative Commons Attribution-ShareAlike 4.0
International License. See [https://creativecommons.org/licenses/by-sa/4.0/](https://creativecommons.org/licenses/by-sa/4.0/).




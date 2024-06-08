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
| 10223-502   | RET           | 20629, 20938 | Retracts stepper motor on power-down. |
| 11743-501   |               | 20938 | Controls H-Bridge enables based on phase state. |
| 11743-521   |               | 20938-300 | Controls H-Bridge enables based on phase state. |
| 11782-501   | STEP SEQR     | 20938 | 5-Phase stepper motor controller. |
| 11791       |               | 20938 | Spindle motor driver. Similar to the HA13406W. |
| 10206-501   | SSI296        | 20629, 20938 | Read channel and write amplifier. |
| 11647-502   | STEP LOGIC    | 20629, 20938 | MFM interface logic. |
| 80118-502   | R1512-12      | 20938 | Rockwell R6518(?) 6502-core microcontroller. |



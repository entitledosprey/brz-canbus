# 2019 Subaru BRZ (6MT) CAN bus reverse engineering

This is a map of the CAN traffic seen at the OBD port. It covers parked tests, a 12-minute drive, a shifter test, and a Link ECU datalog from the same engine runs. All captures were listen-only through a PEAK PCAN-USB, and nothing was transmitted to the car.

**This car runs a Link G5 ECU (Topboard, firmware 7.8.2), a 4.30 final drive and 245/40R17 tires.** Engine messages may be generated or shaped by the Link's configuration, so don't assume they match a stock BRZ.

**The full per-ID table is in [PROTOCOL.md](PROTOCOL.md):** ID, byte count, period, byte order, counters and checksums, and contents. `analysis/protocol.csv` has the same data.

## Bus

| Property | Value | Evidence |
|---|---|---|
| Bitrate | 500 kbit/s | Listen-only detection, 0 bus errors in every capture |
| Frame format | Classic CAN, standard 11-bit IDs | No extended IDs in 1,793,929 analyzed frames |
| Unique IDs | 37 | 36 periodic, plus `0x000` seen once |
| Byte count | Fixed per ID: 8, except `0x0D1` (4) and `0x0D3` (7) | No ID ever changed length |
| Traffic | About 975 frames/s, about 21% bus load | Load is estimated, ignoring bit stuffing, so the real figure is a little higher |
| Drops | 0 lost by the adapter on the drive | Kernel interface counters. 30 were counted across the earlier parked work, cause unknown |

Bits are written `byte.bit`, with byte 0 first and bit 0 least significant.

## Byte order

Byte order is per message and can't be assumed across the bus.

| Result | IDs |
|---|---|
| **Big-endian, proven** (1) | `0x018` |
| **Little-endian, proven** (8) | `0x0D0`, `0x0D1`, `0x0D4`, `0x140`, `0x141`, `0x142`, `0x360`, `0x370` |
| Likely little-endian (1) | `0x0D2`, from a single 0.6 s event |
| No multi-byte field moved (17) | `0x0D3`, `0x152`, `0x280`, `0x282`, `0x284`, `0x361`, `0x372`, `0x374`, `0x375`, `0x37A`, `0x3D1`, `0x440`, `0x442`, `0x44D`, `0x46C`, `0x6E1`, `0x6E2` |
| Payload never moved (10) | `0x000`, `0x144`, `0x156`, `0x4C1`, `0x4C3`, `0x4C6`, `0x4C8`, `0x4DC`, `0x4DD`, `0x63B` |

"Proven" means a low byte wrapped while its neighbour stepped by exactly 1, in one direction only. `0x141`, `0x142` and `0x360` have a second, independent proof: their fields match ECU channels when read little-endian (r about 0.96) and not big-endian (r 0.35 or less). The ECU log did not add a proof for any other ID. The last two groups have no field that can show byte order in this data, so it isn't known for them. `PROTOCOL.md` also has a "Decode as" column that gives a usable answer for every ID. It assumes little-endian where nothing was measured, because 8 of the 9 measured IDs are little-endian, and it labels those entries as assumed.

"None seen" doesn't prove a field is single-byte: `0x282` b3 and b4, for example, move slowly and independently, which suggests separate bytes but doesn't rule out anything.

## Decoded signals

### Confirmed by a physical action or by physics

| Signal | Location | Evidence |
|---|---|---|
| Vehicle speed | `0x0D1` b0-1, little-endian, about 0.0563 km/h per bit | RPM-per-speed clusters match the six gear ratios. Follows the rear wheels. See the scale note below |
| Wheel speeds | `0x0D4` b0-1 FL, b2-3 FR, b4-5 RL, b6-7 RR, little-endian, same scale | b4-7 run 2.0% faster on power, equal with the clutch in, so they're rear on this RWD car. Left wheels run about 8% slower in left turns |
| Engine RPM | `0x140` b2-3 and `0x141` b4-5, little-endian, low 14 bits, 1 rpm/bit | Identical in both. ECU Engine Speed: r=0.995, slope 0.996, peaks 7360 and 7358 |
| Accelerator pedal | `0x140` b0, mirrored in b4 and b5, about 0.38 %/bit | Slow sweeps with the engine off. ECU APS: r=0.974 |
| Pedal pressed | `0x140` 3.6 | Equal to b0 > 0 |
| Throttle position | `0x140` b6, about 0.38 %/bit; `0x142` b2, about 0.33 %/bit | ECU TPS: r=0.972 and 0.968 |
| Oil temperature | `0x360` b2, raw - 40 °C | ECU Oil Temperature: r=1.000, within 1 °C |
| Coolant temperature | `0x360` b3, raw - 40 °C | ECU ECT: r=0.999, within 1 °C |
| A/C request | `0x280` 0.0 | Agrees with the ECU channel in 97.6% of samples |
| A/C clutch | `0x142` 1.7 | Agrees with the ECU channel in 97.4% of samples |
| Clutch | `0x140` 1.7 | 10 changes for 5 presses |
| Reverse switch | `0x152` 6.2 | 6 changes for 3 shifts into reverse and out, engine off |
| Parking brake | `0x152` 6.3, 1 = applied | 6 changes for 3 cycles |
| Brake switch | `0x152` 6.4, `0x0D3` 2.5 | 10 changes for 5 presses |
| Brake pressure | `0x0D1` b2, raw, up to 84 | Rises only during presses. No calibration |
| Door | `0x374` 3.0, `0x375` 1.0 and 3.2, 1 = open | 7 changes for an initial close plus 3 cycles. `0x375` 3.2 also changed once without the door during the ignition change |
| Door lock | `0x375` b0 bits 0-1: 3 = unlocked, 0 = locked | 6 changes for 3 lock and unlock cycles |
| Lock button | `0x375` 2.0, 2.1 pulse on each press; 2.7 on unlock | Same test |
| Seatbelt | `0x282` 5.0, 1 = unbuckled | 7 changes after the door group. One 1 s blip at speed in the drive is unexplained |
| Exterior lights | `0x152` 7.2 parking, 7.3 low beam, 7.4 high beam | Stalk sequence and 3 flashes |
| Low beam lamp | `0x374` 1.6 | On with low beam, off during each high-beam flash |
| Distance counter | `0x0D3` b4, bits 2-7, wraps at 64, about 0.2 m per count | r=+1.00000 with integrated speed, in 5 s windows |
| Steering angle | `0x018` b0-1 big-endian signed; `0x0D0` b0-1 little-endian signed | r=-0.9998 between them, about ±4590 raw, 0 at center. Left is positive in `0x018`, negative in `0x0D0` |
| `0x018` checksum | b7 = (b0 + ... + b6 + `0x18` + 8) & `0xFF` | Matched all 73,381 frames on the drive and 83,698 in the parked session |

### Derived: the meaning is clear but the scale is estimated

| Signal | Location | Basis |
|---|---|---|
| Yaw rate | `0x0D0` b2-3, little-endian signed, positive turning left, about 0.13 °/s per bit | Sign follows steering in turns. Scale assumes lateral and longitudinal bytes share a scale |
| Lateral force | `0x0D0` b6, signed, positive turning left | r=+0.896 with speed times yaw |
| Longitudinal force | `0x0D0` b7, signed, negative when speeding up, about 0.085 m/s² per bit | r=-0.92 with rate of speed change |
| Driver torque request | `0x141` b0-1, low 12 bits, little-endian, about 0.081 Nm/bit, offset about -121 Nm | ECU channel r=0.962, big-endian r=0.17 |
| Estimated torque | `0x141` b2-3, low 12 bits, about 0.083 Nm/bit, offset about -154 Nm; `0x142` b0-1, low 12 bits, about 0.081 Nm/bit, offset about -120 Nm | ECU channel r=0.958 and 0.962, big-endian r 0.35 or less. Scales are fits, so check them before relying on them. All three fields stop at 4095 under high load |
| Gear (no message found) | Calculate from RPM / `0x0D1`: 7.67, 4.62, 3.24, 2.55, 2.10, 1.61 for 1st to 6th, clutch out | No byte in the log follows gear, and the ECU's own Gear channel stayed 0 |

**Speed scale:** the ECU's RPM/kph channel gives 0.0560-0.0563 km/h per bit in all six gears. Separately, recalled gear ratios, the 4.30 final drive and an unloaded 245/40R17 tire give 0.0578, and a loaded tire rolling about 2.6% smaller would close that gap. It hasn't been compared with GPS, and the ECU's own speed calibration isn't known, so treat it as within a few percent. The peak recorded was 2292 raw, about 129 km/h on this scale.

### Likely: consistent, but the meaning isn't proven

| Field | Behaviour | Best guess |
|---|---|---|
| `0x6E2` b3-b6 | `FE` after ignition on, filled in one at a time. Rise in steps of 4 during the drive (`5C`, `60`, `64`) | Tire pressures. Scale and wheel order unknown |
| `0x361` 1.5 | Set just after start, clear just after stop | Engine running |
| `0x284` 5.4 | Follows the clutch near the floor only when the engine is off. Rarely changed while driving | Starter clutch interlock |
| `0x152` 7.1 | Opposite of parking brake in the drive | Mirror of parking brake |
| `0x141` 5.7, 6.6 | Opposite level to pedal pressed | Closed-throttle flags |
| `0x360` b0-1, little-endian | r=0.964 with the ECU's calculated engine power, big-endian r=-0.34 | Airflow or fuel |
| `0x360` b4 | r=0.962 with estimated torque, r=0.950 with MAP. Doesn't fit either exactly | Load related |
| `0x370` b0-1, little-endian | r=+0.805 with steering lock, parked | Power steering load |

### Network management

`0x440`, `0x442`, `0x44D` and `0x46C` pass a token: in each frame, b0 + `0x400` names the next sender. That held for 8,634 of 8,634 hand-offs in the main session. The order is `0x440` → `0x442` → `0x44D` → `0x46C`, about 97 ms per hop, and b1 is `02`. Around an ignition change, the ring re-forms and b1 briefly changes. This matches the OSEK direct network management pattern, though the name is an inference.

### Not explained

- **`0x0D2` b2-4 with `0x0D3` 2.1:** one 0.6 s event at full throttle, about 127 km/h. Stability or traction related is a guess.
- **`0x0D0` b4-5:** zero when parked, then held for a few seconds at a time while moving.
- **`0x37A` b5:** `04` or `10`, changes around door closing and pressing the clutch before a start.
- **`0x375` 5.6:** on from just after buckling up to 10 s after unbuckling.
- **`0x282` 5.1:** a few changes while driving. Possibly passenger-seat related, since the laptop was there.
- **`0x282` b0, b3, b4; `0x3D1` b0; the rest of `0x280` b0:** slow changes. They share trends with temperatures, but none fits an ECU channel's scale.
- **`0x000`:** one frame at engine stop.
- **`0x4C1` to `0x4DD`, `0x63B`, `0x144`, `0x156`, `0x6E1`:** nothing moved, or almost nothing.

### Tested and not found

- **Turn signals:** neither direction produced a matching bit, so they may not be on this bus.
- **Gear position and neutral:** no bit followed them, apart from reverse.
- **`0x361` b0 as gear, from recalled community notes:** wrong for this car. It stayed 0 throughout.

## Limits

- Only the bus at the OBD port was captured. Other buses and request-only diagnostic data aren't covered.
- Each signal was only tested in the states it was exercised in. Wipers, A/C, cruise control and similar controls weren't tested.
- The ECU comparison covers the 678 s of the drive where the logs overlap, aligned on RPM to within about 0.3 s. It can only identify channels the ECU logged.
- Other checksums may exist: only `0x018` matched the sum and XOR formulas tried.
- The left-right sign comes from being asked to steer left first, and yaw sign is consistent with it.
- The engine-start marker began about 1 s after cranking. In the shifter capture, the first 67 s include engine running, throttle blips and an ignition change, which briefly reset many bits, so only events after 70 s were used.

## What's here

| Path | Contents |
|---|---|
| `PROTOCOL.md`, `analysis/protocol.csv` | Message table |
| `BRZ_2019.dbc` | Confirmed and likely signals. Decodes every frame of the listed IDs |
| `captures/1793645158-can0.*` | Parked session, with labelled tests |
| `captures/1793647302-can0.*` | 12-minute drive |
| `captures/1793648539-can0.*` | Shifter and door lock test |
| `captures/1793645037-can0.*`, `captures/1793644816-can0.*` | Ignition-on baselines |
| `analysis/` | Raw output behind the claims above |

The Link ECU datalog and the analysis scripts aren't in this repo. `analysis/` holds their output.

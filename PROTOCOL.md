# BRZ CAN message table

All 37 IDs seen, standard 11-bit, classic CAN at 500 kbit/s. Bits are `byte.bit` with bit 0 least significant, and `b0` is the first byte. "Clean carries" counts low-byte wraps where the neighbouring byte stepped by exactly 1. "Decode as" is a usable answer for every ID, but only entries marked measured are proven. Assumed entries use little-endian because 8 of the 9 measured IDs are little-endian.

| ID | Bytes | Period | Frames | Byte order | Decode as | Counter / checksum | Carries |
|---|---|---|---|---|---|---|---|
| `0x000` | 8 | once | 1 | Unknown: payload never moved | Little (assumed, not measured) | None found | Seen once, at engine stop: 01 00 00 00 00 00 00 00. Meaning unknown |
| `0x018` | 8 | 10 ms | 183402 | Big (610 clean carries) | Big (measured) | b3 bits 1-3 counter; b7 checksum | Steering angle (b0-1) |
| `0x0D0` | 8 | 20 ms | 91660 | Little (579 clean carries) | Little (measured) | None found | Steering angle (b0-1), yaw (b2-3), lateral (b6), longitudinal (b7); b4-5 unknown |
| `0x0D1` | 4 | 20 ms | 91660 | Little (160 clean carries) | Little (measured) | None found | Vehicle speed (b0-1), brake pressure (b2) |
| `0x0D2` | 8 | 20 ms | 91660 | Likely little: b2 moved with b3 fixed in a single 0.6 s event | Little (weak evidence) | None found | Zero/FFFF except one 0.6 s event at full throttle; meaning unknown |
| `0x0D3` | 7 | 20 ms | 91659 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | b6 counter 0-255 | Brake switch (2.5), distance counter (b4 bits 2-7), 2.1 set in the 0x0D2 event |
| `0x0D4` | 8 | 20 ms | 91659 | Little (692 clean carries) | Little (measured) | None found | Wheel speeds FL, FR, RL, RR (b0-7, 16 bits each) |
| `0x140` | 8 | 10 ms | 183278 | Little (1036 clean carries) | Little (measured) | b1 low nibble counter | Pedal (b0, b4, b5), clutch (1.7), RPM (b2-3), pedal pressed (3.6), throttle (b6) |
| `0x141` | 8 | 10 ms | 183278 | Little (3063 clean carries) | Little (measured) | None found | RPM (b4-5), driver torque request (b0-1 low 12 bits), estimated torque (b2-3 low 12 bits); flags 5.7, 6.6 |
| `0x142` | 8 | 10 ms | 183278 | Little (1295 clean carries) | Little (measured) | None found | Throttle (b2), torque (b0-1 low 12 bits), A/C clutch (1.7) |
| `0x144` | 8 | 20 ms | 91639 | Unknown: payload never moved | Little (assumed, not measured) | None found | Never changed |
| `0x152` | 8 | 20 ms | 92030 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | b1 high nibble counter | Reverse (6.2), parking brake (6.3, inverse at 7.1), brake (6.4), lights (7.2-7.4) |
| `0x156` | 8 | 20 ms | 91936 | Unknown: payload never moved | Little (assumed, not measured) | None found | All zero, even while driving |
| `0x280` | 8 | 20 ms | 91675 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | b1 low nibble counter | A/C request (0.0); other b0 bits unknown |
| `0x282` | 8 | 60 ms | 30646 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | b1 high nibble counter | Seatbelt (5.0); 5.1 unknown; b0, b3, b4 slow values, meaning unknown |
| `0x284` | 8 | 100 ms | 18391 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | b1 low nibble counter | Clutch start interlock (5.4); 5.7 clear while running |
| `0x360` | 8 | 50 ms | 36656 | Little (364 clean carries) | Little (measured) | None found | Oil temp (b2), coolant temp (b3), both raw - 40 C; b0-1 tracks engine power; b4 tracks torque and MAP |
| `0x361` | 8 | 50 ms | 36656 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | b4 low nibble counter | Engine running (1.5) |
| `0x370` | 8 | 51 ms | 36263 | Little (20 clean carries) | Little (measured) | b5 low nibble counter | b0-1 rises with steering lock, possibly power steering load; b2-3 set with engine off |
| `0x372` | 8 | 100 ms | 18339 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | b1 high nibble counter | Only the counter moves, apart from power changes |
| `0x374` | 8 | 1000 ms | 1848 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | None found | Door (3.0), low beam lamp (1.6) |
| `0x375` | 8 | 1000 ms | 1856 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | None found | Door lock (b0 bits 0-1), lock and unlock presses (2.0, 2.1, 2.7), door (1.0, 3.2), parking lights (3.3) |
| `0x37A` | 8 | 1001 ms | 1841 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | None found | b5 04 or 10, meaning unknown |
| `0x3D1` | 8 | 120 ms | 15323 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | None found | b0 slowly rises, meaning unknown |
| `0x440` | 8 | 390 ms | 4743 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | None found | Network management ring, next node 0x442 |
| `0x442` | 8 | 390 ms | 4746 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | None found | Network management ring, next node 0x44D |
| `0x44D` | 8 | 390 ms | 4745 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | None found | Network management ring, next node 0x46C |
| `0x46C` | 8 | 390 ms | 4727 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | None found | Network management ring, next node 0x440 |
| `0x4C1` | 8 | 1000 ms | 1834 | Unknown: payload never moved | Little (assumed, not measured) | None found | Never changed, b0 = 01 |
| `0x4C3` | 8 | 1000 ms | 1835 | Unknown: payload never moved | Little (assumed, not measured) | None found | Never changed, b0 = 03 |
| `0x4C6` | 8 | 999 ms | 1834 | Unknown: payload never moved | Little (assumed, not measured) | None found | Never changed, b0 = 07 |
| `0x4C8` | 8 | 1011 ms | 1813 | Unknown: payload never moved | Little (assumed, not measured) | None found | Never changed, b0 = 09 |
| `0x4DC` | 8 | 1000 ms | 1834 | Unknown: payload never moved | Little (assumed, not measured) | None found | Never changed, b0 = 1D |
| `0x4DD` | 8 | 1000 ms | 1834 | Unknown: payload never moved | Little (assumed, not measured) | None found | Never changed, b0 = 1E |
| `0x63B` | 8 | 500 ms | 3679 | Unknown: payload never moved | Little (assumed, not measured) | None found | Never changed: 00 00 FF FF 00 FF FF FF |
| `0x6E1` | 8 | 1000 ms | 1832 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | None found | b0 = A1, its ID's low byte; 5.5 and 5.6 changed once, after engine stop |
| `0x6E2` | 8 | 1000 ms | 1839 | None seen: only single-byte or bit fields moved | Not applicable to fields seen; little assumed for any others | None found | b0 = A2, its ID's low byte; likely tire pressures in b3-b6, FE until each is received |

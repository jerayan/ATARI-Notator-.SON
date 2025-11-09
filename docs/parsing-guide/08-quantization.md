# Track Quantization

## Overview

Each track in a Notator .SON file has a quantization setting that determines how notes are quantized. The quantization value is stored in a special pattern `2c XX ?? YY` located within the track data.

## Location

The quantization pattern `2c XX ?? YY` is typically found at **offset +32 bytes from the start of the track** (after the boundary marker and track name).

## Pattern Structure

```
Offset: +32 from track start
Byte 0: 0x2c (pattern identifier)
Byte 1: XX (MIDI channel or flag - can be 0x01, 0xA1, 0xA11, etc.)
Byte 2: ?? (quantization value - this byte contains the quantization)
Byte 3: YY (track transpose)
```

**Note about Byte 1 (XX):**
- Byte 1 contains **MIDI channel** in format A1, A11, etc.
- According to `notator-parser.js`, Byte 1 in pattern `2c XX ?? YY` contains channel information
- Format can be:
  - `0x01` = Channel 1 (or Port A, Channel 1 = A1)
  - `0xA1` = Port A, Channel 1 (A1)
  - `0xA11` = Port A, Channel 11 (A11)
- Channel is parsed from Byte 1, not from Byte 3 and Byte 5 of the track header (those are used in `son2midi.js` for a different purpose)

## Dynamic Parsing Algorithm

Quantization is stored in Byte 2 (`??`) of the pattern `2c XX ?? YY`. This byte acts as an index into a global table stored in `NOTATOR.PRG` (data section offset `0x00B4`). No dynamic calculation is required: Notator simply looks up the tick resolution in this table. The 11 defined entries are:

| Byte Value | Quantization (ticks) |
|------------|----------------------|
| `0x00`     | 4                    |
| `0x01`     | 6                    |
| `0x02`     | 8                    |
| `0x03`     | 12                   |
| `0x04`     | 16                   |
| `0x05`     | 24                   |
| `0x06`     | 32                   |
| `0x07`     | 48                   |
| `0x08`     | 64                   |
| `0x09`     | 96                   |
| `0x0A`     | 768                  |


## Using the Value in a Parser

1. Locate the pattern `2c XX ?? YY` in the track header (typically ~+32 bytes from the track start; the first track may appear slightly later).
2. Read byte `??`, treat it as an index, and look up the tick value in the table above.
3. Apply that tick resolution when displaying or quantising the track.

## Notes

- The lookup table resides in `NOTATOR.PRG` (data section offset `0x00B4`).
- `0x0A` (768 ticks) is the default used by factory patterns; the other entries correspond to the Notator quantise menu.
- Pattern `2c 01 ?? 00` appears in every track boundary; byte `??` is the only per-track quantisation parameter.

## References

- Test files: `QUANTIZE.SON`, `QUANTIZ1.SON`, `QUANTIZ2.SON`, `FL123.SON`.
- Pattern: `2c 01 ?? 00` where `??` is the quantisation index.

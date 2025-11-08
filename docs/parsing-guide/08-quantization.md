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

Quantization is parsed **dynamically** from the byte value using the following algorithm:

```javascript
function parseQuantize(byteValue) {
    if (byteValue === 0) {
        return 8;  // Default quantization
    } else if (byteValue < 6) {
        return byteValue * 16;  // For values 1-5
    } else if (byteValue === 6) {
        return byteValue * 128;  // Special case: 6 * 128 = 768
    } else if (byteValue >= 8) {
        return Math.pow(2, byteValue - 8);  // For values >= 8: 2^(byte-8)
    }
    // For byteValue == 7 not defined (not in test data)
    return 8;  // Fallback to default
}
```

### Alternative Implementation with Bit Shift

```javascript
function parseQuantize(byteValue) {
    if (byteValue === 0) {
        return 8;
    } else if (byteValue < 6) {
        return byteValue << 4;  // byte * 16
    } else if (byteValue === 6) {
        return byteValue << 7;  // byte * 128
    } else if (byteValue >= 8) {
        return 1 << (byteValue - 8);  // 2^(byte-8)
    }
    return 8;
}
```

## Test Values

| Track | Byte Value | Calculation | Quantization |
|-------|------------|-------------|--------------|
| FLUTE1 | 6 | 6 * 128 | 768 |
| FLUTE2 | 2 | 2 * 16 | 32 |
| FLUTE3 | 0 | default | 8 |
| FLUTE4 | 10 | 2^(10-8) = 2^2 | 4 |

## Implementation in Parser

### Step 1: Find Pattern 2c

```javascript
// Search for pattern 2c XX ?? YY in track
// Usually at offset +32 from track start (after boundary + name + header)
const trackStart = /* track start offset */;
const searchStart = trackStart + 32;
const searchEnd = Math.min(trackStart + 100, data.length - 4);

for (let i = searchStart; i < searchEnd; i++) {
    if (data[i] === 0x2c && i + 3 < data.length) {
        const quantizeByte = data[i + 2];
        const quantize = parseQuantize(quantizeByte);
        // ... use quantization
        break;
    }
}
```

### Step 2: Apply Algorithm

```javascript
const quantize = parseQuantize(quantizeByte);
```

## Notes

- **No lookup table is needed** - everything is calculated dynamically
- The algorithm covers all known values from test data
- For value `byte == 7` not present in test data, fallback to default (8) is used
- Pattern `2c` may be at different offsets depending on track structure, but usually at +32 bytes

## References

- Test file: `QUANTIZE.SON`
- Tracks: FLUTE1 (768), FLUTE2 (32), FLUTE3 (8), FLUTE4 (4)
- Pattern: `2c 01 ?? 00` where `??` contains quantization byte value


# File Header Structure

## Overview

The .SON file header contains global song information including tempo, time signature, and file identification.

## Header Offsets

### Magic Number
- **Offset**: `0x0000`
- **Size**: 2 bytes (16-bit)
- **Format**: Big-endian
- **Value**: `0x3B9E` (file identifier)
- **Purpose**: Identifies file as Notator .SON format

### Pulses Per Quarter (PPQ)
- **Offset**: `0x0002`
- **Size**: 2 bytes (16-bit)
- **Format**: Big-endian unsigned integer
- **Typical Value**: `192` (0x00C0)
- **Purpose**: Defines timing resolution for MIDI events

### Time Signature Numerator
- **Offset**: `0x001D`
- **Size**: 1 byte (8-bit)
- **Format**: Unsigned integer
- **Range**: 2-7 (common values: 2, 3, 4, 5, 6, 7)
- **Purpose**: Initial time signature numerator (beats per measure)

### Time Signature Denominator
- **Offset**: `0x0021`
- **Size**: 1 byte (8-bit)
- **Format**: Unsigned integer
- **Range**: 2, 4, 8, 16 (common values: 2, 4, 8)
- **Purpose**: Initial time signature denominator (note value)

**Example:**
- Numerator = `4`, Denominator = `4` → 4/4 time signature
- Numerator = `6`, Denominator = `8` → 6/8 time signature

### Tempo
- **Offset**: `0x03F0`
- **Size**: 4 bytes (32-bit)
- **Format**: Big-endian signed integer
- **Encoding**: Scaled tempo value (BPM × 10000)
- **Calculation**: `tempo_bpm = tempo_value / 10000.0`
- **Purpose**: Initial song tempo in BPM

**Examples:**
- `0x0010C8E0` (1100000) → 110.0 BPM
- `0x000E4E40` (920000) → 92.0 BPM
- `0x0011F950` (1179984) → 117.9984 BPM

## Header Structure Summary

```
Offset    Size    Description
─────────────────────────────────────────
0x0000    2       Magic number (0x3B9E)
0x0002    2       PPQ (typically 192)
...       ...     (reserved/unused)
0x001D    1       Time signature numerator
...       ...     (reserved/unused)
0x0021    1       Time signature denominator
...       ...     (reserved/unused)
0x03F0    4       Tempo (scaled, 32-bit BE)
```

## Reading the Header

1. Read magic number at `0x0000` - must be `0x3B9E`
2. Read PPQ at `0x0002` - used for timing calculations
3. Read time signature at `0x001D` (numerator) and `0x0021` (denominator)
4. Read tempo at `0x03F0` - divide by 10000 to get BPM

## Notes

- Header is fixed size and always starts at offset `0x0000`
- Tempo can have decimal precision (e.g., 109.0013 BPM)
- Time signature changes during song are stored in pattern data, not header
- Tempo changes during song are stored in special track "* New *"


# Time Signature

## Overview

Time signature information is stored in two places:
1. **Initial time signature** in file header (offsets `0x001D` and `0x0021`)
2. **Time signature changes** within pattern data as special events

## Initial Time Signature

### Numerator
- **Offset**: `0x001D`
- **Size**: 1 byte
- **Format**: Unsigned integer
- **Range**: 2-7 (common: 2, 3, 4, 5, 6, 7)
- **Purpose**: Beats per measure

### Denominator
- **Offset**: `0x0021`
- **Size**: 1 byte
- **Format**: Unsigned integer
- **Range**: 2, 4, 8, 16 (common: 2, 4, 8)
- **Purpose**: Note value (2=half note, 4=quarter note, 8=eighth note)

**Example**:
- Numerator = `4`, Denominator = `4` → 4/4 time signature
- Numerator = `6`, Denominator = `8` → 6/8 time signature

## Time Signature Changes

Time signature changes during the song are stored as **6-byte events** in pattern data, typically in the meta track "* New *".

### Event Format

Time signature changes use a **6-byte event** with specific markers:

```
Byte 0: 0xB7 (TS change marker)
Byte 1: 0x70 (Meta event type)
Byte 2-3: Lookup key (16-bit big-endian)
Byte 4: Flag (0x00 = valid, 0xFF = ignore)
Byte 5: Measure number - 1 (0-based)
```

### Event Structure

```
B7 70 [KEY_HI] [KEY_LO] [FLAG] [MEASURE-1]
 ↑  ↑     ↑        ↑       ↑        ↑
 │  │     └────────┴─ 16-bit lookup key
 │  └─ Meta event marker
 └─ TS change marker
```

**Critical**: Always check byte 4. If byte 4 = `0xFF`, this event must be ignored (it is an end marker or invalid change).

### Lookup Key

The 16-bit key (bytes 2-3) maps to a time signature via lookup table:

| Key (hex) | Numerator | Denominator | Verified |
|-----------|-----------|-------------|----------|
| 0x2100    | 5         | 4           | Yes      |
| 0x2280    | 3         | 4           | Yes      |
| 0x2400    | 6         | 8           | Yes      |
| 0x24C0    | 4         | 4           | Yes      |
| 0x2580    | 4         | 4           | Yes      |
| 0x2700    | 2         | 4           | Yes      |
| 0x2B80    | 4         | 4           | Yes      |
| 0x2DC0    | 5         | 4           | Yes      |
| 0x3C00    | 4         | 4           | Yes      |
| 0x9300    | 2         | 4           | Yes      |
| 0xD200    | 4         | 4           | Yes      |

**Total**: 11 keys decoded and verified

**Special Case**:
- Key `0x0000` with byte 4 = `0xFF` → End marker (ignore this event, not present in MIDI exports)

### Flag Byte (Byte 4)

- **0x00**: Valid time signature change
- **0xFF**: Invalid/end marker (ignore this event)

**Important**: Always check byte 4. If it is `0xFF`, skip this event.

### Measure Number (Byte 5)

- **Format**: 0-based measure number
- **Calculation**: `measure_number = byte5 + 1`
- **Purpose**: Specifies which measure the time signature change occurs

**Example**:
- Byte 5 = `0x03` → Measure 4 (3 + 1)
- Byte 5 = `0x13` → Measure 20 (19 + 1)

## Parsing Rules

### Critical: Byte 4 Flag Check

**Always check byte 4 before processing a TS change event:**

- **Byte 4 = `0x00`**: Valid time signature change - process normally
- **Byte 4 = `0xFF`**: Invalid/end marker - **ignore this event**

Events with byte 4 = `0xFF` are not present in MIDI exports and should be skipped during parsing.

### Paired Events (Optional)

Time signature changes may sometimes appear as **paired events**:

1. **Start marker**: `0x37 0x70` (beginning of TS change region)
2. **End marker**: `0xB7 0x70` (actual TS change)

Both events contain the same lookup key and measure number. The start marker (`0x37`) may indicate the beginning of a TS change region, while the end marker (`0xB7`) indicates the actual change. However, parsing should focus on `0xB7 0x70` events with byte 4 = `0x00`.

## Finding Time Signature Changes

1. **Locate Meta Track**: Find track named "* New *" (or "* Ne")
2. **Read Pattern Data**: Use pattern pointer from track header
3. **Scan for Events**: Look for `0xB7 0x70` pattern (bytes 0 and 1)
4. **Verify Flag**: Check byte 4 - **if `0xFF`, skip this event**
5. **Extract Key**: Read bytes 2-3 as 16-bit big-endian
6. **Lookup TS**: Use lookup table to find numerator/denominator
7. **Get Measure**: Calculate `measure = byte5 + 1`

**Important**: The byte 4 flag check is critical - events with byte 4 = `0xFF` are end markers and should not be processed as time signature changes.

## Example

**Hex Dump**:
```
B7 70 24 00 00 05
```

**Parsing**:
- Byte 0: `0xB7` → TS change marker ✓
- Byte 1: `0x70` → Meta event ✓
- Bytes 2-3: `0x2400` → Lookup key
- Byte 4: `0x00` → Valid (not 0xFF) ✓
- Byte 5: `0x05` → Measure 6 (5 + 1)

**Result**:
- Key `0x2400` → 6/8 time signature
- Change occurs at measure 6

## Lookup Table Implementation Notes

- Keys are 16-bit big-endian values
- Multiple keys can map to the same time signature (e.g., 4/4 has multiple keys: 0x24C0, 0x2580, 0x2B80, 0x3C00, 0xD200)
- Unknown keys should default to 4/4 or log a warning
- The lookup table was reverse-engineered from actual .SON files and verified by comparing with MIDI exports
- **Correction**: Key `0x2280` maps to 3/4 (not 6/4 as initially documented)

## Notes

- Initial TS is in header, changes are in pattern data
- TS changes are typically in the "* New *" meta track
- **Always check byte 4 flag before processing TS change** - if `0xFF`, skip the event
- Measure numbers are 1-based in display but 0-based in file (byte 5 = measure - 1)
- Events with byte 4 = `0xFF` are not present in MIDI exports and should be ignored
- The lookup table contains 11 verified keys covering common time signatures: 2/4, 3/4, 4/4, 5/4, 6/8


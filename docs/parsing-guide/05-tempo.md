# Tempo

## Overview

Tempo information is stored in two places:
1. **Initial tempo** in file header (offset `0x03F0`)
2. **Tempo changes** within pattern data in the "* New *" track

## Initial Tempo

### Tempo Offset
- **Offset**: `0x03F0`
- **Size**: 4 bytes (32-bit)
- **Format**: Big-endian signed integer
- **Encoding**: Scaled tempo value (BPM × 10000)
- **Calculation**: `tempo_bpm = tempo_value / 10000.0`

### Reading Initial Tempo

1. Read 4 bytes at offset `0x03F0` as big-endian 32-bit integer
2. Divide by 10000.0 to get BPM
3. Result can have decimal precision

**Examples**:
- `0x0010C8E0` (1100000) → 110.0 BPM
- `0x000E4E40` (920000) → 92.0 BPM
- `0x0010A8B1` (1090013) → 109.0013 BPM
- `0x0011F950` (1179984) → 117.9984 BPM

## Tempo Changes

Tempo changes during the song are stored in the **"* New *"** track (Pattern 1, Track 1).

### Event Format

Tempo changes use **6-byte events** with specific status bytes:

```
Byte 0: Status byte (varies: 0xDD, 0xF1, 0xFF, etc.)
Byte 1: 0x00 (reserved/padding)
Byte 2-3: Position in ticks (16-bit big-endian)
Byte 4-5: Additional data (depends on status byte)
```

### Status Bytes

Different status bytes indicate different tempo values or event types:

- **0xDD**: Tempo change (often 110 BPM or similar)
- **0xF1**: Tempo change (often 120 BPM or similar)
- **0xFF**: Standard MIDI meta event (Set Tempo)

**Note**: The exact mapping of status bytes to tempo values requires further analysis. Status bytes may encode tempo values directly or serve as markers.

### Position (Bytes 2-3)

- **Format**: 16-bit big-endian unsigned integer
- **Units**: Ticks (absolute time position)
- **Purpose**: When the tempo change occurs

**Calculation**: `absoluteTime = (byte2 << 8) | byte3`

### Finding Tempo Changes

1. **Locate Meta Track**: Find track named "* New *"
2. **Read Pattern Data**: Use pattern pointer from track header
3. **Scan for Events**: Look for events with:
   - Byte 1 = `0x00`
   - Byte 0 = tempo-related status (`0xDD`, `0xF1`, `0xFF`, etc.)
4. **Extract Position**: Read bytes 2-3 as position in ticks
5. **Determine Tempo**: Analyze bytes 4-5 or status byte to determine new tempo

## Tempo Change Event Structure

```
Event Format (6 bytes):
┌──────┬──────┬──────────┬──────────┐
│STATUS│ 0x00 │ POS_HIGH │ POS_LOW  │
└──────┴──────┴──────────┴──────────┘
   ↑      ↑        ↑          ↑
   │      │        └─ Position (16-bit)
   │      └─ Reserved/padding
   └─ Tempo change marker
```

## Example Tempo Change Events

### Example 1: Tempo Change at Bar 1
**Hex**: `DD 00 44 90 22 7F`

**Parsing**:
- Byte 0: `0xDD` → Tempo change marker
- Byte 1: `0x00` → Padding
- Bytes 2-3: `0x4490` → Position = 17552 ticks
- Bytes 4-5: `0x227F` → Additional tempo data

### Example 2: Tempo Change at Bar 2
**Hex**: `F1 00 89 20 45 3E`

**Parsing**:
- Byte 0: `0xF1` → Tempo change marker
- Byte 1: `0x00` → Padding
- Bytes 2-3: `0x8920` → Position = 35104 ticks
- Bytes 4-5: `0x453E` → Additional tempo data

## Tempo Calculation from Events

The exact method to calculate tempo from tempo change events requires further analysis. Possible approaches:

1. **Status Byte Encoding**: Status byte directly encodes tempo value
2. **Bytes 4-5 Encoding**: Bytes 4-5 contain tempo data
3. **Combined Encoding**: Tempo calculated from status byte + bytes 4-5

**Current Status**: Tempo change event format is partially decoded. Further analysis needed to determine exact tempo calculation method.

## Tempo Change Location

Tempo changes are typically found in:
- **Track**: "* New *" (or "* Ne")
- **Pattern**: Usually Pattern 1
- **Track Index**: Usually Track 1 (first track)

## Notes

- Initial tempo is always in header at `0x03F0`
- Tempo changes are in pattern data, not header
- Tempo changes use 6-byte event format
- Position is absolute time in ticks
- Status bytes (`0xDD`, `0xF1`, etc.) need further decoding
- Tempo can have decimal precision (e.g., 109.0013 BPM)
- Multiple tempo changes can occur throughout the song

## Verification

Tempo changes can be verified by:
1. Exporting .SON to MIDI
2. Comparing MIDI Set Tempo events (FF 51 03) with .SON tempo changes
3. Matching positions and tempo values

**Status**: Tempo change event format requires further reverse engineering to fully decode tempo calculation.


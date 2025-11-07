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

Time signature changes use **paired 6-byte events** (12 bytes total):

**First Event** (`0x37 0x70`):
```
Byte 0: 0x37 (TS change start marker)
Byte 1: 0x70 (Meta event type)
Byte 2-3: Lookup key (16-bit big-endian, for Notator internal use)
Byte 4: Velocity (contains TS numerator: numerator = (velocity - 1) / 2)
Byte 5: Reserved/padding
```

**Second Event** (`0xB7 0x70`):
```
Byte 0: 0xB7 (TS change end marker)
Byte 1: 0x70 (Meta event type)
Byte 2-3: Lookup key (same as first event)
Byte 4: Flag (0x00 = valid, 0xFF = ignore)
Byte 5: Measure number - 1 (0-based)
```

**Critical**: Always check byte 4 in the second event (`0xB7 0x70`). If byte 4 = `0xFF`, this pair must be ignored (it is an end marker or invalid change).

### Paired Events Structure

Time signature changes are stored as **paired 6-byte events**:

1. **Start marker**: `0x37 0x70` (first event)
2. **End marker**: `0xB7 0x70` (second event, 6 bytes after first)

Both events contain the same lookup key and measure number. The **velocity byte (byte 4) in the `0x37 0x70` event** contains the actual time signature numerator.

### Dynamic Decoding Algorithm

Time signature is decoded **algorithmically** from the velocity byte in the `0x37 0x70` event:

```
numerator = (velocity - 1) / 2
denominator = 4  // Notator always uses /4
```

**Examples**:
- Velocity = `7` → numerator = `(7-1)/2 = 3` → **3/4**
- Velocity = `5` → numerator = `(5-1)/2 = 2` → **2/4**
- Velocity = `9` → numerator = `(9-1)/2 = 4` → **4/4**
- Velocity = `11` → numerator = `(11-1)/2 = 5` → **5/4**
- Velocity = `13` → numerator = `(13-1)/2 = 6` → **6/4**

**Note**: The 16-bit lookup key (bytes 2-3) is used by Notator for internal identification but is **not needed for parsing** - the velocity byte contains the actual TS data.

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

**Always check byte 4 in the `0xB7 0x70` event before processing:**

- **Byte 4 = `0x00`**: Valid time signature change - process normally
- **Byte 4 = `0xFF`**: Invalid/end marker - **ignore this event**

Events with byte 4 = `0xFF` are not present in MIDI exports and should be skipped during parsing.

### Paired Events Parsing

Time signature changes **always** appear as **paired events**:

1. **First event**: `0x37 0x70` - Contains velocity byte (byte 4) with TS numerator
2. **Second event**: `0xB7 0x70` - Contains measure number (byte 5) and validation flag (byte 4)

**Parsing steps**:
1. Find `0x37 0x70` event (start marker)
2. Read velocity from byte 4: `velocity = event[4]`
3. Calculate numerator: `numerator = (velocity - 1) / 2`
4. Check next event (6 bytes later) is `0xB7 0x70`
5. Verify byte 4 in `0xB7 0x70` event is `0x00` (not `0xFF`)
6. Read measure number: `measure = event[11] + 1` (byte 5 of second event)

## Finding Time Signature Changes

1. **Locate Meta Track**: Find track named "* New *" (or "* Ne")
2. **Read Pattern Data**: Use pattern pointer from track header
3. **Scan for Paired Events**: Look for `0x37 0x70` followed by `0xB7 0x70` (6 bytes later)
4. **Read Velocity**: From `0x37 0x70` event, read byte 4 (velocity)
5. **Calculate TS**: `numerator = (velocity - 1) / 2`, `denominator = 4`
6. **Verify Flag**: Check byte 4 in `0xB7 0x70` event - **if `0xFF`, skip this pair**
7. **Get Measure**: Read byte 5 from `0xB7 0x70` event: `measure = byte5 + 1`

**Important**: 
- Always parse paired events together (12 bytes total: two 6-byte events)
- Skip initialization events where velocity = 0 or byte 4 in second event = `0xFF`
- The lookup key (bytes 2-3) is not needed for parsing - use velocity algorithm instead

## Example

**Hex Dump** (paired events, 12 bytes total):
```
37 70 24 00 0D 00  B7 70 24 00 00 05
```

**Parsing**:
1. **First event** (`0x37 0x70`):
   - Byte 0: `0x37` → Start marker ✓
   - Byte 1: `0x70` → Meta event ✓
   - Byte 4: `0x0D` → Velocity = 13
   - Calculate numerator: `(13 - 1) / 2 = 6`
   - Denominator: `4`
   - Result: **6/4 time signature**

2. **Second event** (`0xB7 0x70`):
   - Byte 6: `0xB7` → End marker ✓
   - Byte 7: `0x70` → Meta event ✓
   - Byte 10: `0x00` → Valid flag (not 0xFF) ✓
   - Byte 11: `0x05` → Measure 6 (5 + 1)

**Result**:
- Time signature: **6/4**
- Change occurs at measure 6

## Algorithm Implementation Notes

- **No lookup table needed**: Time signature is calculated algorithmically from velocity byte
- **Velocity range**: Typically 5-13 (for 2/4 to 6/4 time signatures)
- **Denominator**: Always 4 (Notator uses /4 as base denominator)
- **Paired events**: Always parse both `0x37 0x70` and `0xB7 0x70` events together (12 bytes)
- **Skip conditions**: Ignore pairs where velocity = 0 or byte 4 in second event = `0xFF`
- The lookup key (bytes 2-3) is generated by Notator for internal identification but is not needed for parsing

## Notes

- Initial TS is in header, changes are in pattern data
- TS changes are typically in the "* New *" meta track
- **Always parse paired events**: `0x37 0x70` + `0xB7 0x70` (12 bytes total)
- **Always check byte 4 flag** in `0xB7 0x70` event - if `0xFF`, skip the pair
- Measure numbers are 1-based in display but 0-based in file (byte 5 = measure - 1)
- Velocity byte (byte 4 in `0x37 0x70` event) contains the actual TS numerator
- Algorithm: `numerator = (velocity - 1) / 2`, `denominator = 4`
- Events with byte 4 = `0xFF` in second event are not present in MIDI exports and should be ignored


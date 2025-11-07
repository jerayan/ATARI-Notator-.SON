# Clefs

## Overview

Clef information in ATARI Notator .SON files is stored within pattern data as special events. Clefs determine the pitch range and note positioning for musical notation.

## Clef Event Format

Clef changes are stored as **6-byte events** in pattern data, similar to other meta events.

### Event Structure

```
Byte 0: Event identifier (varies)
Byte 1: Status byte (0x80 = Clef/key change marker)
Byte 2-3: Position (16-bit big-endian)
Byte 4: Clef type identifier
Byte 5: Additional parameter
```

### Status Byte

- **0x80**: Clef/key change marker
- **Purpose**: Indicates this event is a clef or key signature change

### Clef Types

Common clef types in musical notation:
- **Treble Clef** (G clef): Used for higher-pitched instruments
- **Bass Clef** (F clef): Used for lower-pitched instruments
- **Alto Clef** (C clef): Used for viola and some other instruments
- **Tenor Clef** (C clef): Used for cello and trombone

### Clef Type Identifier (Byte 4)

The exact encoding of clef types in byte 4 requires further analysis. Possible values:
- `0x00` - Treble clef
- `0x01` - Bass clef
- `0x02` - Alto clef
- `0x03` - Tenor clef
- (Other values may exist)

**Note**: The exact mapping of byte 4 values to clef types needs verification with actual .SON files.

## Finding Clef Events

1. **Scan Pattern Data**: Read through pattern events sequentially
2. **Look for Status**: Find events where byte 1 = `0x80`
3. **Check Context**: Verify this is a clef event (not another 0x80 event type)
4. **Extract Clef Type**: Read byte 4 to determine clef type
5. **Get Position**: Read bytes 2-3 to get position where clef change occurs

## Clef Change Event Example

**Hypothetical Event**:
```
80 80 00 C0 00 00
```

**Parsing**:
- Byte 0: `0x80` → Event identifier
- Byte 1: `0x80` → Clef/key change marker
- Bytes 2-3: `0x00C0` → Position = 192 ticks
- Byte 4: `0x00` → Clef type (possibly Treble)
- Byte 5: `0x00` → Additional parameter

## Clef and Key Signature

Clef events may be combined with key signature information:
- **Clef**: Determines note positioning
- **Key Signature**: Determines accidentals (sharps/flats)

The `0x80` status byte may indicate both clef and key signature changes.

## Position

Clef changes occur at specific positions within the pattern:
- **Format**: 16-bit big-endian unsigned integer
- **Units**: Ticks
- **Purpose**: When the clef change takes effect

## Notes

- Clef events use status byte `0x80`
- Clef type is encoded in byte 4 (exact mapping needs verification)
- Clef changes can occur at any position in the pattern
- Multiple clef changes can occur throughout a track
- Clef information affects note rendering in notation software
- The exact clef type encoding requires analysis of actual .SON files with clef changes

## Status

**Current Status**: Clef event format is partially documented. The exact encoding of clef types (byte 4) and the relationship to key signatures requires further reverse engineering.

**Recommendation**: Analyze .SON files that contain clef changes and compare with notation output to determine exact clef type encoding.


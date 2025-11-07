# Event Types

## Overview

All events in pattern data are **6 bytes** in length. This document describes the various event types and their byte-level structure.

## Common Event Format

```
Byte 0: Note/Event identifier
Byte 1: Status byte
Byte 2-3: Position (16-bit big-endian)
Byte 4: Velocity/Parameter
Byte 5: Duration/Argument
```

## Event Type Identification

Events are identified by the **status byte** (byte 1) and sometimes by the **note byte** (byte 0).

## Event Types

### Note On/Off Events

**Status**: `0x90` or `0x91`

**Structure**:
```
Byte 0: MIDI note number (0-127)
Byte 1: 0x90 or 0x91 (Note On)
Byte 2-3: Position in ticks (16-bit BE)
Byte 4: Velocity + 2 (if > 0, subtract 2 for actual velocity)
Byte 5: Duration or additional parameter
```

**Notes**:
- Velocity is stored as `actual_velocity + 2`
- If byte 4 = 0, this is a Note Off event
- If byte 4 > 2, this is a Note On event with velocity = byte4 - 2

### Pitch Wheel Events

**Status**: `0xE0`

**Structure**:
```
Byte 0: MIDI note or event identifier
Byte 1: 0xE0 (Pitch Wheel)
Byte 2-3: Position in ticks (16-bit BE)
Byte 4: Pitch wheel value
Byte 5: Additional parameter
```

### Meta Events

**Status**: `0x70`

**Structure**:
```
Byte 0: Event identifier (varies)
Byte 1: 0x70 (Meta event marker)
Byte 2: Type/position parameter
Byte 3: Parameter 1 (0x00 = standard, 0x80 = control)
Byte 4: Parameter 2 (sub-type: 0x03, 0x05, 0x09, 0x11)
Byte 5: Additional argument
```

**Common Patterns**:
- `[0x00, 0x03]` - Most common meta event (standard event)
- `[0x00, 0x05]` - Specific event type
- `[0x00, 0x09]` - Possible tempo-related
- `[0x00, 0x11]` - Possible tempo-related
- `[0x80, 0x00]` - Control flag

### Time Signature Change Events

**Status**: `0xB7` + `0x70`

**Structure**:
```
Byte 0: 0xB7 (TS change marker)
Byte 1: 0x70 (Meta event type)
Byte 2-3: Lookup key (16-bit BE)
Byte 4: Flag (0x00 = valid, 0xFF = ignore)
Byte 5: Measure number - 1 (0-based)
```

**See**: [Time Signature Changes](04-time-signature.md) for detailed information.

### Tempo Change Events

**Status**: Various (`0xDD`, `0xF1`, `0xFF`, etc.)

**Structure**:
```
Byte 0: Status byte (0xDD, 0xF1, 0xFF, etc.)
Byte 1: 0x00 (reserved/padding)
Byte 2-3: Position in ticks (16-bit BE)
Byte 4-5: Additional tempo data
```

**See**: [Tempo Changes](05-tempo.md) for detailed information.

### Clef/Key Change Events

**Status**: `0x80`

**Structure**:
```
Byte 0: Event identifier
Byte 1: 0x80 (Clef/key change marker)
Byte 2-3: Position in ticks (16-bit BE)
Byte 4: Clef type identifier
Byte 5: Additional parameter
```

**See**: [Clefs](06-clefs.md) for detailed information.

### End of Pattern Markers

**Status**: `0xFC` or `0xFD`

**Structure**:
```
Byte 0: 0x7F or event identifier
Byte 1: 0xFC (end marker) or 0xFD (section marker)
Byte 2-5: End position or padding
```

**Purpose**:
- `0xFC`: End of pattern data
- `0xFD`: Section boundary marker

### Structural Events

#### Bar Marker
- **Note**: `0x82`
- **Status**: `0x00`
- **Purpose**: Marks bar/measure boundaries

#### Section Boundary
- **Note**: `0x80`
- **Status**: `0x00`
- **Purpose**: Marks section boundaries

#### Padding Events
- **Note**: Various (`0xB7`-`0xBF`)
- **Status**: Varies
- **Purpose**: Padding or structural markers

## Event Reading Algorithm

1. **Read 6 bytes** from current offset
2. **Check byte 1** (status byte) to determine event type
3. **Parse event** based on event type
4. **Extract data** according to event structure
5. **Move to next event** (offset += 6)
6. **Stop** if end marker (`0xFC` or `0xFD`) encountered

## Event Type Decision Tree

```
Byte 1 (Status)?
├── 0x90/0x91 → Note On/Off
├── 0xE0 → Pitch Wheel
├── 0x70 → Meta Event
│   └── Byte 0 = 0xB7? → Time Signature Change
├── 0x80 → Clef/Key Change
├── 0xFC → End of Pattern
├── 0xFD → Section Marker
└── 0x00 → Structural Event (check Byte 0)
    ├── 0x82 → Bar Marker
    └── 0x80 → Section Boundary
```

## Position Values

Position (bytes 2-3) is always:
- **Format**: 16-bit big-endian unsigned integer
- **Range**: 0x0000 - 0xFFFF
- **Units**: Ticks
- **Context**: May be relative to pattern start or absolute, depending on implementation

## Notes

- All events are exactly 6 bytes
- Events are read sequentially
- Status byte (byte 1) is primary identifier
- Some events require checking byte 0 as well
- End markers terminate pattern reading
- Structural events may appear between musical events
- Event types can be mixed within a single pattern


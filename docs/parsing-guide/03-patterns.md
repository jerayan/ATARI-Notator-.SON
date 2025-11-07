# Patterns

## Overview

Patterns contain the actual musical event data (notes, meta events, etc.). Each track points to its pattern data via a pattern pointer stored in the track header.

## Pattern Data Structure

Pattern data consists of a sequence of **6-byte events**. Events are read sequentially until an end marker is encountered.

## Event Format

All events are **6 bytes** in length:

```
Byte 0: Note/Event identifier
Byte 1: Status byte
Byte 2-3: Position (16-bit big-endian)
Byte 4: Velocity/Parameter
Byte 5: Duration/Argument
```

### Position (Bytes 2-3)
- **Format**: 16-bit big-endian unsigned integer
- **Range**: 0x0000 - 0xFFFF
- **Units**: Ticks (relative to pattern start or absolute, depending on context)
- **Purpose**: Timing position of event within pattern

## Event Types

### Note On/Off Events
- **Status**: `0x90` or `0x91` (Note On)
- **Byte 0**: MIDI note number (0-127)
- **Byte 1**: `0x90` or `0x91`
- **Byte 2-3**: Position in ticks
- **Byte 4**: Velocity + 2 (if velocity > 0, subtract 2 to get actual velocity)
  - `velocity = byte4 - 2` if `byte4 > 2`
  - `byte4 = 0` indicates Note Off
- **Byte 5**: Duration or additional parameter

**Note**: Velocity is stored as `actual_velocity + 2`. A value of `0` in byte 4 indicates Note Off.

### Meta Events (0x70)
- **Status**: `0x70`
- **Byte 0**: Event identifier (varies)
- **Byte 1**: `0x70` (meta event marker)
- **Byte 2**: Type/position parameter
- **Byte 3**: Parameter 1 (0x00 = standard, 0x80 = control)
- **Byte 4**: Parameter 2 (sub-type: 0x03, 0x05, 0x09, 0x11)
- **Byte 5**: Additional argument

**Common Patterns**:
- `[0x00, 0x03]` - Most common meta event
- `[0x00, 0x05]` - Specific event type
- `[0x00, 0x09]` - Possible tempo-related
- `[0x00, 0x11]` - Possible tempo-related
- `[0x80, 0x00]` - Control flag

### Time Signature Change Events
See [Time Signature Changes](04-time-signature.md) section for details.

### End of Pattern Marker
- **Status**: `0xFC` or `0xFD`
- **Purpose**: Indicates end of pattern data
- **Byte 0**: `0x7F` or event identifier
- **Byte 1**: `0xFC` (end marker) or `0xFD` (section marker)
- **Bytes 2-5**: May contain end position or padding

### Structural Events
- **Bar Marker**: `NOTE=0x82, STATUS=0x00`
- **Section Boundary**: `NOTE=0x80, STATUS=0x00`
- **Padding Events**: Various NOTE values (0xB7-0xBF) with varying STATUS

## Reading Pattern Data

1. **Get Pattern Pointer**: Read from track header at offset `+0x14` (32-bit BE)
2. **Navigate to Offset**: Jump to absolute file offset specified by pattern pointer
3. **Read Events**: Read 6-byte events sequentially
4. **Check Status**: Examine byte 1 to determine event type
5. **Parse Event**: Extract data based on event type
6. **Continue**: Move to next event (offset += 6)
7. **Stop**: When end marker (`0xFC` or `0xFD`) is encountered

## Pattern Data Layout

```
Pattern Data (starts at pattern pointer)
├── Event 0 (6 bytes)
│   ├── Note/ID
│   ├── Status
│   ├── Position (16-bit)
│   ├── Velocity/Param
│   └── Duration/Arg
├── Event 1 (6 bytes)
│   └── ...
├── Event 2 (6 bytes)
│   └── ...
└── End Marker (0xFC or 0xFD)
```

## Position Calculation

Position values are stored as 16-bit big-endian integers representing ticks. The relationship to musical time depends on:
- PPQ value from header (typically 192)
- Time signature (affects beats per measure)
- Tempo (affects ticks per second)

**Example**:
- PPQ = 192
- Time signature = 4/4
- Position = 192 ticks = 1 quarter note = 1 beat

## Notes

- All events are exactly 6 bytes
- Events are read sequentially until end marker
- Position is relative to pattern start (or absolute, verify with your implementation)
- Velocity is stored as `actual_velocity + 2` for Note On events
- Pattern data can contain various event types mixed together
- End markers (`0xFC`, `0xFD`) terminate pattern reading


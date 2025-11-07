# Tracks

## Overview

Tracks contain MIDI channel assignments, names, and pointers to pattern data. Track headers are stored in a block starting around offset `0x1900` (varies by file).

## Track Header Structure

Each track header is **44 bytes** in length (exact size may vary, verify with your files).

### Track Header Layout

```
Offset (relative)  Size    Description
─────────────────────────────────────────────────────
+0x00              1       Track type (0x00 = MIDI track)
+0x01              1       Flags (reserved)
+0x02              1       Mute flag (0x01 = muted, 0x00 = unmuted)
+0x03              1       (reserved)
+0x04              1       (reserved)
+0x05              1       MIDI channel (1-16, 0-based: 0-15)
+0x06              10      Track name (ASCII, padded with spaces)
+0x10              4       (reserved/unused)
+0x14              4       Pattern pointer (32-bit big-endian)
...                ...     (remaining bytes)
```

### Track Name
- **Offset**: `+0x06` (relative to track header start)
- **Size**: 10 bytes
- **Format**: ASCII string, null-padded or space-padded
- **Purpose**: Human-readable track name

**Example:**
- `"Piano    "` (10 bytes, space-padded)
- `"* New *"` (8 bytes + 2 padding)

### MIDI Channel
- **Offset**: `+0x05` (relative to track header start)
- **Size**: 1 byte
- **Format**: Unsigned integer
- **Range**: 1-16 (stored as 0-15 internally)
- **Purpose**: MIDI channel assignment for track

**Note**: Some parsers may store channel as 1-based (1-16), others as 0-based (0-15). Verify with your implementation.

### Pattern Pointer
- **Offset**: `+0x14` (relative to track header start)
- **Size**: 4 bytes (32-bit)
- **Format**: Big-endian signed integer
- **Purpose**: Absolute file offset to pattern data for this track

**Important**: This is an **absolute offset** from the start of the file (0x0000), not relative to track header.

### Mute Flag
- **Offset**: `+0x02` (relative to track header start)
- **Size**: 1 byte
- **Format**: Boolean (0x01 = muted, 0x00 = unmuted)
- **Purpose**: Track mute state

## Finding Track Headers

### Method 1: Fixed Offset (Approximate)
Track headers typically start around offset `0x1900`, but this varies by file.

### Method 2: Track Count
The number of tracks must be determined by:
1. Scanning for track headers
2. Reading until end of track block
3. Counting valid track entries

### Method 3: Pattern Analysis
Track headers can be found by:
1. Locating pattern data
2. Working backwards to find track header block
3. Each track header contains pattern pointer

## Special Tracks

### Meta Track "* New *"
- **Name**: `"* New *"` (8 characters + padding)
- **Purpose**: Contains tempo changes and other meta events
- **Pattern**: Usually Pattern 1, Track 1
- **Events**: Contains tempo change events (see Tempo Changes section)

## Reading Tracks

1. Locate track header block (typically ~0x1900)
2. Read track header (44 bytes)
3. Extract:
   - Track name (bytes +0x06 to +0x0F)
   - MIDI channel (byte +0x05)
   - Pattern pointer (bytes +0x14 to +0x17, 32-bit BE)
   - Mute flag (byte +0x02)
4. Use pattern pointer to locate pattern data
5. Repeat for next track header (offset += 44)

## Track Header Block Structure

```
Track Header Block (starts ~0x1900)
├── Track 0 Header (44 bytes)
│   ├── Type, Flags, Mute
│   ├── MIDI Channel
│   ├── Track Name (10 bytes)
│   └── Pattern Pointer (32-bit)
├── Track 1 Header (44 bytes)
│   └── ...
├── Track 2 Header (44 bytes)
│   └── ...
└── ...
```

## Notes

- Track header size may vary - verify with your files
- Pattern pointers are absolute file offsets
- Track names are ASCII, may contain spaces or null bytes
- MIDI channel is typically 1-based (1-16) but stored as 0-based (0-15)
- The "* New *" track is special and contains meta events


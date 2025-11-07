# ATARI Notator .SON File Parsing Guide

This guide provides detailed byte-level information for parsing ATARI Notator .SON files. All offsets are hexadecimal unless otherwise noted.

## Table of Contents

1. [File Header Structure](01-file-header.md)
2. [Tracks](02-tracks.md)
3. [Patterns](03-patterns.md)
4. [Time Signature](04-time-signature.md)
5. [Tempo](05-tempo.md)
6. [Clefs](06-clefs.md)
7. [Event Types](07-event-types.md)

## Quick Reference

### Critical Offsets

| Element | Offset | Size | Description |
|---------|--------|------|-------------|
| Magic Number | 0x0000 | 2 bytes | File identifier (0x3B9E) |
| PPQ | 0x0002 | 2 bytes | Pulses Per Quarter (typically 192) |
| Time Signature Numerator | 0x001D | 1 byte | Initial time signature numerator |
| Time Signature Denominator | 0x0021 | 1 byte | Initial time signature denominator |
| Tempo | 0x03F0 | 4 bytes | Initial tempo (32-bit big-endian, scaled) |
| Track Headers Block | ~0x1900 | Variable | Track header table (varies by file) |

### Event Format

All events in pattern data are **6 bytes** in length:

```
Byte 0: Note/Event identifier
Byte 1: Status byte
Byte 2-3: Position (16-bit big-endian)
Byte 4: Velocity/Parameter
Byte 5: Duration/Argument
```

## Endianness

ATARI ST uses **Motorola 68000** processor with **big-endian** byte order. All multi-byte values must be read as big-endian.

## Data Types

- **8-bit**: Single byte (0x00 - 0xFF)
- **16-bit**: Two bytes, big-endian (0x0000 - 0xFFFF)
- **32-bit**: Four bytes, big-endian (0x00000000 - 0xFFFFFFFF)

## Notes

- All offsets are relative to the start of the file (0x0000)
- Track header offsets vary between files and must be calculated
- Pattern data offsets are stored in track headers as 32-bit pointers
- Time signature changes occur within pattern data as special events


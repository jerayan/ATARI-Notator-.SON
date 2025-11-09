# ATARI Notator .SON File Format Documentation

This repository contains comprehensive documentation about the ATARI Notator .SON file format, including byte structure, file format specifications, and reverse engineering notes.

## Overview

Notator SL was a MIDI sequencer for Atari ST computers, developed by C-LAB. The program used a pattern-oriented arrangement system where individual MIDI tracks were organized into patterns that could be arranged into complete compositions.

## Quick Start

### Parsing Guide

The most comprehensive documentation is in the **parsing-guide** directory:

- **[Parsing Guide Overview](docs/parsing-guide/README.md)** - Start here for byte-level parsing instructions
- **[File Header](docs/parsing-guide/01-file-header.md)** - Magic number, PPQ, tempo, time signature offsets
- **[Tracks](docs/parsing-guide/02-tracks.md)** - Track headers, MIDI channels, pattern pointers
- **[Patterns](docs/parsing-guide/03-patterns.md)** - Pattern data structure and 6-byte events
- **[Time Signature](docs/parsing-guide/04-time-signature.md)** - Initial TS and changes (algorithmic decoding)
- **[Tempo](docs/parsing-guide/05-tempo.md)** - Initial tempo and tempo changes
- **[Clefs](docs/parsing-guide/06-clefs.md)** - Clef information and changes
- **[Event Types](docs/parsing-guide/07-event-types.md)** - Complete reference of all event types
- **[Track Quantization](docs/parsing-guide/08-quantization.md)** - Track quantization lookup table

### Key Features Documented

- ✅ **File Header**: Magic number, PPQ, tempo (0x03F0), time signature (0x001D, 0x0021)
- ✅ **Tracks**: Header structure, MIDI channels, track names, pattern pointers
- ✅ **Patterns**: 6-byte event format, note events, meta events
- ✅ **Time Signature Changes**: Paired events (0x37 0x70 + 0xB7 0x70), algorithmic decoding
- ✅ **Tempo Changes**: Events in "* New *" track
- ✅ **Clefs**: Clef change events
- ✅ **Track Quantization**: Pattern `2c XX ?? YY` lookup table (11 values: 4-768 ticks)

## Repository Structure

```
├── README.md                           # This file
├── LICENSE                             # MIT License
├── .gitignore                          # Git ignore rules
├── docs/                               # Documentation files
│   ├── file-structure.md              # Detailed file structure
│   ├── byte-reference.md              # Byte-by-byte reference guide
│   ├── resources.md                   # External resources and links
│   └── parsing-guide/                 # Comprehensive parsing guide
│       ├── README.md                  # Parsing guide overview
│       ├── 01-file-header.md         # File header structure
│       ├── 02-tracks.md               # Track structure and parsing
│       ├── 03-patterns.md             # Pattern data and events
│       ├── 04-time-signature.md       # Time signature (initial + changes)
│       ├── 05-tempo.md                # Tempo (initial + changes)
│       ├── 06-clefs.md                # Clef information
│       └── 07-event-types.md          # All event types reference
│       └── 08-quantization.md          # Quantization
├── examples/                           # Example .SON files (if available)
└── tools/                             # Analysis tools and scripts
```

## Resources

### Official Sources
- [Notator/Creator SL Software](https://notator.atari.org/html/software)
- [Notator Mailing List](https://notator.atari.org/)

### Community Forums
- [AtariAge Forums](https://forums.atariage.com/)
- [OldComp.cz Forum](https://www.oldcomp.cz/viewtopic.php?f=91&t=189)

### Documentation Sites
- [AtariUpToDate](https://atariuptodate.de/en/8493/notator)

## Contributing

This repository aims to document the .SON file format comprehensively. Contributions are welcome, especially:
- Byte structure analysis
- File format specifications
- Example files (if legally shareable)
- Analysis tools and parsers

## License

This documentation is provided for educational and preservation purposes.


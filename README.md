# ATARI Notator .SON File Format Documentation

This repository contains comprehensive documentation about the ATARI Notator .SON file format, including byte structure, file format specifications, and reverse engineering notes.

## Overview

Notator SL was a MIDI sequencer for Atari ST computers, developed by C-LAB. The program used a pattern-oriented arrangement system where individual MIDI tracks were organized into patterns that could be arranged into complete compositions.

## File Format

The `.SON` file format is the native file format used by Notator SL to store song data, including:
- MIDI patterns
- Track information
- Arrangement data
- Song settings

## Repository Structure

```
├── README.md                 # This file
├── docs/                     # Documentation files
│   ├── file-structure.md    # Detailed file structure
│   ├── byte-reference.md    # Byte-by-byte reference
│   └── resources.md         # External resources and links
├── examples/                 # Example .SON files (if available)
└── tools/                   # Analysis tools and scripts
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


# Byte-by-Byte Reference Guide

This document provides a detailed byte-level reference for the ATARI Notator .SON file format.

## Analysis Methods

### Tools for Analysis
- Hex editors (HxD, Hex Fiend, etc.)
- Binary file analyzers
- Custom parsing scripts

### Recommended Approach
1. Open .SON files in a hex editor
2. Compare multiple files to identify patterns
3. Document consistent byte positions
4. Test modifications to verify findings

## Known Byte Positions

### To Be Documented

This section will be populated as byte positions are identified and verified.

## Data Types

Based on Atari ST architecture:
- **16-bit integers**: Big-endian format
- **32-bit integers**: Big-endian format
- **Strings**: Null-terminated or length-prefixed (to be determined)

## Endianness

Atari ST used Motorola 68000 processor with big-endian byte order.

## Contributing Byte Information

When documenting bytes, please include:
- Offset (hex and decimal)
- Size in bytes
- Data type
- Description
- Example values
- Verification status


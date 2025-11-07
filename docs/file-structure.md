# ATARI Notator .SON File Structure

## File Header

The file begins with a header section that identifies the file format and version.

### Header Structure (Preliminary)

| Offset | Size | Description | Notes |
|--------|------|-------------|-------|
| 0x00   | 4    | Magic number / Signature | To be documented |
| 0x04   | ?    | Version information | To be documented |
| ...    | ...  | ... | ... |

## Pattern Data

Patterns contain MIDI event data organized by tracks.

## Arrangement Data

The arrangement section defines how patterns are sequenced in the song.

## Notes

This document is a work in progress. Detailed byte-level documentation will be added as information is gathered and verified.

## How to Contribute

If you have information about the file structure:
1. Analyze .SON files using hex editors
2. Document byte offsets and their meanings
3. Test findings with different .SON files
4. Submit findings via issues or pull requests


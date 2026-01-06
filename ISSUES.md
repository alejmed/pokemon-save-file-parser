# Pokemon Save File Parser - ISSUES & FINDINGS

**Updated**: January 6, 2026  
**Status**: Active Investigation - Move Parsing Discrepancy

---

## Current Issue: Move Parsing Incorrect

### What's Happening
Parser output shows INCORRECT moves compared to in-game reality.

### Example: Level 10 Charmander

**User Reports (Actual in-game):**
1. Metal Claw
2. Scratch
3. Growl
4. Ember

**Parser Shows (From save file):**
1. Flamethrower (ID: 52)
2. Scratch (ID: 10)
3. Low Kick (ID: 45)
4. None (ID: 0)

**Problem: Parser is COMPLETELY WRONG about moves!**

---

## Raw Data Analysis

### Current Save State (After 2nd save)
```
PID: 0xDCA1D9A6
OT ID: 0x16EF52DC
PID % 24: 14 → Block order: [2, 1, 0, 3]
Decryption seed: 0xCA4E8B7A
```

### XOR Decryption Check
After XOR decryption (before unshuffle):

**Block 0:**
- Word 0: 9472 (0x2500)
- Word 1: 10753 (0x2A01)
- Word 2: 0 (0x0000)
- Word 3: 0 (0x0000)
- Word 4: 0 (0x0000)
- Word 5: 0 (0x0000)

**Block 1:**
- Word 0: 232 (0x00E8) ← Invalid move ID!
- Word 1: 10 (0x000A) ← Scratch ✓
- Word 2: 45 (0x002D) ← Low Kick ✓
- Word 3: 52 (0x0034) ← Flamethrower
- Word 4: 1811 (0x0713)
- Word 5: 40 (0x0028)

**Block 2:**
- Word 0: 4 (0x0004)
- Word 1: 0 (0x0000)
- Word 2: 2534 (0x09E6)
- Word 3: 0 (0x0000)
- Word 4: 30208 (0x7600)
- Word 5: 0 (0x0000)

**Block 3:**
- Word 0: 22528 (0x5800)
- Word 1: 8709 (0x2205)
- Word 2: 24602 (0x601A)
- Word 3: 5223 (0x1467)
- Word 4: 0 (0x0000)
- Word 5: 0 (0x0000)

### Checksum Status
```
Calculated: 50688 (0xC600)
Stored: 50687 (0xC5FF)
Match: FALSE (Difference: 1)
```
Checksum is off by 1, suggesting close but not exact decryption.

---

## Move IDs We're Looking For

### Charmander Level 10 Learnset
- **Metal Claw** - Learned at Level 10
- **Scratch** - Starting move
- **Growl** - Starting move
- **Ember** - Learned at Level 7

### Expected Move IDs (if valid)
- Metal Claw: Need to verify ID (possibly ID: 246)
- Scratch: 10 ✓
- Growl: 94 ✓
- Ember: 50 ✓

---

## Known Issues

### 1. Invalid Move ID: 232
After XOR decryption, we're seeing move ID 232, which is OUTSIDE valid Gen 3 range (0-354).

This suggests:
- Decryption key is INCORRECT
- XOR algorithm has an error
- Block structure interpretation is wrong

### 2. Block Shuffle Order Might Be Wrong
Current implementation:
```python
shuffle_order = BLOCK_ORDER[pid % 24]
unshuffled = [bytearray(12) for _ in range(4)]
for i, block_idx in enumerate(shuffle_order):
    unshuffled[i] = decrypted_blocks[block_idx]
```

**Question**: Are we applying the shuffle order correctly?
- Block order [2, 1, 0, 3] means:
  - Position 0 gets Block 2
  - Position 1 gets Block 1
  - Position 2 gets Block 0
  - Position 3 gets Block 3

But maybe we have this backwards?

### 3. Checksum Mismatch
Calculated checksum differs from stored by exactly 1.

This indicates:
- Decryption is VERY close but not exact
- Might be a byte-swapping issue
- Or endianness problem

---

## Critical Observation

### The Moves Are NOT Where Expected

If we look at ALL blocks, do we see Metal Claw, Growl, Ember anywhere?

**Searching raw data for valid move IDs:**
- Metal Claw: ~246 (0x00F6)
- Growl: 94 (0x005E)
- Ember: 50 (0x0032)

Looking at all block words after XOR:
- Block 0: 0x2500, 0x2A01, 0x0000... (no match)
- Block 1: 0x00E8, 0x000A, 0x002D, 0x0034... (no match)
- Block 2: 0x0004, 0x0000, 0x09E6, 0x0000... (no match)
- Block 3: 0x5800, 0x2205, 0x601A, 0x1467... (no match)

**None of the words match Metal Claw (246/0x00F6), Growl (94/0x005E), or Ember (50/0x0032)!**

This means either:
1. XOR key is completely wrong
2. Decryption algorithm is wrong
3. Data is in different format than expected

---

## What to Investigate Next

### 1. Verify PKHeX Block Shuffle Logic
Review exact PKHeX implementation of block shuffling. Check if:
- We're using correct byte order
- We're interpreting shuffle order array correctly
- Blocks should be copied differently

### 2. Alternative Decryption Methods
Try:
- Different XOR key calculations
- Byte-level XOR instead of word-level
- Checksum-based decryption detection

### 3. Move Location Verification
Confirm moves are at offsets 0x2C, 0x2E, 0x30, 0x32 in party data.

PKHeX shows:
```csharp
Move1 = 0x2C, Move2 = 0x2E, Move3 = 0x30, Move4 = 0x32
```

### 4. Compare with Real PKHeX Output
Run actual PKHeX on the save file and see what it extracts. Compare byte-by-byte with our implementation.

---

## Charmander Level 10 Normal Learnset

**Standard FireRed Charmander moves:**
| Level | Move | Move ID |
|--------|-------|----------|
| 1 | Scratch | 10 |
| 1 | Growl | 94 |
| 7 | Ember | 50 |
| 10 | Metal Claw | 246 |
| 32 | Flamethrower | 52 |
| 39 | Slash | 163 |
| 46 | Fire Spin | 172 |

**User should NOT have Flamethrower at Level 10!**

If user says Charmander has Metal Claw, Scratch, Growl, Ember, then:
- Parser showing Flamethrower is WRONG
- Parser showing Low Kick is WRONG
- Parser showing move IDs 232, 52, 10, 45 is WRONG

This is a MAJOR parsing failure.

---

## Action Items

1. ✅ Document move parsing discrepancy
2. ⏳ Compare PKHeX source code more carefully
3. ⏳ Try running actual PKHeX on this save file
4. ⏳ Check if decryption should be byte-wise instead of word-wise
5. ⏳ Verify block unshuffle logic step-by-step
6. ⏳ Check if we're reading from correct offsets

---

## Related Files

- `pokemon_parser.py` - Current implementation (HAS BUG)
- `AGENTS.md` - Original technical documentation
- `ISSUES.md` - This file (active bug tracking)

---

## Questions for Future Investigation

1. Why is XOR checksum off by exactly 1?
2. Where are the actual move IDs (246, 10, 94, 50) in the data?
3. Is block shuffle order inverted in our implementation?
4. Should we be decrypting byte-by-byte instead of word-by-word?
5. Are we reading from correct section/block in save file?
6. Is this a different save format than standard FireRed?

---

**Last Updated**: Jan 6, 2026 - 11:00 PM  
**Status**: Critical bug - Moves are being incorrectly parsed

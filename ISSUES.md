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

---

## Update: January 6, 2026 - 11:10 PM (3rd Save Check)

### Latest Save: Charmander Level 15

**In-Game Moves (User Reports):**
- Metal Claw (Level 10 learn)
- Growl (starting move)
- Ember (Level 7 learn)
- Plus learned more by Level 15

**Parser Output (WRONG):**
1. Frustration (ID: 232) ← INVALID ID!
2. Scratch (ID: 10)
3. Low Kick (ID: 45)
4. Flamethrower (ID: 52)

### Key Observations

1. **PID and OT ID unchanged** (as expected, same Pokemon)
   - PID: 0xDCA1D9A6
   - OT ID: 0x16EF52DC

2. **Raw encrypted values CHANGED from previous saves:**
   - Move 1: 35730 (0x8B92) → 35730 (0x8B92) (unchanged)
   - Move 2: 51780 (0xCA44) → 51780 (0xCA44) (unchanged)
   - Move 3: 35671 (0x8B57) → 35671 (0x8B57) (unchanged)
   - Move 4: 51834 (0xCA7A) → 51834 (0xCA7A) (unchanged)

   ⚠️ RAW DATA HASN'T CHANGED! Yet game shows different moves!

3. **After XOR decryption (before unshuffle):**
   - Move 1: 232 (0x00E8) ← PERSISTENT!
   - Move 2: 10 (0x000A) ✓
   - Move 3: 45 (0x002D) ← Changed!
   - Move 4: 52 (0x0034) ← Changed!

4. **Checksum analysis:**
   - Calculated: 50688 (0xC600)
   - Stored: 50687 (0xC5FF)
   - Difference: 1

   ⚠️ Same checksum offset as before!

### Critical Finding

**Raw save data is NOT changing between saves, yet the game shows different moves!**

This suggests one of:
1. **We're looking at the WRONG SAVE BLOCK**
   - Save A (0x000000) vs Save B (0x00E000)
   - Game alternates between them
   - We might be reading an outdated save block

2. **We're in the WRONG SECTION**
   - Section 1 is correct for team data
   - But maybe different Pokemon slot?

3. **Game is using memory, not save file**
   - In-game data is in RAM
   - Save file is only updated periodically
   - User might be seeing RAM data that hasn't flushed to disk

4. **Decryption is fundamentally wrong**
   - Raw data is encrypted
   - XOR key might be incorrect
   - Block unshuffle might be wrong

### Test Required

**Check both save blocks:**
```bash
# Read Save A (offset 0x000000)
# Read Save B (offset 0x00E000)
# Find which one was updated most recently
```

**Check save index from footer:**
```python
save_index = read_u32(section_offset + 0x0FFC)
# Compare Save A vs Save B
# Use the one with higher index (more recent)
```

### Hypothesis

**Most likely: We're reading the OLD save block**

Game uses 2 save blocks and alternates. We're always finding section 1 in Save A (offset 0x000000), but Save B (offset 0x00E000) might be the most recent one.

**Action item: Modify parser to check BOTH save blocks and use the one with higher save index.**


---

## Update: January 6, 2026 - 11:25 PM (4th Save Check)

### DISCOVERY: Species ID Found at Wrong Offset

**Finding:**
- Species ID 5 (CharMELEON) encrypted as 0x8B7F found at offset 0x18
- This is in UNENCRYPTED section (OT Name field: offsets 0x14-0x1A)
- Species ID should be at offset 0x20 (start of encrypted section)

### Evidence

```
Offset 0x18 (in OT Name): 0x7FD3 (32723 decimal)
Species ID 5 XOR lower 16 bits of (PID ^ OT ID) = 35711 (0x8B7F) ✓ MATCH!
```

### Problem Identified

**The species ID is NOT at offset 0x20!**

Current hex dump shows:
- Offset 0x20: 0x8B7A4F (encrypted species)
- Offset 0x22: 0x8B4ECA (encrypted held item)

But species ID 5 is found at offset 0x18, which is PART OF OT NAME FIELD!

This means:
1. **My Pokemon structure understanding is WRONG**
2. **Species ID might be stored at different location in party data**
3. **PKHeX party structure differs from what I'm assuming**

### PKHeX Party vs PC Structure

According to AGENTS.md:
- **SIZE_3PARTY = 100** (Party Pokemon)
- **SIZE_3STORED = 80** (PC Box Pokemon)

**Key difference:**
- Party Pokemon have EXTRA 20 bytes (battle stats)
- PC Pokemon are 80 bytes only

**But both use same basic structure!**

### Hypothesis

**In PARTY data, species ID might be at DIFFERENT OFFSET!**

Or maybe:
- The encrypted section starts at different offset in party data
- The block structure is different
- I need to review PKHeX.PK3.cs more carefully

### Critical Finding

**The XOR decryption IS working correctly!**
- Species ID 5 XORed with correct seed appears in data
- At wrong offset, but still found

This means:
- Decryption algorithm is correct
- XOR key is correct
- Block unshuffle might be correct
- BUT I'm looking at wrong structure or offset

### Required Research

1. ✅ Re-read PKHeX PK3.cs - Check exact field offsets for PARTY Pokemon
2. ✅ Compare SIZE_3PARTY vs SIZE_3STORED definitions
3. ✅ Check if species ID is at different offset in party vs stored
4. ✅ Verify battle stats section (offsets 0x50-0x63) are correct

### Update Status

✅ Decryption verified as working
❌ Pokemon structure/offsets need verification
❌ Moves still showing wrong values


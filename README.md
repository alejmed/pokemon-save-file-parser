# Pokemon Save File Parser

Complete parser for Pokemon Fire Red/Leaf Green (Generation 3) save files based on PKHeX implementation.

---

## Quick Start

```bash
# Clone repository
git clone https://github.com/alejmed/pokemon-save-file-parser.git
cd pokemon-save-file-parser

# Parse your save file
python3 pokemon_parser.py path/to/pokemon.sav
```

---

## File Organization

### Documentation Files

| File | Purpose | When Updated |
|-------|----------|--------------|
| **CURRENT-STATE.md** | Latest Pokemon team data from your most recent save | Every time you ask for "latest" updates |
| **AGENTS.md** | Technical documentation and quick reference for parser | When parser needs updates/research |
| **ISSUES.md** | Active bug investigations and debugging notes | When discovering/fixing bugs |
| **README.md** | This file - repository overview and usage | When repository structure changes |

### Main Script

| File | Purpose |
|-------|----------|
| **pokemon_parser.py** | Main parser script - extracts complete Pokemon data |

---

## How Files Are Used

### CURRENT-STATE.md
**Purpose**: Store your current Pokemon team state

**When Updated**: Each time you ask for "latest" or "current" Pokemon information

**Contents**:
- Current Pokemon species, level, nature
- Battle stats (HP, Attack, Defense, Speed, Sp. Attack, Sp. Defense)
- EVs (Effort Values) - all 6 stats
- IVs (Individual Values) - all 6 stats
- Moves (what parser shows)
- Other info: Experience, Friendship, Held Item, PID, OT ID

**How I Use It**:
1. Read FIRST before running parser - see what's stored
2. Check timestamp at top - is it recent?
3. Note any discrepancies with your in-game reality
4. Update with new data after parsing
5. Focus on what changed - not repeating entire dump

**Example Response**:
```
EVs - CharMELEON (Level 19)
HP: 0
Attack: 70
Defense: 1
Speed: 78
Sp. Attack: 0
Sp. Defense: 0
```

### AGENTS.md
**Purpose**: Complete technical documentation for Pokemon Gen 3 save files

**When Updated**: When researching new algorithms or updating parser logic

**Contents**:
- Save file structure (128KB, two save blocks, 14 sections)
- Pokemon data structure (100 bytes per Pokemon)
- Byte offsets for all fields
- Encryption/decryption algorithms (PKHeX-based)
- Block shuffling logic
- Species ID mappings (internal vs National Dex)
- Move database (all 354 Gen 3 moves)
- Nature calculations
- Shiny determination formula

**How I Use It**:
1. Read FIRST before making any code changes
2. Verify byte offsets are correct
3. Check PKHeX implementation details
4. Reference decryption algorithms
5. Look up move/ability/species IDs

**Critical Sections**:
- ⚠️ "CRITICAL ISSUES & REMINDERS" at top - always check for active bugs
- Debugging checklist (7 steps to verify before assuming code is correct)
- Known limitations (nickname encoding, PP calculation, etc.)

### ISSUES.md
**Purpose**: Track active bugs, discoveries, and debugging notes

**When Updated**: When finding problems or making discoveries about data structure

**Contents**:
- Move parsing discrepancy (parser shows wrong moves)
- Raw data analysis and checksum calculations
- Block structure discoveries
- Decryption verification results
- Timestamps of each save check
- Hypotheses and test results

**How I Use It**:
1. Document discrepancies when you report them
2. Add raw data analysis for context
3. Track what I've tried and results
4. Note discoveries about data structure
5. Keep debugging checklist at the top

**Important**: This is where I document that parser might be wrong, not where I assume it's correct without verification.

---

## Current Status

### Working Features
- Complete Pokemon team extraction
- All battle stats (HP, Attack, Defense, Speed, Sp. Attack, Sp. Defense)
- EVs (Effort Values) for all 6 stats
- IVs (Individual Values) with quality ratings
- Nature, experience, friendship, held item
- XOR decryption algorithm (verified correct)
- Block shuffling logic
- Save file navigation (section finding)
- Checksum verification

### Known Issues
- Move parsing showing incorrect moves (see ISSUES.md for details)
- Nickname encoding not implemented
- OT name encoding not handled
- Ability lookup needs species-specific data
- Held item names not complete
- PP calculation (base PP + bonuses) not implemented

---

## Pokemon Save File Format

### File Structure
- **Size**: 131,072 bytes (128KB)
- **Format**: Two save blocks (A and B) at offsets 0x000000 and 0x00E000
- **Section Size**: 4KB (0x1000 bytes) per section
- **Sections per Save**: 14 sections
- **Team Data**: Section 1 (contains player's Pokemon team)

### Pokemon Team Location
- **Section ID**: 1
- **Team Size**: 1 byte at `Section 1 + 0x34`
- **Pokemon List**: 600 bytes starting at `Section 1 + 0x38` (100 bytes per Pokemon × 6)

---

## Requirements

- Python 3.6+
- Pokemon Fire Red/Leaf Green save file (.sav)

---

## Repository

https://github.com/alejmed/pokemon-save-file-parser

---

## License

Educational/Research purposes. Based on PKHeX (MIT License).

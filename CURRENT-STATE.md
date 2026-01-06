# Current Pokemon Team State

**Purpose**: Store current Pokemon team data for quick reference when you ask for "latest updates"

**File Type**: Output log - Write-only, updated by parsing script

---

## Current State

**Last Updated**: January 6, 2026 - 11:35 PM

### Charmeleon (Level 19)

**Battle Stats**
- HP: 52/55
- Attack: 32
- Defense: 34
- Speed: 41
- Sp. Attack: 36
- Sp. Defense: 27

**EVs (Effort Values)**
- HP: 0 / 255
- Attack: 154 / 255
- Defense: 1 / 255
- Speed: 147 / 255
- Sp. Attack: 0 / 255
- Sp. Defense: 0 / 255
- **TOTAL: 303 / 510 (59% of max)**

**IVs (Individual Values)**
- HP: 26 (Excellent!)
- Attack: 0 (Poor)
- Defense: 24 (Great!)
- Speed: 14 (Average)
- Sp. Attack: 6 (Poor)
- Sp. Defense: 10 (Average)
- Ability Bit: 0

**Moves**
1. Scratch (ID: 10)
2. Frustration (ID: 232)
3. Low Kick (ID: 45)
4. Flamethrower (ID: 52)

**Other**
- Nature: Lax
- Experience: 4,601
- Friendship: 132
- Held Item: None
- Species ID: 5
- PID: 0xDCA1D9A6
- OT ID: 21,212

---

## How This File is Used

### When You Ask for "Latest" or "Current State"

ALWAYS read this file FIRST before parsing:
1. Check if data is recent (timestamp at top)
2. Compare with your actual in-game stats if needed
3. Note discrepancies (e.g., moves not matching)
4. Use as baseline for detecting changes

### Response Format

When you ask for latest updates:
1. Brief summary of current Pokemon
2. Key stats (level, HP, main EVs/IVs)
3. Notable changes from previous saves
4. Clean formatting - no tables if terminal doesn't render well
5. Focus on what's new - you already know to rest

---

## Context for Troubleshooting

### Known Issue: Move Parsing

**Problem**: Parser shows wrong moves (Frustration, Low Kick, Flamethrower)

**In-Game Reality** (you reported):
- Charmeleon evolved from Charmander at Level 16
- Has Metal Claw, Growl, Ember (Level 7 learnset)

**Current Parser Output**:
- Shows: Scratch, Frustration, Low Kick, Flamethrower
- These moves don't match Charmeleon learnset

**Important**: When you ask for updates, I should:
1. Check CURRENT-STATE.md first
2. Note any discrepancies with in-game reality
3. Document raw data analysis in ISSUES.md with timestamp
4. Update AGENTS.md with debugging checklist
5. Verify with raw data analysis before changing code

---

## Nuances to Remember

### Response Guidelines

**When user asks for "latest"**:
- Keep it concise - they already know to basics
- Focus on what changed since last request
- If terminal format issues with tables, use simple list format
- Don't show entire dump - extract just what's requested (e.g., "only EVs", "just moves")

**When user reports discrepancies**:
- Trust user's in-game observations over parser output
- Document raw data analysis in ISSUES.md
- Check CURRENT-STATE.md FIRST before running parser
- Compare timestamps to track save progression

**When user wants only specific info**:
- Don't show entire parser output
- Extract just what's requested (e.g., "only EVs", "just moves")
- Keep it brief and focused

### File Reading Workflow

**ALWAYS READ IN THIS ORDER**:
1. CURRENT-STATE.md - Latest Pokemon data
2. ISSUES.md - Active bugs and investigations
3. AGENTS.md - Technical documentation only when needed

This ensures I have context before parsing or making changes.

---

## Notes

- This file is OVERWRITTEN each time you ask for latest updates
- Keep timestamps to track save progression
- Use AGENTS.md for technical details, not CURRENT-STATE.md
- Use ISSUES.md for problems, not CURRENT-STATE.md

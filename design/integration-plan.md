# Integration Plan: Cuphead Host + Mega Man SFR Companion

## Architecture

### Host Game: Cuphead
- **Engine:** Unity (C#)
- **Mod Route:** Cuphead Mod Loader (DLL injection)
- **Asset Store:** Unity `.assets` and `.resource` files (encrypted/compressed)
- **Modification Points:** 
  - Player character model and behavior (replace with Mega Man)
  - Weapon systems (inject Mega Man weapon selection and firing)
  - Audio track loading (hook audio streamer to read from companion folder)
  - Boss encounter setup (use existing Cuphead boss phases, rebalance HP)

### Companion Game: Mega Man SFR
- **Engine:** GameMaker: Studio
- **Asset Location:** Monolithic `.exe` + standalone BGM folder
- **Shared Content:**
  - BGM folder: 59 OGG tracks (directly accessible, no extraction needed)
  - Weapon metadata: Read from game executable if available, or hardcode weapon stats
  - Robot master list: Use OGG filenames as reference

## Content Reading Strategy

### Mega Man SFR Music (Runtime)
```
Melty passes {game:mega-man-sfr} folder path at launch
→ Mashup reads from: {game:mega-man-sfr}/BGM/*.ogg
→ No files copied; each player uses their own copy
```

### Cuphead Assets (Build Time + Runtime)
- Extract and analyze Cuphead's `.assets` files to identify:
  - Boss sprite animations and hitboxes
  - Boss health values and phase transitions
  - Boss attack patterns (projectile timing, spawning)
  - Audio asset references
- Read from player's installed copy at runtime via Cuphead Mod Loader

## First Playable Version (0.1.0-alpha)

### Scope
- **1 stage** with 2 sequential boss encounters
- **Bosses:** Goopy Le Grande → Hilda Berg (Cuphead W1)
- **Player:** Mega Man character model (asset-swap or sprite overlay)
- **Weapons:** 4 weapon slots (Axe, Coal, Dagger, Leaf) with basic firing
- **Music:** Boss.ogg (Mega Man SFR) during fights, GameOver/GameClear on defeat/win
- **Controls:** Arrow keys / gamepad (inherit from Cuphead)
- **Win Condition:** Defeat all bosses in sequence → "Run Complete" screen
- **Lose Condition:** 0 health → Game Over screen with retry option

### What's NOT in 0.1.0
- Multiple stages or stage select
- Weapon ammo / resource management UI
- Robot master stage select screen
- Parry mechanic from Cuphead
- Co-op multiplayer
- Difficulty modes

## Technical Implementation Steps

1. **Extract Cuphead Data**
   - Use UABE or unity-parse to open `sharedassets*.assets` files
   - Locate Goopy Le Grande and Hilda Berg boss prefabs
   - Extract: health values, sprite frames, attack timing, audio references
   - Save as JSON data files in project

2. **Set Up Mod Loader**
   - Download Cuphead Mod Loader from GameBanana or Nexus Mods
   - Verify DLL injection into Cuphead.exe
   - Create mod folder structure: `Cuphead/Mods/CupheadMegaManMashup/`

3. **Implement Mega Man Player**
   - Override Cuphead's player character controller
   - Implement dash (forward/backward), jump, fall velocity
   - Connect to Cuphead's existing input system
   - Test movement in an existing Cuphead boss arena

4. **Implement Weapon System**
   - Create weapon data structure (damage, fire rate, ammo, spread)
   - Implement weapon swap (1-4 keys or bumpers)
   - Create projectile prefab with configurable properties
   - Load Mega Man SFR weapon themes on swap (audio hooks)

5. **Audio Integration**
   - Hook Cuphead's AudioListener to read from companion folder
   - Set up file path mapping: {game:mega-man-sfr}/BGM/Boss.ogg → runtime load
   - Test audio playback without stuttering or sync loss

6. **Build and Test**
   - Package mod as release archive with melty.json recipe
   - Test on PC with both games installed
   - Verify: Mega Man appears, weapons fire, bosses take damage, music plays, boss defeated → next boss
   - Capture gameplay video/screenshot

## Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Cuphead assets are heavily encrypted | Use established mod loader; community tools handle decryption |
| Weapon balance may be too hard/easy | Tune damage/fire-rate values in JSON before each build |
| Music sync issues with boss intro/outro | Pre-load tracks; test with actual boss transitions |
| Mega Man SFR music folder may not exist | Melty recipe checks for companion game at install; stops with error if missing |
| Player can't reach boss hitbox with Mega Man movement | Adjust Mega Man's jump height or boss arena collision layer |

## Success Criteria for 0.1.0

- [ ] Mega Man appears in Cuphead boss arena instead of Cuphead
- [ ] Jump and dash work with Mega Man model
- [ ] Weapon swap UI shows 4 weapon slots
- [ ] Firing a weapon creates visible projectiles that damage Goopy Le Grande
- [ ] Goopy Le Grande defeated → Hilda Berg spawns
- [ ] Mega Man SFR Boss.ogg plays during fight
- [ ] Defeat Hilda Berg → Game Clear screen with music
- [ ] Lose all health → Game Over screen
- [ ] Melty recipe references both games correctly
- [ ] Build can be tested on any PC with both games installed

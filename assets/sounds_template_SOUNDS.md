# SOUNDS.md — Audio Library (CC0)

All sounds are pre-copied to `assets/sounds/` by the pipeline.
Load them via `AssetManager` — never use `new Sound()` or `new Music()` directly.

---

## Music tracks — `assets/sounds/music/`

### Core tracks (always copied to every project)

| File | Style | Use for |
|------|-------|---------|
| `sounds/music/music_menu.ogg` | Chiptune 8-bit | Main menu, leaderboard, settings |
| `sounds/music/music_gameplay.ogg` | Chiptune 8-bit | In-game loop (runners, puzzles, general) |
| `sounds/music/music_gameplay_alt.ogg` | Chiptune 8-bit | Tension / boss fights / alternate gameplay |
| `sounds/music/music_gameplay_space.ogg` | Ambient electronic | Space shooters, sci-fi games |
| `sounds/music/music_game_over.ogg` | Chiptune sting | Game over screen — plays ONCE, no loop |

### Junkala Retro Pack (Juhani Junkala, CC0)

| File | Style | Use for |
|------|-------|---------|
| `sounds/music/music_title.ogg` | Chiptune 8-bit energetic | Title screen / main menu variant |
| `sounds/music/music_level1.ogg` | Chiptune 8-bit upbeat | Level 1 / easy stages / casual gameplay |
| `sounds/music/music_level2.ogg` | Chiptune 8-bit driving | Level 2 / medium stages |
| `sounds/music/music_level3.ogg` | Chiptune 8-bit intense | Level 3 / hard stages / boss fights |
| `sounds/music/music_ending.ogg` | Chiptune triumphant | Win screen / credits / completion |

### Ambient / Electronic

| File | Style | Use for |
|------|-------|---------|
| `sounds/music/music_ambient.ogg` | Ambient electronic | Space / atmospheric / exploration games |

---

## Sound effects — `assets/sounds/sfx/`

### Core UI / Navigation

| File | Use for |
|------|---------|
| `sounds/sfx/sfx_button_click.ogg` | Any button press (Play, Retry, Buy, etc.) |
| `sounds/sfx/sfx_button_back.ogg` | Back / cancel / close buttons |
| `sounds/sfx/sfx_toggle.ogg` | Settings toggles (music on/off, sfx on/off) |
| `sounds/sfx/sfx_error.ogg` | Not enough coins / invalid action |
| `sounds/sfx/sfx_confirm.ogg` | Confirm / purchase / unlock |
| `sounds/sfx/sfx_close.ogg` | Close dialog / panel |
| `sounds/sfx/sfx_select.ogg` | Select option / highlight item |
| `sounds/sfx/sfx_toggle_alt.ogg` | Alternative toggle (different tone) |
| `sounds/sfx/sfx_switch.ogg` | Switch mode / tab |
| `sounds/sfx/sfx_tick.ogg` | Timer tick / countdown |
| `sounds/sfx/sfx_pluck.ogg` | Light tap / selection pop |
| `sounds/sfx/sfx_bong.ogg` | Notification / achievement pop |

### Gameplay

| File | Use for |
|------|---------|
| `sounds/sfx/sfx_coin.ogg` | Coin / gem / star collected |
| `sounds/sfx/sfx_jump.ogg` | Player jump / thrust / launch |
| `sounds/sfx/sfx_hit.ogg` | Player hit by obstacle / enemy |
| `sounds/sfx/sfx_game_over.ogg` | Game over event |
| `sounds/sfx/sfx_level_complete.ogg` | Level complete / high score |
| `sounds/sfx/sfx_power_up.ogg` | Power-up collected / activated |
| `sounds/sfx/sfx_powerup_alt.ogg` | Power-up variant (chiptune style) |
| `sounds/sfx/sfx_powerup_alt2.ogg` | Power-up variant 2 |

### Space / Sci-fi

| File | Use for |
|------|---------|
| `sounds/sfx/sfx_shoot.ogg` | Laser / projectile fired (classic) |
| `sounds/sfx/sfx_laser.ogg` | Laser fire variant 1 |
| `sounds/sfx/sfx_laser2.ogg` | Laser fire variant 2 |
| `sounds/sfx/sfx_zap.ogg` | Zap / electric hit |
| `sounds/sfx/sfx_phase_jump.ogg` | Phase jump / teleport |
| `sounds/sfx/sfx_phaser_up.ogg` | Phaser charge up / power increase |
| `sounds/sfx/sfx_phaser_down.ogg` | Phaser wind down / power decrease |
| `sounds/sfx/sfx_high_up.ogg` | Pitch rising (collect, level up) |
| `sounds/sfx/sfx_high_down.ogg` | Pitch falling (lose life, fail) |

---

## Loading in MainGame

```java
// Music (streaming — do NOT use Sound for music)
// Load only the tracks your game actually uses:
manager.load("sounds/music/music_menu.ogg",           Music.class);
manager.load("sounds/music/music_gameplay.ogg",       Music.class);
manager.load("sounds/music/music_gameplay_alt.ogg",   Music.class);
manager.load("sounds/music/music_gameplay_space.ogg", Music.class);
manager.load("sounds/music/music_game_over.ogg",      Music.class);
// Optional extra tracks:
// manager.load("sounds/music/music_title.ogg",      Music.class);
// manager.load("sounds/music/music_level1.ogg",     Music.class);
// manager.load("sounds/music/music_level2.ogg",     Music.class);
// manager.load("sounds/music/music_level3.ogg",     Music.class);
// manager.load("sounds/music/music_ending.ogg",     Music.class);
// manager.load("sounds/music/music_ambient.ogg",    Music.class);

// Core SFX (always load these)
manager.load("sounds/sfx/sfx_button_click.ogg", Sound.class);
manager.load("sounds/sfx/sfx_button_back.ogg",  Sound.class);
manager.load("sounds/sfx/sfx_toggle.ogg",       Sound.class);
manager.load("sounds/sfx/sfx_coin.ogg",         Sound.class);
manager.load("sounds/sfx/sfx_jump.ogg",         Sound.class);
manager.load("sounds/sfx/sfx_hit.ogg",          Sound.class);
manager.load("sounds/sfx/sfx_game_over.ogg",    Sound.class);
manager.load("sounds/sfx/sfx_level_complete.ogg", Sound.class);
manager.load("sounds/sfx/sfx_power_up.ogg",     Sound.class);
manager.load("sounds/sfx/sfx_shoot.ogg",        Sound.class);
manager.load("sounds/sfx/sfx_error.ogg",        Sound.class);
```

## Playing sounds

```java
// Background music (looping) — menu, gameplay
Music menuMusic = game.manager.get("sounds/music/music_menu.ogg", Music.class);
menuMusic.setLooping(true);
menuMusic.setVolume(0.7f);
if (game.musicEnabled) menuMusic.play();

// Game over music — plays ONCE, no loop
Music gameOverMusic = game.manager.get("sounds/music/music_game_over.ogg", Music.class);
gameOverMusic.setLooping(false);
gameOverMusic.setVolume(0.7f);
if (game.musicEnabled) gameOverMusic.play();

// Stop previous music before starting new
if (menuMusic.isPlaying()) menuMusic.stop();

// SFX
Sound clickSfx = game.manager.get("sounds/sfx/sfx_button_click.ogg", Sound.class);
if (game.sfxEnabled) clickSfx.play(1.0f);
```

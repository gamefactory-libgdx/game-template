# CLAUDE.md — Global Rules for libGDX Android Games

> **READ THIS FIRST.** Before writing any code, read `GAME_SPEC.md` and `GDD.md`
> in this project root. All rules here apply to every game. Do not deviate.

---

## 1. Tech Stack

| Layer      | Technology                        |
|------------|-----------------------------------|
| Framework  | libGDX 1.14.0                     |
| Language   | Java 17                           |
| Build      | Gradle wrapper (./gradlew only)   |
| UI         | Scene2D exclusively               |
| Assets     | AssetManager exclusively          |
| Target     | Android only                      |

---

## 2. Exact Project Structure

```
project-root/
├── CLAUDE.md
├── GAME_SPEC.md
├── GDD.md
├── assets/                          ← all game assets go here
│   ├── backgrounds/
│   └── ui/
├── core/
│   └── src/main/java/com/factory/GAME_SLUG/
│       ├── MainGame.java            ← extends Game
│       ├── Constants.java           ← ALL magic numbers
│       └── screens/
│           ├── MainMenuScreen.java
│           ├── SettingsScreen.java
│           ├── GameOverScreen.java
│           ├── LeaderboardScreen.java
│           └── ... (all other screens from GAME_SPEC)
├── android/
│   └── src/main/java/com/factory/GAME_SLUG/android/
│       └── AndroidLauncher.java     ← entry point only, do not modify
└── build.gradle
```

**IMPORTANT:** Replace `GAME_SLUG` with the actual game slug from GAME_SPEC.
Example: if slug is `brick-smash`, package is `com.factory.bricksmash`
Java files go in: `core/src/main/java/com/factory/bricksmash/`

---

## 3. Screen Requirements — MANDATORY

Every game MUST have **between 8 and 10 screens**. Hard requirement.

### Always required:
| Screen | Purpose |
|--------|---------|
| `MainMenuScreen` | Title, play button, navigation hub |
| `SettingsScreen` | Music toggle, SFX toggle |
| `GameOverScreen` | Score, personal best, retry, menu |
| `LeaderboardScreen` | Local top 10 — SharedPreferences |

### Fill remaining 4-6 from GAME_SPEC screen list.

### Screen rules:
- Every screen is a separate Java class implementing `com.badlogic.gdx.Screen`
- Constructor receives the main `Game` instance
- Navigate via `game.setScreen(new XxxScreen(game))`
- Dispose old screen after setting new one

---

## 4. Navigation Rules — MANDATORY

These are hard requirements based on real bugs found in previous games. Violating them will cause poor UX and failed QA.

### Every screen MUST have a "Main Menu" button
- No screen is a dead end
- Every screen — game, pause, game over, leaderboard, settings, level complete — must have a clearly visible button that returns to `MainMenuScreen`
- The Android system back button must also navigate to `MainMenuScreen`, never exit the app directly

```java
// CORRECT — handle back button on every screen
@Override
public boolean keyDown(int keycode) {
    if (keycode == Input.Keys.BACK) {
        game.setScreen(new MainMenuScreen(game));
        return true;
    }
    return false;
}
```

### Restart / Game Over mechanics — CRITICAL
- After game over, the game state MUST be fully reset before restarting
- Never reuse a live game state object after game over
- Always create a new `GameScreen` instance on restart — do NOT call `resume()` or `show()` on the old one
- The ball/player/projectile must start from its initial spawn position with zero velocity

```java
// CORRECT — restart creates fresh screen
retryButton.addListener(new ChangeListener() {
    public void changed(ChangeEvent event, Actor actor) {
        game.setScreen(new GameScreen(game)); // NEW instance
    }
});

// WRONG — reusing old state
retryButton.addListener(new ChangeListener() {
    public void changed(ChangeEvent event, Actor actor) {
        game.setScreen(gameScreen); // OLD instance — state is corrupted
    }
});
```

### Pause screen
- Every game screen must have a pause button visible during gameplay
- Pause must freeze all game logic (`delta` time must be ignored while paused)
- Pause screen must have: Resume, Restart, Main Menu buttons

---

## 5. Location / World Variants

If the game has multiple environments:
- Each gets its own `GameScreen` subclass
- Each has a unique background asset
- Each has at least one unique obstacle/hazard type
- Share base logic via a parent class (e.g. `BaseGameScreen extends ScreenAdapter`)

---

## 5. AndroidLauncher.java

The `AndroidLauncher.java` already exists. Update ONLY the package name and
the `AndroidApplicationConfiguration` if needed. Do not rewrite the file structure.

The applicationId in `android/build.gradle` must match the package name exactly.
Update it from `com.factory.template` to `com.factory.GAME_SLUG`.

---

## 6. Assets

```java
// CORRECT
manager.load("backgrounds/bg_desert.png", Texture.class);
manager.finishLoading();
Texture bg = manager.get("backgrounds/bg_desert.png", Texture.class);

// WRONG — never do this
Texture bg = new Texture("backgrounds/bg_desert.png");
```

- All filenames exactly as in GAME_SPEC
- If AssetManager owns it, use `manager.unload()` not `texture.dispose()`

---

## 7. Constants.java

All magic numbers go here — speeds, sizes, timings, score values,
SharedPreferences keys. Never hardcode numbers inline.

---

## 8. Build

```bash
./gradlew android:assembleDebug
```

- Zero errors, zero warnings
- Gradle wrapper only — never system gradle
- Fix ALL errors before finishing

---

## 9. Data Persistence

```java
Preferences prefs = Gdx.app.getPreferences("GamePrefs");
prefs.putInteger("highScore", score);
prefs.flush();
```

Keys defined in `Constants.java`. Save: high scores, unlocks, settings.

---

## 10. Reference Games

If reference games are provided in the prompt, they are located at:
```
/home/kaliuzhnyi/asocity/reference/<game-folder>/
```

Study them BEFORE writing any code:
- Read their `core/src/` Java files to understand class structure
- Look at how screens are organized and how navigation is implemented
- Look at how `AssetManager` is loaded and disposed
- Look at how input is handled per screen
- Match their code quality, architecture, and scope complexity

If no reference games are provided — build a clean, minimal implementation matching the archetype described in the GDD.

---

## 11. Completion Checklist

- [ ] `./gradlew android:assembleDebug` → BUILD SUCCESSFUL
- [ ] Exactly 8-10 screens implemented
- [ ] All screens in GAME_SPEC implemented
- [ ] All screens navigate correctly per GAME_SPEC screen flow
- [ ] Every screen has a visible "Main Menu" button
- [ ] Android back button navigates to MainMenuScreen on every screen
- [ ] Game Over screen creates a NEW GameScreen instance on retry
- [ ] Pause screen implemented with Resume / Restart / Main Menu
- [ ] Game state fully resets on restart — no stale velocity or position
- [ ] All assets in GAME_SPEC exist in `assets/`
- [ ] All assets loaded via AssetManager — zero `new Texture()` calls
- [ ] `Constants.java` exists with all magic numbers
- [ ] SharedPreferences saves/loads scores and settings
- [ ] No `System.out.println` — use `Gdx.app.log`
- [ ] `dispose()` on every screen
- [ ] Package name updated from `com.factory.template` to `com.factory.GAME_SLUG`
- [ ] `applicationId` in `android/build.gradle` updated to match package name

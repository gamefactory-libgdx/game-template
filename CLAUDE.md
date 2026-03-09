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
│   ├── ui/
│   └── sprites/                     ← Kenney CC0 sprites (pre-copied by pipeline)
│       └── SPRITES.md               ← manifest — READ THIS before using any sprite
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

## 4. Physics Constants — MANDATORY

### Hold-to-fly / jetpack mechanic
If the game uses a "hold screen to fly/rise" mechanic (jetpack, flappy, balloon, rocket):
- `JETPACK_THRUST` (or equivalent) **MUST be at least 2× |GRAVITY|**
- With additive physics both forces apply every frame: net = THRUST + GRAVITY
- If THRUST < |GRAVITY| the player can never rise — game is unplayable

```java
// CORRECT — thrust overcomes gravity, net force is upward when holding
public static final float GRAVITY        = -900f;
public static final float JETPACK_THRUST = 1800f; // 2× gravity → net +900 upward
public static final float MAX_FALL_SPEED = -700f;

// WRONG — thrust less than gravity, player always falls even when holding
public static final float GRAVITY        = -900f;
public static final float JETPACK_THRUST =  600f; // net -300 → never rises
```

### Touch input in GameScreen with a HUD Stage
When a HUD `Stage` is in the `InputMultiplexer`, always poll input via `Gdx.input.isTouched()`
for game physics — never rely on Stage touch events for continuous hold detection.

```java
// CORRECT — polling works regardless of whether Stage consumed the event
boolean thrusting = Gdx.input.isTouched();

// WRONG — Stage touchDown fires once, not continuously while held
```

---

## 5. Sprites — MANDATORY

### How sprites work
The pipeline automatically copies a curated Kenney CC0 sprite library into
`assets/sprites/` before Claude runs. A manifest `assets/sprites/SPRITES.md`
is also written there, listing every available file.

**Before writing ANY game object code:**
1. Read `assets/sprites/SPRITES.md` — it is always present
2. Use the sprites listed there for player, enemies, obstacles, collectibles
3. Never use plain colored rectangles for game objects when sprites are available

### Categories available
| Category | Sprites | Use for |
|----------|---------|---------|
| `runner/` | player walk/jump/hurt, enemies (slime, fly, bee, saw), obstacles (spike, rock, lava), ground tiles | Side-scrollers, runners, jetpack, flappy-style |
| `space/`  | player ships, enemy ships (4 types), lasers, asteroids | Space shooters, asteroid dodge |
| `racing/` | car_player (yellow), car_red, car_blue, car_green, car_black | Top-down car games |
| `puzzle/` | balls (blue/yellow/grey), paddles, back tiles | Brick breaker, ball games |
| `generic/`| coin_gold, coin_silver, star, gem (4 colors) | Collectibles in any game |

### Loading sprites with AssetManager

```java
// In your loading screen or MainGame:
manager.load("sprites/player_idle.png", Texture.class);
manager.load("sprites/player_walk1.png", Texture.class);
manager.load("sprites/enemy_slime.png", Texture.class);
manager.load("sprites/coin_gold.png", Texture.class);
// ... load all sprites you'll use

// In your screen (after finishLoading):
Texture playerTex = manager.get("sprites/player_idle.png", Texture.class);
```

### Animation example (walk cycle)

```java
// Declare
private Animation<TextureRegion> walkAnim;
private float stateTime = 0f;

// In constructor (after assets loaded):
TextureRegion[] frames = {
    new TextureRegion(manager.get("sprites/player_walk1.png", Texture.class)),
    new TextureRegion(manager.get("sprites/player_walk2.png", Texture.class))
};
walkAnim = new Animation<>(0.15f, frames);

// In render():
stateTime += delta;
TextureRegion frame = walkAnim.getKeyFrame(stateTime, true);
batch.draw(frame, x, y, width, height);
```

### Important rules
- Load sprites via `AssetManager` — zero `new Texture("sprites/...")` calls
- Scale sprites to world units (not pixel size): typical player is 64×64 world units
- If a sprite category doesn't match the game theme, use `generic/` collectibles
  and draw obstacles/terrain using `ShapeRenderer` as fallback

---

## 6. Navigation Rules — MANDATORY

These are hard requirements based on real bugs. Violating them causes broken UX.

### Every screen MUST have a "Main Menu" button
- No screen is a dead end
- Every screen — game, pause, game over, leaderboard, settings, level complete — must have a clearly visible button that returns to `MainMenuScreen`
- The Android system back button must also navigate to `MainMenuScreen`, never exit the app

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
- After game over, game state MUST be fully reset before restarting
- Always create a NEW `GameScreen` instance on retry — never reuse the old one
- Ball/player/projectile must start from initial spawn position with zero velocity

```java
// CORRECT — restart creates fresh screen
retryButton.addListener(new ChangeListener() {
    public void changed(ChangeEvent event, Actor actor) {
        game.setScreen(new GameScreen(game)); // NEW instance every time
    }
});

// WRONG — reusing old state causes ball stuck, input broken
retryButton.addListener(new ChangeListener() {
    public void changed(ChangeEvent event, Actor actor) {
        game.setScreen(gameScreen); // OLD instance — never do this
    }
});
```

### Pause screen — MANDATORY
- Every game screen must have a visible pause button during gameplay
- Pause must freeze all game logic (delta time ignored while paused)
- Pause screen must have: Resume, Restart, Main Menu buttons

---

## 7. Viewport — MANDATORY — No Black Bars

Every screen MUST use `StretchViewport` to fill the entire screen with no black bars.

```java
// In every screen — declare fields
private OrthographicCamera camera;
private Viewport viewport;
private Stage stage;

// In constructor
camera = new OrthographicCamera();
viewport = new StretchViewport(Constants.WORLD_WIDTH, Constants.WORLD_HEIGHT, camera);
stage = new Stage(viewport, game.batch);
Gdx.input.setInputProcessor(stage);

// resize() — MANDATORY in every screen
@Override
public void resize(int width, int height) {
    viewport.update(width, height, true);
}
```

Define in `Constants.java`:
```java
public static final float WORLD_WIDTH  = 480f;
public static final float WORLD_HEIGHT = 854f;  // 16:9 portrait
```

All game coordinates use world units (480x854), never raw pixel values.

---

## 8. Location / World Variants

If the game has multiple environments:
- Each gets its own `GameScreen` subclass
- Each has a unique background asset
- Each has at least one unique obstacle/hazard type
- Share base logic via a parent class (e.g. `BaseGameScreen extends ScreenAdapter`)

---

## 9. AndroidLauncher.java

Create `android/src/main/java/com/factory/GAME_SLUG/android/AndroidLauncher.java`
(replace `GAME_SLUG` with the actual slug, dots removed — same as the package name).

```java
package com.factory.GAME_SLUG.android;

import android.os.Bundle;
import com.badlogic.gdx.backends.android.AndroidApplication;
import com.badlogic.gdx.backends.android.AndroidApplicationConfiguration;
import com.factory.GAME_SLUG.MainGame;

public class AndroidLauncher extends AndroidApplication {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        AndroidApplicationConfiguration configuration = new AndroidApplicationConfiguration();
        configuration.useImmersiveMode = true;
        initialize(new MainGame(), configuration);
    }
}
```

The applicationId in `android/build.gradle` must match the package name exactly.
Update it from `com.factory.template` to `com.factory.GAME_SLUG`.

---

## 10. Assets

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

## 11. Constants.java

All magic numbers go here — speeds, sizes, timings, score values,
SharedPreferences keys, world dimensions. Never hardcode numbers inline.

```java
public class Constants {
    public static final float WORLD_WIDTH  = 480f;
    public static final float WORLD_HEIGHT = 854f;
    // ... all other constants
}
```

---

## 12. Build

```bash
./gradlew android:assembleDebug
```

- Zero errors, zero warnings
- Gradle wrapper only — never system gradle
- Fix ALL errors before finishing

---

## 13. Data Persistence

```java
Preferences prefs = Gdx.app.getPreferences("GamePrefs");
prefs.putInteger("highScore", score);
prefs.flush();
```

Keys defined in `Constants.java`. Save: high scores, unlocks, settings.

---

## 14. Reference Games

If reference games are provided in the prompt, they are at:
```
/home/kaliuzhnyi/asocity/reference/<game-folder>/
```

Study them BEFORE writing any code:
- Read their `core/src/` Java files to understand class structure
- Look at how screens are organized and navigation implemented
- Look at how `AssetManager` is loaded and disposed
- Look at how input is handled per screen
- Match their code quality, architecture, and scope

If no reference games provided — build clean, minimal implementation matching the GDD archetype.

---

## 15. Completion Checklist

- [ ] `./gradlew android:assembleDebug` → BUILD SUCCESSFUL
- [ ] Exactly 8-10 screens implemented
- [ ] All screens in GAME_SPEC implemented
- [ ] All screens navigate correctly per GAME_SPEC screen flow
- [ ] Every screen has a visible "Main Menu" button
- [ ] Android back button navigates to MainMenuScreen on every screen — never exits app
- [ ] Game Over retry creates a NEW GameScreen instance — never reuses old state
- [ ] Ball/player starts from spawn position with zero velocity on restart
- [ ] Pause screen implemented with Resume / Restart / Main Menu
- [ ] Every screen uses StretchViewport — no black bars
- [ ] All assets in GAME_SPEC exist in `assets/`
- [ ] All assets loaded via AssetManager — zero `new Texture()` calls
- [ ] `Constants.java` has WORLD_WIDTH, WORLD_HEIGHT and all magic numbers
- [ ] SharedPreferences saves/loads scores and settings
- [ ] No `System.out.println` — use `Gdx.app.log`
- [ ] `dispose()` on every screen
- [ ] Package name updated from `com.factory.template` to `com.factory.GAME_SLUG`
- [ ] `applicationId` in `android/build.gradle` updated to match package name
- [ ] android/res/values/strings.xml app_name updated to game title
- [ ] `assets/sprites/SPRITES.md` was read before writing game object code
- [ ] Player, enemies, collectibles use sprites from `assets/sprites/` — no plain rectangles

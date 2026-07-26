# Matcha

A compact, native macOS companion for Minecraft with independent controls for **Fullbright**, **Player Glow**, **Aim Assist**, and **Auto Shoot**.

Matcha is designed to feel at home on macOS: it stays available across Spaces and over fullscreen games, remembers your settings, and connects to Minecraft through a small local Fabric bridge. It does not contact the internet or generate macOS mouse and keyboard input.

> [!IMPORTANT]
> Player visibility, aim assistance, and automated attacks may be prohibited by a server's rules. Use these features only in single-player worlds or where the server explicitly permits them.

## Features

### Visuals

- **Fullbright** adjusts the selected display's gamma and restores the original appearance immediately when switched off.
- **Player Glow** outlines every other player within 128 blocks, including wall-hidden players. Your own player is excluded.
- Fullbright and Player Glow operate independently.
- The app automatically prefers the display containing a visible Minecraft, Java, or Roblox window.

### Aim Assist

Choose from grouped targets—players, passive mobs, hostile mobs, or all living entities—or select one of 51 individual mob types.

- **Normal** smoothly guides the camera toward the selected Head, Torso, or Random aim point.
- **Random** chooses a new inset point within the target every 400 milliseconds, keeping it stable between changes to avoid frame-by-frame jitter.
- **Edge** leaves the camera completely free while the crosshair remains inside the entity's projected bounds, then applies only the smallest correction needed at the boundary.
- **Speed** controls smoothing and the maximum correction per rendered frame.
- **Crosshair range** controls the activation cone from 5° to 180°.
- **Minimum reaction** and **Maximum reaction** set a 0–3 second range. Each newly acquired target receives a random reaction time inside that range; matching values produce a fixed reaction time.

Aim Assist pauses while a Minecraft menu is open. It also disables automatically if the app's local heartbeat disappears for 750 milliseconds.

### Auto Shoot

Auto Shoot has its own target picker and can trigger on the **Head**, **Torso**, or **Every Part** of a matching entity. Separate **Minimum delay** and **Maximum delay** sliders range from 0.25 to 3 seconds. Each first or repeat shot receives a fresh random delay inside that range; setting both sliders to the same value produces a fixed delay.

It attacks only when all of the following are true:

- Minecraft's real crosshair hit falls inside the selected region.
- The entity matches the selected target.
- The target is within normal vanilla reach.
- The standard attack cooldown is at least 95% ready.
- The randomly selected delay between the configured minimum and maximum has elapsed.

Attacks run at the end of Minecraft's client tick, after the normal movement and camera update. The feature does not extend reach or create system-level input events.

## Requirements

- Apple Silicon Mac
- macOS 14 or later
- Minecraft 1.21.1 with Fabric
- The included `matcha-player-bridge-1.0.0.jar`

The supplied bridge is specific to Minecraft 1.21.1 and Fabric. Other Minecraft versions and mod loaders require a compatible bridge build.

## Install and run

1. Add `matcha-player-bridge-1.0.0.jar` to the `mods` folder of a compatible Fabric modpack. It is already installed in **FullBright Test**.
2. Start the modpack.
3. Unzip `Fullbright Controller.zip`.
4. Open **Fullbright Controller.app**.
5. Enter a world and choose a feature from the **Visuals**, **Aim Assist**, or **Auto Shoot** tab.

If macOS blocks the app because it is from an unidentified developer, Control-click **Fullbright Controller.app**, choose **Open**, and confirm.

The global Fullbright shortcut is **Control–Option–F**.

## How it works

The Fabric bridge sends game-state updates to the controller over UDP on `127.0.0.1:40117`. Communication stays on your Mac; the bridge does not contact the internet.

Each update includes the Minecraft Java process ID so the app can identify the correct game window even when other launchers, servers, or Java applications are open. Player rectangles are projected from Minecraft's live view and projection matrices at up to 120 Hz, keeping outlines aligned through camera movement, sprinting, view bobbing, and FOV changes.

The controller uses a single click-through, GPU-composited window for player outlines. A capture of only the Minecraft window should not include the overlay, while a recording of the entire display will.

If Minecraft is minimized or closed—or if player data stops arriving for 750 milliseconds—the overlay hides automatically.

## Privacy and system access

- No internet connection is used.
- No gameplay data leaves your Mac.
- No mouse or keyboard events are generated at the macOS level.
- The global shortcut receives only its registered key combination; it does not monitor typing.
- Screen Recording and Accessibility permissions are not required.
- Disabling Fullbright immediately restores the display's original transfer settings.

Fullbright changes the entire display containing the likely game window. If the window cannot be identified, the main display is used. HDR and some externally managed displays may reject gamma changes; in that case, the app reports the problem and leaves Fullbright off.

## Project structure

- `App` — lifecycle and central state coordination
- `UI` — SwiftUI controller, onboarding, menu bar interface, and native visual-effect surfaces
- `WindowManagement` — game-window discovery, exact Java-process matching, bridge updates, and the player-outline window
- `Rendering` — display-gamma control and rendering support
- `Hotkey` — the registered Carbon global shortcut
- `Preferences` — versioned local settings stored with `UserDefaults`
- `MatchaSheepBridge` — the client-only Fabric 1.21.1 bridge
- `FullbrightControllerTests` — preference, targeting, tracking, message, and rendering tests

The bridge directory retains its original `MatchaSheepBridge` name so existing installations can upgrade cleanly.

## Build from source

Building requires Xcode 16 or later. The checked-in project is also compatible with Xcode 26.

1. Open `FullbrightController.xcodeproj` in Xcode.
2. Select the **FullbrightController** scheme and **My Mac** destination.
3. In **Signing & Capabilities**, select your Apple Developer team and replace the example bundle identifier if needed.
4. Press **Run**.

For an unsigned local build:

```sh
xcodebuild -project FullbrightController.xcodeproj \
  -scheme FullbrightController \
  -destination 'platform=macOS,arch=arm64' \
  -derivedDataPath work/DerivedData \
  CODE_SIGNING_ALLOWED=NO build
```

To run the tests:

```sh
xcodebuild -project FullbrightController.xcodeproj \
  -scheme FullbrightController \
  -destination 'platform=macOS,arch=arm64' \
  -derivedDataPath work/DerivedData \
  CODE_SIGNING_ALLOWED=NO test
```

If XcodeGen is installed, regenerate the Xcode project with:

```sh
xcodegen generate
```

## Sign and distribute

For direct distribution outside the Mac App Store:

1. Set a unique bundle identifier and select an Apple Developer team under **Signing & Capabilities**.
2. In Xcode, choose **Product → Archive**.
3. In Organizer, choose **Distribute App**.
4. Select **Developer ID** and submit the app for notarization when prompted.
5. Export the notarized app.

Xcode's Organizer handles signing, notarization, and ticket stapling and is the simplest distribution path. A local-only build can instead be exported with **Copy App**.

## Troubleshooting

### The app cannot connect to Minecraft

Confirm that the bridge JAR is in the active Fabric modpack's `mods` folder and that the modpack is running Minecraft 1.21.1.

### Player Glow disappears

The overlay hides when Minecraft is minimized, closed, or has not sent player data for 750 milliseconds. Return to an active world and confirm that the bridge is loaded.

### Fullbright will not turn on

HDR or display-management software may prevent gamma changes. Try disabling HDR or testing on a different display.

### Aim Assist or Auto Shoot stops unexpectedly

Both features disable when the controller loses its local connection to the bridge. Check that the app is open and Minecraft is still running.

# Matcha

Matcha is a native macOS companion for Minecraft with a compact controller, full-screen support, and a local Fabric bridge. Its features are organized into four tabs: **Visuals**, **Aim Assist**, **Auto Shoot**, and **Movement**.

> Matcha **ONLY** supports fabric 1.21.1 MacOS

## Features

- **Fullbright** brightens dark areas and restores the original display appearance immediately when switched off.
- **Player Glow** outlines up to 64 other players within 128 blocks, including players behind walls. Your own player is excluded.
- **Aim Assist** provides smooth camera guidance with target categories, Normal and Edge modes, configurable aim points, speed, crosshair range, and randomized reaction times.
- **Auto Shoot** attacks only when the crosshair is over a matching target, vanilla reach and cooldown checks pass, and the selected randomized delay has elapsed.
- **No-Slow** preserves normal local movement while using items or moving through cobwebs.
- **Full-screen support** keeps Matcha’s click-through overlays above Minecraft in a native macOS full-screen Space.

All features work independently. The Fabric-powered features automatically switch off inside the bridge if the Matcha app disconnects.

## Requirements

- Apple Silicon Mac
- macOS 14 or newer
- Minecraft Java Edition 1.21.1
- Fabric Loader and Fabric API
- Java 21 for Minecraft

The included bridge is specifically built for **Minecraft 1.21.1 with Fabric**. Other Minecraft versions and mod loaders require a matching bridge build.

## Installation

### 1. Install the Fabric bridge

1. Locate your Minecraft instance’s `mods` folder.
2. Copy `matcha-player-bridge-1.0.0.jar` into that folder.
3. Confirm the instance also contains Fabric API.
4. Start Minecraft using the Fabric 1.21.1 profile.

If you are using the provided **FullBright Test** instance, the bridge may already be installed. Replace the existing JAR when updating Matcha.

### 2. Install the macOS app

1. Download and unzip `Fullbright Controller.zip`.
2. Move **Fullbright Controller.app** to your Applications folder if desired.
3. Double-click the app to launch it.

## If macOS says the developer cannot be verified

Matcha’s downloadable build is locally signed rather than notarized with an Apple Developer ID, so macOS may display an **unidentified developer**, **cannot verify the developer**, or **Apple could not verify this app is free of malware** warning.

Try opening it normally once, then:

1. Open **System Settings** from the Apple menu.
2. Select **Privacy & Security** in the sidebar.
3. Scroll down to the **Security** section.
4. Find the message stating that **Fullbright Controller** was blocked.
5. Click **Open Anyway**.
6. Authenticate with Touch ID or your Mac password if prompted.
7. Click **Open** in the final confirmation dialog.

macOS remembers this choice, so you should not need to repeat these steps for that copy of the app. If **Open Anyway** is missing, try launching the app again and return to **Privacy & Security**. You can also Control-click the app in Finder, choose **Open**, and confirm **Open** when that option is available.

Do not disable Gatekeeper globally.

## Getting started

1. Start Minecraft and enter a world.
2. Open Matcha.
3. Choose a tab and enable the feature you want.
4. Use **Control–Option–F** to toggle Fullbright from anywhere.

Matcha uses the Java process ID reported by the bridge to attach to the correct Minecraft window, even when launchers or other Java applications are running. The controller and overlays remain available in Minecraft’s normal macOS full-screen mode.

## Feature details

### Fullbright and Player Glow

Fullbright applies a display gamma adjustment to the screen containing the detected game window. Turning it off immediately restores the original display settings. Player Glow uses one click-through, GPU-composited overlay and does not take mouse or keyboard focus.

### Aim Assist

Aim Assist supports Players, passive mobs, hostile mobs, all living entities, and 51 individual targets.

- **Normal** guides the camera toward the selected Head, Torso, or Random aim point.
- **Random** chooses a stable inset point across the target every 400 milliseconds.
- **Edge** leaves the camera free while the crosshair remains inside the entity’s projected bounds, then applies the smallest correction needed at the edge.
- **Minimum reaction** and **Maximum reaction** define a randomized 0–3 second reaction-time range. Set both to the same value for a fixed reaction time.

Aim Assist pauses while a Minecraft menu is open and does not generate macOS mouse or keyboard events.

### Auto Shoot

Auto Shoot includes its own target selector, Head/Torso/Every Part trigger regions, and separate minimum and maximum delays from 0.25 to 3 seconds. Every shot receives a new random delay within that range; matching values create a fixed delay.

An attack occurs only when:

- Minecraft’s actual crosshair hit matches the chosen target and region.
- The target is within normal vanilla reach.
- The standard attack cooldown is at least 95% ready.
- The selected delay has elapsed.

Attacks run during Minecraft’s normal key-input phase to preserve ordinary interaction packet ordering.

### No-Slow

No-Slow preserves normal local walking input while eating, drinking, drawing a bow, blocking, using other items, or moving through cobwebs. Cobweb handling applies only to the local player; berry bushes, powder snow, and other block effects remain unchanged.

The server remains authoritative and may correct or reject modified movement.

## Privacy and networking

- Communication stays on `127.0.0.1` using UDP port `40117`.
- The bridge does not contact the internet.
- Matcha does not inject macOS mouse or keyboard input into Minecraft.
- The global shortcut listens only for its registered key combination.
- Feature settings are stored locally in macOS `UserDefaults`.
- If Minecraft closes, is minimized, or stops sending bridge data, Matcha hides its player overlay automatically.

## Known limitations

- Some servers may reject or correct Aim Assist, Auto Shoot, or No-Slow behavior according to their own rules.
- HDR and some externally managed displays may reject Fullbright’s gamma adjustment.
- Display recordings can include Matcha’s overlays; a recording limited to the Minecraft window generally will not.
- The supplied app is locally signed but not Apple-notarized, which causes the first-launch security warning described above.

## Project structure

- `FullbrightController` — native Swift and SwiftUI macOS app.
- `MatchaSheepBridge` — client-only Fabric 1.21.1 bridge.
- `FullbrightControllerTests` — macOS unit tests.
- `project.yml` — XcodeGen project definition.

## Build from source

Open `FullbrightController.xcodeproj` in Xcode, select the **FullbrightController** scheme and **My Mac**, choose your signing team, and press **Run**.

Build the Fabric bridge from `MatchaSheepBridge` with Java 21:

```sh
./gradlew build
```

The finished bridge JAR is written to `MatchaSheepBridge/build/libs`.


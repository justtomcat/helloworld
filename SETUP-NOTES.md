# HelloApp — environment setup notes

Scaffolded with React Native **0.87.1** (`@react-native-community/cli` 20.2.0) on
2026-09-02. The JavaScript layer is verified and working. **The Android build is not
usable yet** — this file lists what is missing and how to fix it.

## What was verified

| Check | Result |
|---|---|
| `npm test` (Jest renders `App.tsx`) | PASS — 1/1 |
| `npx tsc --noEmit` | PASS — no type errors |
| `npm start` (Metro bundler) | PASS — "Dev server ready" on port 8081 |
| Android build / run on device | **NOT RUN** — see gaps below |
| iOS build | **NOT POSSIBLE** — requires macOS + Xcode |

## What this project's native build needs

Read out of the generated `android/` folder, not guessed:

- Gradle **9.4.1** (`android/gradle/wrapper/gradle-wrapper.properties`)
- Android Gradle Plugin **9.2.1** (`node_modules/@react-native/gradle-plugin/gradle/libs.versions.toml`)
- Kotlin 2.2.0
- `compileSdkVersion` **37**, `targetSdkVersion` 36, `minSdkVersion` 24
- `buildToolsVersion` **37.0.0**
- NDK **27.1.12297006**

These are recent versions — an older Android Studio / SDK install will not have SDK 37.

## Gap 1 — JDK

`JAVA_HOME` is currently `C:\Program Files\Java\jdk-11`. **JDK 11 is too old** and the
Gradle build will fail on it.

Installed on this machine: JDK 11, 23, 25, plus a JRE 8. There is no JDK 17 or 21.

Recommended fix — install Temurin **17** (the version React Native documents) or **21**:

```powershell
winget install EclipseAdoptium.Temurin.17.JDK
```

Then point `JAVA_HOME` at it and put it ahead of the others on `PATH`:

```powershell
[Environment]::SetEnvironmentVariable('JAVA_HOME','C:\Program Files\Eclipse Adoptium\jdk-17...','User')
```

Note: Gradle 9.4.1 does support newer JDKs, so the already-installed JDK 23 or 25 may work.
That was not tested here — JDK 17 or 21 is the low-risk choice.

## Gap 2 — Android SDK is not installed

`ANDROID_HOME` is set to `C:\Users\care\AppData\Local\Android\Sdk`, but **that directory
does not exist**. `C:\Program Files\Android\Android Studio` is also empty — Android Studio
appears to have been uninstalled, leaving only stale config under
`%LOCALAPPDATA%\Google\AndroidStudio2025.2.3`.

Fix — install Android Studio:

```powershell
winget install Google.AndroidStudio
```

Then in Android Studio's SDK Manager install:

- SDK Platform **37**
- Build-Tools **37.0.0**
- Platform-Tools
- NDK **27.1.12297006**
- Android Emulator

## Gap 3 — ANDROID_HOME must be repointed

After the SDK is installed, set `ANDROID_HOME` to the real path (Android Studio reports it
in SDK Manager) and add `platform-tools` to `PATH` so `adb` resolves.

Verify with:

```bash
adb version
```

## Gap 4 — no emulator defined

`~/.android/avd` is empty. Either create an AVD in Android Studio's Device Manager, or
connect a physical Android device with USB debugging enabled and confirm it appears in
`adb devices`.

## Gap 5 — iOS

Not buildable on Windows. The `ios/` folder and `Gemfile` were generated but are inert here;
they need macOS with Xcode and CocoaPods.

## Running once the above is done

```bash
npm start            # terminal 1 — Metro bundler
npm run android      # terminal 2 — build + install on emulator/device
```

## Notes on this machine's tooling

- npm is **6.14.18**, unusually old for Node 22 (which normally ships npm 10). It installed
  this project fine. If dependency installs start misbehaving later, upgrading npm is the
  first thing to try.
- The RN CLI **rejects any project name containing "HelloWorld"** — it is the CLI's internal
  placeholder. That is why this project is named `HelloApp`.

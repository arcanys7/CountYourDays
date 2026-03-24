# CountYourDays — Android App

A Kotlin/MVVM Android app that automatically sets your phone wallpaper to the
number of days remaining until your exam — with a precise red-intensity color
system, dark/light UI mode, and rich background customisation.

---

## Project Structure

```
CountYourDays/
├── app/
│   ├── build.gradle
│   └── src/main/
│       ├── AndroidManifest.xml
│       └── java/com/countyourdays/
│           ├── ui/
│           │   └── MainActivity.kt          ← Full UI, theme switching, pickers
│           ├── viewmodel/
│           │   └── MainViewModel.kt         ← MVVM state & coroutines
│           ├── worker/
│           │   ├── WallpaperUpdateWorker.kt ← WorkManager daily job
│           │   └── BootReceiver.kt          ← Reschedule after reboot
│           └── utils/
│               ├── CountdownManager.kt      ← Color scale + countdown logic
│               ├── WallpaperGenerator.kt    ← Bitmap engine
│               ├── WallpaperApplier.kt      ← WallpaperManager wrapper
│               └── PreferencesManager.kt    ← SharedPreferences
```

---

## Color Scale (10 days → 0 days)

| Days | Shade Name       | Hex       |
|------|-----------------|-----------|
| >10  | Neutral White   | `#FFFFFF` |
| 10   | Misty Rose      | `#FFE4E1` |
| 9    | Light Salmon Red| `#FFA07A` |
| 8    | Coral Red       | `#FF6F61` |
| 7    | Soft Red        | `#FF4D4D` |
| 6    | True Red        | `#FF0000` |
| 5    | Crimson         | `#DC143C` |
| 4    | Firebrick       | `#B22222` |
| 3    | Dark Red        | `#8B0000` |
| 2    | Blood Red       | `#660000` |
| 1    | Deep Blood Red  | `#3B0000` |
| 0    | Abyss Blood Red | `#1A0000` |

Color only starts changing at 10 days remaining. Above 10 days → neutral white.
Days 0–2 receive a red glow effect (BlurMaskFilter) for visibility on dark backgrounds.

---

## Features

### Dark / Light Mode
Toggle in the top-right of the UI. Both modes have high-contrast text and
clearly visible Dark/Light buttons at all times.

### Background Options
Three curated categories of wallpaper backgrounds:
- **Dark & Moody** — pure blacks and near-blacks
- **Colorful Contrast** — Deep Indigo, Midnight Navy, Forest Black, Dark Amber,
  Deep Teal, Dark Purple, Deep Blue, Dark Olive, Dark Ember, Dark Cyan
  (all chosen to make the red scale pop)
- **Rich Tones** — Violet Night, Ocean Depth, Ember Glow, Deep Forest,
  Dark Magenta, Cosmic, Moss Night, Dark Gold, Abyss Teal
- **Upload from device** — any photo from your gallery
- **Hex input** — any custom `#RRGGBB` color

### Daily Auto-Update
WorkManager PeriodicWorkRequest fires once per day. BootReceiver reschedules
after device restart. Runs only when auto-update is enabled.

### Font Scaling
Number font size is computed dynamically to fill 78% of screen width on any device:
```kotlin
val scale = (screenWidth * 0.78f) / measuredTextWidth
val fontSize = (1000f * scale).coerceIn(120f, screenHeight * 0.72f)
```

---

## How to Build

1. Open in **Android Studio Giraffe** (or newer)
2. Sync Gradle
3. Run on a device or emulator — **minSdk 26**

### Permissions
- `SET_WALLPAPER`
- `READ_MEDIA_IMAGES` (API 33+) / `READ_EXTERNAL_STORAGE` (API ≤32)
- `RECEIVE_BOOT_COMPLETED`

---
## Current Limitation
- **Lock screen wallpaper is not working properly yet**

## Architecture

```
MainActivity  ──observes LiveData──▶  MainViewModel
                                           │
                              ┌────────────┼────────────────┐
                              ▼            ▼                 ▼
                    CountdownManager  WallpaperGenerator  PreferencesManager
                    (pure logic)      (Bitmap drawing)    (SharedPreferences)
                                           │
                                           ▼
                                    WallpaperApplier
                                    (WallpaperManager)

WorkManager (background)
    WallpaperUpdateWorker
        └── WallpaperGenerator + WallpaperApplier + PreferencesManager
```

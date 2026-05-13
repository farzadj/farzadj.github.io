[FlashDeutsch website](https://farzadj.github.io/FlashDeutsch/)

# FlashDeutsch

FlashDeutsch is a native Android flashcard app for learners of German with Persian, English, Spanish, Russian, and Arabic learning-language modes. It is fully offline after install and uses the bundled study PDFs in [`app/src/main/assets/pdfs`](app/src/main/assets/pdfs) as its source material.

## What It Builds

- Native Android app
- Kotlin + Jetpack Compose + Material 3
- Navigation Compose
- MVVM with screen-level `ViewModel`s
- Room for offline persistence
- DataStore for settings
- Offline deck import from generated JSON assets

## Source PDFs

The repository currently imports these PDFs:

- `B1_perplexity.pdf`
- `B1_plus_perplexity.pdf`
- `B1_plus2_perplexity.pdf`
- `B1_plus3_perplexity.pdf`
- `B1_plus_voc_perplexity.pdf`
- `B1_plus_voc_200_perplexity.pdf`
- `B1_plus_voc2_200_perplexity.pdf`
- `B1_plus_voc3_200_perplexity.pdf`

The generated import summary currently contains:

- 116 decks
- 4259 cards
- 80 import and translation warnings

## Parsing Strategy

The app does **not** parse PDFs on-device. Instead, it uses a build-time normalization pipeline:

1. PDFs are stored in `app/src/main/assets/pdfs/`.
2. [`tools/generate_decks.py`](tools/generate_decks.py) reads the PDFs with `pypdf`.
3. The script detects sentence vs vocabulary decks from the observed line format.
4. It preserves deck names, topics, and source filenames.
5. It normalizes Persian text into logical RTL order.
6. It generates offline learning-language support text at build time:
   - German is the primary source for English, Spanish, Russian, and Arabic
   - Persian can be used as a comparison/fallback signal where helpful
   - translations are cached in [`tools/translation_cache.json`](tools/translation_cache.json) to avoid repeating long network runs
   - [`tools/fill_learning_language_assets.py`](tools/fill_learning_language_assets.py) can fill an added learning-language layer without reparsing every PDF
7. It writes:
   - [`app/src/main/assets/generated/decks.json`](app/src/main/assets/generated/decks.json)
   - [`app/src/main/assets/generated/import_summary.json`](app/src/main/assets/generated/import_summary.json)
8. The Android app imports the generated JSON into Room on first launch or when re-import is triggered from Settings.

This is more reliable than on-device PDF parsing because the PDF text structure is consistent but not clean enough to trust runtime extraction across devices.

## App Features

- Workflow-first browsing by library, type, category, and generated study set
- Flashcards with flip, next, previous, shuffle, hard, and favorite
- Resume last studied position per deck and mode
- Search across German, Persian, English, Spanish, Russian, Arabic, categories, and study metadata
- Saved favorite and hard-card decks
- Persian/Arabic RTL rendering and German LTR rendering
- German device TTS with visible-side autoplay/read-mode support
- Theme selection: system, light, dark
- First-run selection for app language and learning language
- Font scale, learning-language-first vocabulary side order, shuffle default, and haptics settings
- Reset progress and re-import actions

## Architecture

### Packages

- `ui`
  - screens, navigation, reusable Compose components, theme
- `domain`
  - app models, quiz logic, progress calculations
- `data`
  - repository orchestration and domain mapping
- `parser/importer`
  - generated asset models and asset importer
- `database`
  - Room entities, DAOs, database
- `settings`
  - DataStore-backed settings repository

### Data Flow

1. `SplashViewModel` calls repository initialization.
2. The repository checks the generated asset version from DataStore.
3. If needed, JSON assets are imported into Room.
4. Screen `ViewModel`s observe immutable UI state from repository `Flow`s.
5. User actions flow back through the `ViewModel` to repository mutations.
6. Settings drive both app chrome language and the learning-language side used in flashcards, quiz, typing, search, and TTS fallback logic.

## Project Layout

```text
app/
  src/main/
    assets/
      generated/
      pdfs/
    java/com/flashdeutsch/
      data/
      database/
      domain/
      parser/importer/
      settings/
      ui/
tools/
  generate_decks.py
  tests/
```

## How To Add a New PDF Deck

1. Copy the PDF into `app/src/main/assets/pdfs/`.
2. Add filename metadata in [`tools/generate_decks.py`](tools/generate_decks.py):
   - display name
   - group name
   - sort order
3. Run:

```powershell
python tools\generate_decks.py --version 2
```

This step may take a while the first time because learning-language support text is generated and cached.

4. Rebuild the app:

```powershell
$env:JAVA_HOME='C:\Program Files\Android\Android Studio\jbr'
$env:GRADLE_USER_HOME=(Resolve-Path '.gradle-home').Path
$env:ANDROID_USER_HOME=(Resolve-Path '.android-home').Path
.\gradlew.bat assembleDebug --no-daemon
```

5. If you changed parsing rules, run the parser tests:

```powershell
python -m unittest discover -s tools\tests -v
```

## Build

```powershell
.\gradlew.bat assembleDebug
```

The debug APK is produced under:

- `app/build/outputs/apk/debug/`

## Tests

### JVM unit tests

```powershell
$env:JAVA_HOME='C:\Program Files\Android\Android Studio\jbr'
$env:GRADLE_USER_HOME=(Resolve-Path '.gradle-home').Path
$env:ANDROID_USER_HOME=(Resolve-Path '.android-home').Path
.\gradlew.bat testDebugUnitTest --no-daemon
```

### Android UI test compilation

```powershell
$env:JAVA_HOME='C:\Program Files\Android\Android Studio\jbr'
$env:GRADLE_USER_HOME=(Resolve-Path '.gradle-home').Path
$env:ANDROID_USER_HOME=(Resolve-Path '.android-home').Path
.\gradlew.bat assembleDebugAndroidTest --no-daemon
```

### PDF parser tests

```powershell
python -m unittest discover -s tools\tests -v
```

## Notes

- The repository includes generated assets so the app builds without requiring Python at build time.
- The first translation pass is the slow step because it fills offline learning-language fields; reruns are much faster because `tools/translation_cache.json` is reused.
- The Android UI tests are included and compile successfully. Running them still requires an emulator or device.
- The project uses the AGP 9 built-in Kotlin toolchain with `com.android.legacy-kapt` for Room code generation because it is the most stable option in this environment.

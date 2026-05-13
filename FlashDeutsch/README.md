# FlashEnglish

FlashEnglish is a native Android flashcard app for English recall practice with Persian support prompts. It is fully offline after install and uses the bundled English/Persian study PDFs in [`app/src/main/assets/pdfs`](app/src/main/assets/pdfs) as its source material.

## What It Builds

- Native Android app
- Kotlin + Jetpack Compose + Material 3
- Navigation Compose
- MVVM with screen-level `ViewModel`s
- Room for offline persistence
- DataStore for settings
- Offline deck import from generated JSON assets

## Source PDFs

The repository currently imports English/Persian sentence and vocabulary PDFs for A1, A2, B1, B2, and C1.

The generated import summary currently contains:

- 9 decks
- 5075 cards
- A1 sentence PDF warning: no extractable text, so that source needs OCR or a selectable-text export before import

## Parsing Strategy

The app does **not** parse PDFs on-device. Instead, it uses a build-time normalization pipeline:

1. PDFs are stored in `app/src/main/assets/pdfs/`.
2. [`tools/generate_decks.py`](tools/generate_decks.py) reads the PDFs with `pypdf`.
3. The script detects sentence vs vocabulary decks from the observed line format.
4. It preserves deck names, topics, and source filenames.
5. It normalizes Persian text into logical RTL order.
6. It mirrors the English target into optional `english*` fields and keeps Persian support text from the source PDFs.
7. It writes:
   - [`app/src/main/assets/generated/decks.json`](app/src/main/assets/generated/decks.json)
   - [`app/src/main/assets/generated/import_summary.json`](app/src/main/assets/generated/import_summary.json)
8. The Android app imports the generated JSON into Room on first launch or when re-import is triggered from Settings.

This is more reliable than on-device PDF parsing because the PDF text structure is consistent but not clean enough to trust runtime extraction across devices.

## App Features

- Workflow-first browsing by library, type, category, and generated study set
- Flashcards with flip, next, previous, shuffle, hard, and favorite
- Resume last studied position per deck and mode
- Search across English, Persian, categories, and study metadata
- Saved favorite and hard-card decks
- Persian/Arabic RTL rendering and English LTR rendering
- English device TTS with visible-side autoplay/read-mode support
- Theme selection: system, light, dark
- First-run selection for app language and learning language
- Font scale, learning-language-first vocabulary side order, shuffle default, and haptics settings
- Reset progress and re-import actions
- Learner-language selection is content-aware: only languages with imported card content are shown in setup and Settings

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
    java/com/flashenglish/
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

This step depends on selectable PDF text. If a PDF has no extractable text, regenerate it as selectable text or run OCR before importing.

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

## How To Add A Learner Language Pack

`German` and `English` remain the only app-interface languages. Learner-language packs are separate study-content layers.

The app schema already supports `Persian`, `Italian`, `Spanish`, `Russian`, `Arabic`, `Chinese`, and `French` as learner languages. `Persian` is bundled today. The other learner languages are exposed in the UI only after the generated deck bundle contains content for them.

1. Export a translation template for one learner language:

```powershell
python tools\export_learning_language_template.py --language italian --output tools\content\italian.csv
```

To prepare every optional learner-language pack at once:

```powershell
python tools\export_learning_language_template.py --all --output-dir tools\content
```

2. Fill these CSV columns:
   - `learningPrimary`
   - `learningSupport`

   `targetPrimary` / `targetSupport` and `persianPrimary` / `persianSupport` are included as source context.

3. Merge the completed CSV back into the generated deck bundle and bump the asset version:

```powershell
python tools\merge_learning_language_assets.py --language italian --input tools\content\italian.csv --version 31
```

To merge multiple learner-language packs from a directory in one pass:

```powershell
python tools\merge_learning_language_assets.py --input-dir tools\content --version 31
```

You can also limit the directory merge to specific languages:

```powershell
python tools\merge_learning_language_assets.py --input-dir tools\content --languages italian spanish french --version 31
```

4. Rebuild the app and open it. The new learner language will now appear automatically in first-run setup and Settings because the app derives supported learner languages from imported card content.

5. Inspect current learner-language coverage at any time:

```powershell
python tools\report_learning_language_coverage.py --write
```

6. Verify with:

```powershell
python -m unittest discover -s tools\tests -v
.\gradlew.bat testDebugUnitTest
.\gradlew.bat assembleDebug
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
- The generator is offline. It does not call translation services; it imports English/Persian content from the source PDFs.
- The Android UI tests are included and compile successfully. Running them still requires an emulator or device.
- The project uses the AGP 9 built-in Kotlin toolchain with `com.android.legacy-kapt` for Room code generation because it is the most stable option in this environment.

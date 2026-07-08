[FlashEnglish website](https://farzadj.github.io/FlashEnglish/)

# FlashEnglish

FlashEnglish is a native Android app for structured English study. It combines level-based workflows, sentence decks, vocabulary decks, grammar practice, grammar explanations, milestone study paths, saved review, speech playback, and custom cards in one offline-first app.

The app is built with Kotlin and Jetpack Compose and imports its bundled content from generated JSON assets under [`app/src/main/assets/generated`](app/src/main/assets/generated).

## Current bundle

The current generated bundle in [`app/src/main/assets/generated/import_summary.json`](app/src/main/assets/generated/import_summary.json) is:

- asset version `42`
- `110` generated decks
- `7000` cards
- `0` import warnings

That bundle includes:

- A1, A2, B1, B2, and C1 sentence decks
- A1, A2, B1, B2, and C1 vocabulary decks
- per-level grammar practice decks
- per-level grammar explanation topics

## App capabilities

- Workflow-first home screen with A1 to C1 paths
- Sentence, vocabulary, and grammar workflows for each visible level
- Guided milestone study planning and milestone preview dialogs
- Grammar workflow with:
  - practice sentences
  - explanation topics
- Saved review decks for favorites and hard cards
- Custom card creation and editing
- Offline-first progress and settings persistence
- English speech playback with device TTS
- Free plan:
  - A1 and A2 libraries
  - first 3 milestones per free level
  - 100 custom cards
  - basic completion summaries
- Pro plan:
  - B1, B2, C1, and their grammar paths
  - all 10 milestones per level
  - Favorite/Hard marking, saved decks, and level review
  - unlimited custom cards
  - detailed performance analysis
- App interface languages:
  - English
  - German
- Learner-language support for bundled study content:
  - Persian
  - Italian
  - Spanish
  - Russian
  - Arabic
  - Chinese
  - French

## Tech stack

- Kotlin
- Jetpack Compose
- Material 3
- Navigation Compose
- MVVM with screen-level `ViewModel`s
- Room
- DataStore
- Google Play Billing

## Architecture

### Main packages

- `ui`
  - screens, navigation, reusable Compose components, theming
- `domain`
  - workflows, flashcard models, grammar models, milestone planning, speech logic
- `data`
  - repository orchestration and app/domain mapping
- `parser/importer`
  - generated asset models and asset import logic
- `database`
  - Room entities, DAOs, and database wiring
- `settings`
  - DataStore-backed settings repository

### Data flow

1. App startup triggers repository initialization.
2. The repository checks the imported asset version in DataStore.
3. If needed, generated JSON assets are imported into Room.
4. Screen `ViewModel`s expose immutable UI state from repository `Flow`s.
5. User actions flow back through `ViewModel`s into repository mutations.
6. Settings drive app language, learner language, theme, TTS behavior, and study preferences.

## Study-content model

The app does not parse PDFs on-device. It uses a build-time pipeline:

1. Source PDFs live at the repo root and in [`app/src/main/assets/pdfs`](app/src/main/assets/pdfs).
2. [`tools/generate_decks.py`](tools/generate_decks.py) reads and normalizes deck content.
3. Generated output is written into:
   - [`app/src/main/assets/generated/decks.json`](app/src/main/assets/generated/decks.json)
   - [`app/src/main/assets/generated/grammar.json`](app/src/main/assets/generated/grammar.json)
   - [`app/src/main/assets/generated/import_summary.json`](app/src/main/assets/generated/import_summary.json)
   - [`app/src/main/assets/generated/learning_language_packs.json`](app/src/main/assets/generated/learning_language_packs.json)
4. The Android app imports those generated assets into Room on first launch or on re-import.

This is intentionally more reliable than runtime PDF extraction on Android devices.

## Learner-language coverage

Current learner-language coverage is tracked in [`app/src/main/assets/generated/learning_language_packs.json`](app/src/main/assets/generated/learning_language_packs.json).

At the current bundle version:

- all bundled learner languages are available in-app
- primary coverage is `100%` for all bundled learner languages
- support coverage is `100%` for all bundled learner languages

The app interface itself still remains English/German only.

## Grammar support

Each level now includes a grammar workflow:

- `Practice Sentences`
- `Grammar Explanations`

Grammar explanations are localized for the bundled learner languages and rendered bilingually in-app through the grammar models and UI.

## Website and legal pages

The GitHub Pages site lives under [`docs`](docs):

- [`docs/index.html`](docs/index.html)
- [`docs/privacy.html`](docs/privacy.html)
- [`docs/terms.html`](docs/terms.html)
- [`docs/support.html`](docs/support.html)

The intended published URL is:

- `https://farzadj.github.io/FlashEnglish/`

To publish it through GitHub Pages, configure the repository Pages source to the default branch `docs/` folder.

## Project layout

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
docs/
tools/
  generate_decks.py
  localize_grammar_explanations.py
  auto_translate_learning_languages.py
  merge_learning_language_assets.py
  report_learning_language_coverage.py
  tests/
```

## Content tooling

### Regenerate deck and grammar assets

```powershell
python tools\generate_decks.py --version 43
```

If you change parsing rules or grammar extraction, also verify:

```powershell
python -m unittest discover -s tools\tests -v
python tools\audit_flashcards.py
```

### Export learner-language templates

```powershell
python tools\export_learning_language_template.py --language italian --output tools\content\italian.csv
```

Export every optional learner-language template:

```powershell
python tools\export_learning_language_template.py --all --output-dir tools\content
```

### Merge learner-language packs

```powershell
python tools\merge_learning_language_assets.py --language italian --input tools\content\italian.csv --version 43
```

Merge multiple packs from one directory:

```powershell
python tools\merge_learning_language_assets.py --input-dir tools\content --version 43
```

### Auto-translate learner-language fields

```powershell
python tools\auto_translate_learning_languages.py
```

### Localize grammar explanations

```powershell
python tools\localize_grammar_explanations.py
```

### Report learner-language coverage

```powershell
python tools\report_learning_language_coverage.py --write
```

## Build

```powershell
$env:JAVA_HOME='C:\Program Files\Android\Android Studio\jbr'
.\gradlew.bat assembleDebug --console=plain
```

The debug APK is produced under:

- `app/build/outputs/apk/debug/`

## Tests

### JVM unit tests

```powershell
$env:JAVA_HOME='C:\Program Files\Android\Android Studio\jbr'
.\gradlew.bat testDebugUnitTest --console=plain
```

### Android UI test compilation

```powershell
$env:JAVA_HOME='C:\Program Files\Android\Android Studio\jbr'
.\gradlew.bat assembleDebugAndroidTest --console=plain
```

### Python content-tool tests

```powershell
python -m unittest discover -s tools\tests -v
```

### Flashcard audit

```powershell
python tools\audit_flashcards.py
```

## Notes

- The repository includes generated assets, so Android builds do not require Python at build time.
- The content pipeline is intentionally offline and file-based.
- Learner-language availability in the UI is derived from imported card content, not from a hardcoded language list.
- Android UI tests are included, but running them still requires an emulator or device.
- The repo also contains a GitHub Pages site in `docs/` for landing, support, privacy, and terms pages.


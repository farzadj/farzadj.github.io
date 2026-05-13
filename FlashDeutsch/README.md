[FlashDeutsch website](https://farzadj.github.io/FlashDeutsch/)

# FlashDeutsch

FlashDeutsch is a native Android app for learning German from `A1` to `B2` with offline flashcards, grammar explanations, grammar practice, milestone-based study paths, saved review, custom cards, speech playback, and a dedicated workplace vocabulary library.

The app is built with Kotlin, Jetpack Compose, Room, and DataStore. Study content is generated offline from bundled PDF sources and imported into the app as JSON assets on first launch.

## Highlights

- `A1`, `A2`, `B1`, `B1+`, and `B2` learning workflows
- Separate sentence, vocabulary, and grammar study paths
- `Study mode` milestone journey with 10 milestone steps per level
- Localized grammar explanations:
  - German source first
  - learner-language translation underneath
- Saved `Favorite` and `Hard` review
- Custom flashcard creation and editing
- German text-to-speech playback
- Optional ad-supported unlock flows and a Google Play Pro subscription
- Jobs / workplace vocabulary workflow with dedicated decks
- Fully offline core learning after install

## Supported Languages

The app currently supports the following learner languages for flashcards and grammar explanations:

- English
- Persian
- Italian
- Spanish
- Russian
- Arabic
- Chinese
- French

The app UI also supports switching app language and learning language inside the app.

## Content Snapshot

Current generated asset snapshot from [`app/src/main/assets/generated/import_summary.json`](app/src/main/assets/generated/import_summary.json):

- asset version: `23`
- decks: `119`
- cards: `7259`
- import warnings tracked in summary: `168`

Main content groups:

- level sentence decks
- level vocabulary decks
- grammar explanation topics
- grammar practice decks
- workplace / jobs vocabulary decks

Source PDFs live in [`app/src/main/assets/pdfs/`](app/src/main/assets/pdfs/).

## Tech Stack

- Kotlin
- Jetpack Compose
- Material 3
- Navigation Compose
- Room
- DataStore Preferences
- Gson
- Google Mobile Ads SDK
- Google UMP consent SDK
- Google Play Billing

Build config:

- `minSdk = 26`
- `targetSdk = 36`
- `compileSdk = 36`
- `applicationId = com.fj.flashdeutsch`

## Architecture

### Main packages

- `app/src/main/java/com/flashdeutsch/ui`
  - screens, navigation, reusable Compose UI, theme, speech
- `app/src/main/java/com/flashdeutsch/domain`
  - app models, milestone planning, study-mode logic
- `app/src/main/java/com/flashdeutsch/data`
  - repository orchestration and domain mapping
- `app/src/main/java/com/flashdeutsch/database`
  - Room entities, DAOs, migrations, database
- `app/src/main/java/com/flashdeutsch/parser/importer`
  - generated asset models and import layer
- `app/src/main/java/com/flashdeutsch/settings`
  - DataStore-backed settings and local unlock state
- `tools/`
  - PDF parsing, asset generation, localization overrides, QA scripts, tests

### Data flow

1. The app reads generated JSON assets from `assets/generated/`.
2. On first launch or asset-version change, the repository imports those assets into Room.
3. Screen `ViewModel`s observe repository `Flow`s and expose immutable UI state.
4. Progress, saved cards, milestone unlocks, app settings, and Pro state are persisted locally.
5. Ads, consent, and Play Billing are integrated on top of the offline core app flow.

## Content Pipeline

The app does **not** parse PDFs on-device. PDFs are normalized at build/content-generation time.

Pipeline:

1. Source PDFs are stored in [`app/src/main/assets/pdfs/`](app/src/main/assets/pdfs/).
2. [`tools/generate_decks.py`](tools/generate_decks.py) parses and normalizes the content.
3. The script generates:
   - [`app/src/main/assets/generated/decks.json`](app/src/main/assets/generated/decks.json)
   - [`app/src/main/assets/generated/grammar.json`](app/src/main/assets/generated/grammar.json)
   - [`app/src/main/assets/generated/import_summary.json`](app/src/main/assets/generated/import_summary.json)
   - [`tools/grammar_localization_report.json`](tools/grammar_localization_report.json)
4. Translation and normalization caches/overrides live under `tools/`, including:
   - [`tools/translation_cache.json`](tools/translation_cache.json)
   - `grammar_translation_overrides*.json`
5. The Android app imports the generated JSON into Room.

This approach avoids runtime PDF parsing issues and keeps the app stable and offline-first.

## Grammar Localization

Grammar explanation topics are localized offline and then manually refined through override files under `tools/`.

Current QA status from [`tools/grammar_localization_report.json`](tools/grammar_localization_report.json):

- Persian: `0` missing, `0` suspicious
- Italian: `0` missing, `0` suspicious
- Spanish: `0` missing, `0` suspicious
- Russian: `0` missing, `0` suspicious
- Arabic: `0` missing, `0` suspicious
- Chinese: `0` missing, `0` suspicious
- French: `0` missing, `0` suspicious

## Jobs / Workplace Library

The app includes a separate jobs workflow on the home page. It is built from these PDF sources:

- `german_beruf_jobs_20_topics_1000_words_persian_examples_varied_sentences.pdf`
- `german_beruf_jobs_set2_20_topics_1000_words_persian_examples.pdf`
- `german_beruf_specialist_jobs_set3_20_topics_1000_words_persian_examples.pdf`
- `german_beruf_jobs_set1_natural_sentences.pdf`
- `german_beruf_jobs_set2_natural_sentences.pdf`
- `german_beruf_specialist_jobs_set3_natural_sentences_v2.pdf`

The jobs workflow uses vocabulary-style flashcards with example sentences and is available alongside the CEFR level workflows.

## Build

### Debug build

```powershell
$env:JAVA_HOME='C:\Program Files\Android\Android Studio\jbr'
$env:Path="$env:JAVA_HOME\bin;$env:Path"
.\gradlew assembleDebug
```

Debug APK output:

- `app/build/outputs/apk/debug/app-debug.apk`

### Unit tests

```powershell
python -m unittest tools.tests.test_generate_decks
```

```powershell
$env:JAVA_HOME='C:\Program Files\Android\Android Studio\jbr'
$env:Path="$env:JAVA_HOME\bin;$env:Path"
.\gradlew testDebugUnitTest
```

### Android test compilation

```powershell
$env:JAVA_HOME='C:\Program Files\Android\Android Studio\jbr'
$env:Path="$env:JAVA_HOME\bin;$env:Path"
.\gradlew assembleDebugAndroidTest
```

## Regenerating Assets

If you add or change PDF content or localization overrides:

```powershell
python tools\generate_decks.py --version 23
```

If you only want to re-apply grammar overrides/polish against the current generated bundle, use the existing Python helpers in `tools/generate_decks.py`.

## Release Signing

Release builds read signing values from [`local.properties`](local.properties).

Expected keys:

```properties
FLASHDEUTSCH_STORE_FILE=...
FLASHDEUTSCH_STORE_PASSWORD=...
FLASHDEUTSCH_KEY_ALIAS=...
FLASHDEUTSCH_KEY_PASSWORD=...
FLASHDEUTSCH_PLAY_PRO_SUBSCRIPTION_PRODUCT_ID=flashdeutsch_pro_monthly
```

Then build:

```powershell
$env:JAVA_HOME='C:\Program Files\Android\Android Studio\jbr'
$env:Path="$env:JAVA_HOME\bin;$env:Path"
.\gradlew bundleRelease
```

Release AAB output:

- `app/build/outputs/bundle/release/app-release.aab`

## Website and Legal Pages

The repository includes the GitHub Pages site under [`docs/`](docs/):

- landing page
- privacy policy
- terms
- support page

Published target:

- `https://farzadj.github.io/FlashDeutsch/`

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
    res/
docs/
tools/
  generate_decks.py
  grammar_translation_overrides*.json
  tests/
```

## Notes

- Generated assets are committed, so Android builds do not require Python.
- The first translation/content generation pass is the slow step; reruns are faster because caches and manual overrides are reused.
- Core learning works offline after install, but ads, consent, and subscription flows require network access.
- The current app uses local device persistence for milestone unlocks, settings, progress, saved cards, and custom cards.

- The repository includes generated assets so the app builds without requiring Python at build time.
- The first translation pass is the slow step because it fills offline learning-language fields; reruns are much faster because `tools/translation_cache.json` is reused.
- The Android UI tests are included and compile successfully. Running them still requires an emulator or device.
- The project uses the AGP 9 built-in Kotlin toolchain with `com.android.legacy-kapt` for Room code generation because it is the most stable option in this environment.

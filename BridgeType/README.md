[BridgeType website](https://farzadj.github.io/BridgeType/)


# BridgeType

An Android Kotlin app that provides a Play Store safe translator keyboard for chat apps.

The app is designed to work with normal text input fields in apps such as WhatsApp, Telegram, Signal, SMS, Instagram DMs, and other messengers. It does not hack, modify, inspect, or overlay other apps.

## Features

- Custom Android keyboard / IME
  - Compose outgoing messages inside the keyboard.
  - Translate before inserting into the active messenger text field.
  - Type directly into the active messenger text field when the cursor is there.
  - Live mode can translate while the user types in the keyboard draft.
  - Receive mode supports pasted or dictated incoming text.

- Selected text translator
  - Supports Android `ACTION_PROCESS_TEXT`.
  - Supports shared text through `ACTION_SEND`.
  - Shows original and translated text.
  - Provides copy, share, retry, and use-translation actions where Android allows it.

- Translation engine
  - Uses ML Kit on-device translation.
  - Supports language detection through ML Kit.
  - Handles missing model downloads and model status.
  - Uses a clean `TranslationEngine` abstraction for future cloud/API engines.

- Keyboard customization
  - Samsung-style, Mint, Midnight, and High Contrast style options.
  - Full keyboard color themes.
  - Saved mode, source language, target language, typing layout, style, and color.
  - Per-language keyboard layouts, including Latin, Persian, Arabic, Hindi, Russian, Chinese Pinyin, Japanese Romaji, and Korean input support.

- Voice input
  - Toolbar mic button in Compose, Receive, and Live modes.
  - Uses Android `SpeechRecognizer`.
  - Voice timing settings are available in the keyboard settings popup.

## Privacy and Play Store Safety

This project intentionally avoids risky messenger-app access patterns:

- No `AccessibilityService`.
- No screen scraping.
- No reading chat bubbles or private chat screens.
- No clipboard monitoring.
- No message storage.
- No modification of other apps.

The keyboard only processes:

- text typed by the user inside the keyboard,
- text inserted through normal IME APIs into the focused field,
- text explicitly selected or shared by the user through Android actions,
- speech recorded after the user taps the mic button.

Clipboard access is only used after an explicit user action, such as pressing paste/copy. The app does not listen for clipboard changes.

## Tech Stack

- Kotlin
- Jetpack Compose
- Android `InputMethodService`
- ML Kit Translation
- ML Kit Language ID
- Android `ACTION_PROCESS_TEXT`
- Android `ACTION_SEND`
- Simple dependency container

## Project Structure

```text
app/src/main/java/com/fjgup/keyboardtranslator/
  data/          ML Kit implementation and settings persistence
  domain/        translation abstractions, language catalog, settings contracts
  keyboard/      InputMethodService and keyboard UI
  presentation/  main Compose app UI
  processText/   selected/shared text translation screen
```

## Android Components

The app declares:

- `TranslatorKeyboardService` as an Android IME.
- `ProcessTextActivity` for selected text translation.
- `ProcessTextActivity` for shared text translation.
- Internet permission for ML Kit model download.
- Record audio permission for explicit voice recognition.

## Build

Open the project in Android Studio, or build from the command line:

```powershell
$env:JAVA_HOME='C:\Program Files\Android\Android Studio\jbr'
$env:Path="$env:JAVA_HOME\bin;$env:Path"
.\gradlew.bat build
```

## GitHub Pages

The static app website is in `docs/`.

To publish it at `https://farzadj.github.io/BridgeType/`, configure GitHub Pages to deploy from the `main` branch using the `/docs` folder in a repository named `BridgeType`.

- Landing page: `docs/index.html`
- Privacy policy: `docs/privacy.html`
- Terms of use: `docs/terms.html`
- Support page: `docs/support.html`

## Setup on Device

1. Install the app.
2. Open the app onboarding screen.
3. Enable `BridgeType Keyboard` in Android keyboard settings.
4. Choose `BridgeType Keyboard` while typing in a messenger.
5. Download needed ML Kit language packs.
6. Use Compose, Receive, or Live mode from the keyboard toolbar.

## Notes

- The keyboard uses standard IME APIs only.
- True automatic reading of unselected messenger messages is intentionally not implemented because it would require unsafe screen access.
- Some apps may limit text selection or replacement. In those cases, use Android share actions when available.
- ML Kit model downloads may require network access during setup, but translation runs on device after the model is available.


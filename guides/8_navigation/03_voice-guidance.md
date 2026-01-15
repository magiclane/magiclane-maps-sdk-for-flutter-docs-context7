---
description: Documentation for Voice Guidance
title: Voice Guidance
---

# Add voice guidance

Enhance navigation experiences with spoken instructions. This guide covers enabling built-in Text-to-Speech (TTS), managing voice settings, switching voices and languages, and integrating custom playback.

---

## What you need

The Maps SDK for Flutter provides two options for instruction playback:

- **Built-in solutions** — playback using human voice recordings or computer-generated TTS

- **External integration** — delivery of TTS strings for use with third-party packages

The built-in solution provides automatic audio session management, ducking other playbacks (such as music) while instructions play.

---

## Step 1: Enable voice guidance

Set the `canPlaySounds` flag of the `SoundPlayingService` to true:
```dart
SoundPlayingService.canPlaySounds = true;
```

Use the `canPlaySounds` getter and setter to check or change whether the SDK plays instructions during navigation or simulation.

Ensure a valid TTS voice is configured, voice volume is set to a positive value, and the `canPlaySounds` flag is enabled to automatically play voice instructions.

By default, the current voice is set to the best computer TTS voice matching the default SDK language.

**Limitations:**  
Customizing the timing of TTS instructions is not supported. Filtering TTS instructions based on custom logic is not available.

---

## Step 2: Configure the Sound Playing Service

The `SoundPlayingService` manages voice playback with the following features:

| Member                            | Description                                                                |
|-----------------------------------|----------------------------------------------------------------------------|
| `playText(text)`                  | Plays the given TTS text string (computer voices only) (async)             |
| `voiceVolume`                     | Gets or sets volume level for voice playback (0-10)                        |
| `canPlaySounds`                   | Gets or sets whether automatic TTS instruction playback is enabled         |
| `cancelNavigationSoundsPlaying()` | Cancels ongoing navigation-related sound playback                          |
| `soundPlayingListener`            | Sound listener providing details about sound playing events                |

---

## Step 3: Select and configure voices

The SDK provides voices for each supported language. Download and activate voices to deliver navigation prompts such as turn instructions, warnings, and announcements.

### Voice types

The SDK offers two types of voice guidance:

- **`VoiceType.human`** — Pre-recorded human voices delivering instructions in a natural tone. Supports basic instruction types only; **does not** include road or settlement names.

- **`VoiceType.computer`** — Device Text-to-Speech (TTS) engine providing detailed, flexible guidance. **Fully supports** street and place names. Quality depends on device capabilities.

### Voice structure

The `Voice` class contains:

| Property    | Type        | Description                                                      |
|-------------|-------------|------------------------------------------------------------------|
| `id`        | int         | Unique identifier for the voice                                  |
| `name`      | String      | Display name of the voice                                        |
| `fileName`  | String      | File name used to load the voice (available for `VoiceType.human`) |
| `language`  | Language    | Associated language object                                       |
| `type`      | VoiceType   | Voice type: `human` or `computer`                                |

⚠️ **Do not confuse `Voice` and `Language` concepts**

- **`Language`** defines **what** is said — words, phrasing, and localization

- **`Voice`** defines **how** it is said — accent, tone, and gender

Ensure the selected `Voice` is compatible with the chosen `Language`. Mismatched combinations may result in unnatural or incorrect pronunciation.

**Relevance:**

- `Language` — relevant for built-in TTS and custom solutions using `onTextToSpeechInstruction`. See the [internationalization guide](/guides/get-started/internationalization).

- `Voice` — relevant **only** for built-in voice-guidance (human and computer voices).

⚠️ **Two language settings**

The SDK distinguishes between:

- **SDK language** (`SdkSettings.language`) — language for on-screen text and UI strings

- **Voice language** (`Voice.language`) — language for spoken output (built-in engine or `onTextToSpeechInstruction` callback)

Both use the same `Language` class. Synchronize SDK language and voice language based on your use case.

### Get the current voice

Use the `voice` getter from `SdkSettings`:
```dart
Voice currentVoice = SdkSettings.voice;
```

### Get available human voices

Retrieve the available human voices list using `getLocalContentList` from the `ContentStore` class. Voice details are stored in the `contentParameters` field of each `ContentStoreItem`. Use `contentParametersAs` getter to safely cast to `VoiceParameters`. 
```dart
List<ContentStoreItem> items = ContentStore.getLocalContentList(ContentType.humanVoice);

for (final contentStoreItem in items) {
  final String name = contentStoreItem.name;
  final String filePath = contentStoreItem.fileName;
  
  final VoiceParameters? voiceContentParameters =
      contentStoreItem.contentParametersAs<VoiceParameters>();
  
  if (voiceContentParameters == null) {
    print("Content parameters are null for voice: $name");
    continue;
  }
  
  final String? languageCode = voiceContentParameters.language;
  final String? gender = voiceContentParameters.gender;
  final VoiceType? type = voiceContentParameters.type;
  final String? nativeLanguage = voiceContentParameters.nativeLanguage;
}
```

See the [Manage Content Guide](../offline/manage-content) for downloading, deleting, and managing voices, plus details about `ContentStore` and `ContentStoreItem`.

### Apply a voice by path

Provide the absolute path (from `ContentStoreItem.fileName`) to `setVoiceByPath`:
```dart
String filePath = contentStoreItem.fileName;
GemError error = SdkSettings.setVoiceByPath(filePath);

if (error != GemError.success) {
  print("Applying the voice failed.");
}
```

`setVoiceByPath` also supports computer voices if the path is known. Specify the `language` optional parameter with a `Language` instance when setting computer voices to ensure proper matching. The `language` parameter is only relevant for computer voices.

### Apply a voice by language

Apply computer voices using `setTTSVoiceByLanguage` from the `SdkSettings` class:
```dart
Language? lang = SdkSettings.getBestLanguageMatch("eng", regionCode: "GBR");
GemError error = SdkSettings.setTTSVoiceByLanguage(lang!);
```

Computer voices use the device's built-in TTS capabilities.

Selecting a computer voice in an unsupported language may cause a mismatch between spoken voice and instruction content. Exact behavior depends on device TTS capabilities.

---

## Step 4: Integrate external TTS (optional)

The `NavigationService` provides TTS instructions as strings for processing with external tools. Use the [flutter_tts](https://pub.dev/packages/flutter_tts) package or other TTS solutions to play instructions.

Add `flutter_tts` to your `pubspec.yaml` and run `dart pub get`.

Use the `onTextToSpeechInstruction` callback during simulation:
```dart
// instantiate FlutterTts
FlutterTts flutterTts = FlutterTts();

void simulationInstructionUpdated(NavigationInstruction instruction, Set<NavigationInstructionUpdateEvents> events) {
  // handle instruction
}

// highlight-start
void textToSpeechInstructionUpdated(String ttsInstruction) {
  flutterTts.speak(ttsInstruction);
}
// highlight-end

TaskHandler? taskHandler = NavigationService.startSimulation(
  route,
  onNavigationInstruction: simulationInstructionUpdated,
  // highlight-start
  onTextToSpeechInstruction: textToSpeechInstructionUpdated,
  // highlight-end
  speedMultiplier: 2,
);
```

For navigation, set the `onTextToSpeechInstruction` callback similarly with `startNavigation`.

See [flutter_tts](https://pub.dev/packages/flutter_tts) documentation for setting TTS voice, language, pitch, volume, speechRate, and other options.

**Change instruction language:**

- Use `setTTSVoiceByLanguage` and specify the preferred language

- Or use `setTTSVoiceByPath` with a voice path for the desired language

**Disable internal playback:**  
Set `SoundPlayingService.canPlaySounds` to `false`. Instructions still arrive via the callback, but no audio plays.

---

## Step 5: Monitor sound playback events

The `SoundPlayingListener` class observes sound playback events, including navigation TTS instructions, custom text, audio files, and alerts.

**Registration methods:**

- `registerOnStart` — triggered when a sound starts playing

- `registerOnStop` — triggered when a sound finishes playing. Error is `GemError.success` if completed, or other errors if canceled

- `registerOnVolumeChangedByKeys` — triggered when the user adjusts volume using physical device buttons

Register callbacks using the singleton instance from `SoundPlayingService.soundPlayingListener`:
```dart
final listener = SoundPlayingService.soundPlayingListener;

listener.registerOnStart(() {
  // Sound started playing
});

listener.registerOnStop((error) {
  // Sound finished playing
});

listener.registerOnVolumeChangedByKeys((newVolume) {
  // Volume changed
});
```

**Retrieve details about currently playing sound:**

- `soundPlayType` — playback type (none, custom text, audio file, alert, navigation TTS, or custom sound by ID)

- `soundPlayContent` — playback string (only for `SoundPlayType.navigationSound` or `SoundPlayType.soundById`)

- `soundPlayFileName` — audio file name (only for `SoundPlayType.file` or `SoundPlayType.alert`)

Only one sound can play at a time. The `SoundPlayingListener` instance persists from SDK initialization to release.

---

## Step 6: Play custom instructions

Use the `playText` method of `SoundPlayingService` to play custom instructions (e.g., road warnings or social reports). This uses the currently selected computer voice and is **not available for human voices**.

The optional `severity` parameter determines interrupt behavior:

- `information` — queued and played **after** current playback

- `warning` — **interrupts** current playback if it has lower priority

Ensure the provided string matches the voice language.

See [speed warnings](../alarms/speed-alarms) and [landmark & overlay alarms](../alarms/landmark-and-overlay-alarms) for notifications about speed warnings and reports.

---

## Relevant examples demonstrating voice features

- [Human Voices](/examples/routing-navigation/human-voices)

- [Voice Download](/examples/routing-navigation/voice-download)

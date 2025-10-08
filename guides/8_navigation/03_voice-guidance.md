---
description: Documentation for Voice Guidance
title: Voice Guidance
---

# Add voice guidance

Voice guidance in the Maps SDK for Flutter allows you to enhance navigation experiences with spoken instructions. This guide covers how to enable built-in Text-to-Speech (TTS), manage voice settings, switch voices and languages, and integrate custom playback using the `onTextToSpeechInstruction` callback for maximum flexibility.

The Maps SDK for Flutter provides two options for instruction playback:

- **Built-in solutions** — playback using human voice recordings or computer-generated TTS.

- **External integration** — delivery of TTS strings for use with third-party packages.

The built-in solution also provides automatic audio session management, ducking other playbacks (such as music) while instructions are playing.

## Basic Voice Guidance

In order to activate the automatic play of TTS instructions using the built-in voice solution, set the `canPlaySounds` flag of the `SoundPlayingService` to true:
```dart
// Set auto play sound to true, so that the voice instructions will be played automatically
SoundPlayingService.canPlaySounds = true;
```

To check or change whether the SDK plays instructions after navigation or simulation begins, use the `canPlaySounds` getter and setter available in the `SoundPlayingService` class.

Ensure that a valid TTS voice is configured, the voice volume is set to a positive value within the allowed range and the `canPlaySounds` flag is enabled during simulation or navigation in order to automatically play voice instructions.

By default, the current voice is set to the best computer TTS voice matching the default SDK language.

Customizing the timing of TTS instructions is not supported. Filtering TTS instructions based on custom logic is not available.

## The Sound Playing Service

The `SoundPlayingService` provides the following features:

| Member                            | Description                                                                |
|-----------------------------------|----------------------------------------------------------------------------|
| `playText(text)`                  | Plays the given TTS text string. Only available when the current voice is a computer voice. |
| `voiceVolume`                     | Gets or sets the volume level used for voice playback. Between 0 and 10    |
| `canPlaySounds`                   | Gets or sets whether automatic TTS instruction playback is enabled.        |
| `cancelNavigationSoundsPlaying()` | Cancels any ongoing navigation-related sound playback.                     |
| `soundPlayingListener`            | The sound listener which provide details about sound playing events        |

## Voice

The Maps SDK for Flutter provides a list of voices for each supported language. These voices can be downloaded and activated to provide navigation prompts such as turn instructions, warnings, and announcements.

The SDK offers two types of voice guidance, defined by the `VoiceType` enum:

- `VoiceType.human`:   Uses pre-recorded human voices to deliver instructions in a friendly and natural tone.  

  These voices support only basic instruction types and **do not** include road or settlement names.

- `VoiceType.computer`:  Leverages the device’s Text-to-Speech (TTS) engine to provide more detailed and flexible guidance.  

  These voices **fully support** street and place names in the spoken instructions. The quality and availability is dependent on the device.

### Voice Structure

The `Voice` class provides the following details:

| Property    | Type        | Description                               |
|-------------|-------------|-------------------------------------------|
| `id`        | int         | Unique identifier for the voice.          |
| `name`      | String      | Display name of the voice.                |
| `fileName`  | String      | File name which can be used to load the voice (available for `VoiceType.human`).         |
| `language`  | Language    | Associated language object.               |
| `type`      | VoiceType   | Enum: `human` or `computer`.              |

Do not confuse the `Voice` and `Language` concepts.

The `Language` defines **what** is said — it determines the words, phrasing, and localization.
The `Voice` defines **how** it is said — it controls attributes like accent, tone, and gender.
Always ensure that the selected `Voice` is compatible with the chosen `Language`, as mismatched combinations may result in unnatural or incorrect pronunciation.

#### Relevance

- `Language` is relevant for both the built-in TTS system and custom solutions using the `onTextToSpeechInstruction` callback. See the [internationalization guide](../2_get-started/05_internationalization.mdx) for more info about the ``Language`` class.

- `Voice` is relevant **only** for the built-in voice-guidance (using human and computer voices) playback.

The SDK distinguishes between two language settings:

- **SDK language** (`SdkSettings.language`) : Defines the language used for all on-screen text and UI intended strings.

- **Voice language** (`Voice.language`) : Defines the language used for spoken output, whether through the built-in engine (computer/human voices) or via the `onTextToSpeechInstruction` callback.

Both settings use the same `Language` class. The synchronization between the SDK language and the voice language should be made by the user, depending on the use case.

### Get the Current Voice

Use the `getVoice` static method provided by the `SdkSettings` class:
```dart
Voice currentVoice = SdkSettings.getVoice();
```

### Get the List of Available Human Voices

The available human voices list is provided by the `getLocalContentList` method provided by the `ContentStore` class.
Details about the voices are stored within the `contentParameters` field of each `OverlayItem` associated with a voice. Since OverlayItems can represent different types of content (maps, voices, etc.), it is recommended to use the `getContentParametersAs` method to safely cast the `contentParameters` to a `VoiceParameters` instance. 
```dart
List<ContentStoreItem> items = ContentStore.getLocalContentList(ContentType.humanVoice);

for (final contentStoreItem in items){
// The voice name (ex: Ella)
final String name = contentStoreItem.name;

// The absolute path to the voice file. Used for applying the voice.
final String filePath = contentStoreItem.fileName;

// Get the content parameters as VoiceParameters
final VoiceParameters? voiceContentParameters =
        contentStoreItem.getContentParametersAs<VoiceParameters>();

if (voiceContentParameters == null) {
    print("Content parameters are null for voice: $name");
    continue;
}

// The BCP 47 Language Code (ex: eng_IRL)
final String? languageCode = voiceContentParameters.language;

// The gender (ex: Female)
final String? gender = voiceContentParameters.gender;

// The voice type (ex: human)
final VoiceType? type = voiceContentParameters.type;

// The native language name (ex: English)
final String? nativeLanguage = voiceContentParameters.nativeLanguage;
}
```

Check the [Manage Content Guide](../offline/manage-content) for information about managing voices and performing operations such as downloading, deleting, and more and details about the `ContentStore` and `ContentStoreItem` classes.

### Apply a Voice by path

A human voice can be applied by providing the absolute path (obtained from the `ContentStoreItem.fileName` getter) to the `setVoiceByPath` method provided by the `SdkSettings` class:
```dart
String filePath = contentStoreItem.fileName;
GemError error = SdkSettings.setVoiceByPath(filePath);

if (error != GemError.success){
  print("Applying the voice failed.");
}
```

The `SdkSettings.setVoiceByPath` method can also be used to set computer voices, provided the computer voice path is known.  

It is recommended to specify the `language` optional parameter with a corresponding `Language` instance when setting computer voices. This helps ensure that the selected voice is properly matched to the intended language.

The `language` parameter of the `setVoiceByPath` is only relevant for computer voices.

### Apply a Voice by language

Computer voices can be applied using the `setTTSVoiceByLanguage` method provided by the `SdkSettings` class. The method requires a `Language` class:
```dart
// Get the language
Language? lang = SdkSettings.getBestLanguageMatch("eng", regionCode: "GBR");

// Apply the best computer voice based on the provided language
GemError error = SdkSettings.setTTSVoiceByLanguage(lang!);
```

The computer voice is implemented using the device included TTS capabilities.

Selecting a computer voice in an unsupported language may cause a mismatch between the spoken voice and the instruction content.
The exact behavior depends on the device and its available text-to-speech capabilities.

## Get the TTS Instruction Strings

The TTS instructions can be provided by the `NavigationService` as strings in order to be further processed and played with external tools. For example, the [flutter_tts](https://pub.dev/packages/flutter_tts) package can be used for playing sound based on the TTS text instructions provided by the SDK. Other packages for TTS can also be used.
The voice instructions strings will come on the ``onTextToSpeechInstruction`` callback set during navigation or simulation.

Add ``flutter_tts`` as a dependency to the ``pubspec.yaml`` file of the project and run the ``dart pub get`` command to install the tts package.

We can use the ``onTextToSpeechInstruction`` callback of the ``startSimulation`` methods in the following way:
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

For navigation we can set the ``onTextToSpeechInstruction`` callback in a similar way with ``startNavigation``.

See the documentation for [flutter_tts](https://pub.dev/packages/flutter_tts) for information on how to set the TTS voice, language and other options such as pitch, volume, speechRate, etc.

To change the language of the instructions provided through the `onTextToSpeechInstruction` callback, you can:

- Use `setTTSVoiceByLanguage` and specify the preferred language.

- Or use `setTTSVoiceByPath` with a voice path corresponding to the desired language.

To disable the internal playback engine, set `SoundPlayingService.canPlaySounds` to `false`.  
The instruction will still be delivered via the callback, but no audio will be played.

## Monitor Sound Playback Events

The `SoundPlayingListener` class allows you to observe events related to sound playback, such as navigation TTS instructions, custom text, audio files, and alerts. It provides the following registration methods:

- `registerOnStart` : Triggered when a sound starts playing  

- `registerOnStop` : Triggered when a sound finishes playing. The provided error is `GemError.success` if the sound has been fully completed or other errors if it was canceled.  

- `registerOnVolumeChangedByKeys` : Triggered when the user adjusts volume using physical device buttons

To register callbacks, use the singleton instance exposed via the `SoundPlayingService.soundPlayingListener` static getter:
```dart
final listener = SoundPlayingService.soundPlayingListener;

listener.registerOnStart(() {
  // A sound has started playing
});

listener.registerOnStop((error) {
  // A sound has finished playing
});

listener.registerOnVolumeChangedByKeys((newVolume) {
  // The volume has been changed
});
```

You can retrieve details about the currently playing sound using the following getters:

- `soundPlayType` : Type of the current playback (e.g., none, custom text, audio file, alert, navigation TTS instructions, or custom sound by ID)

- `soundPlayContent` : Playback string (only for `SoundPlayType.navigationSound` or `SoundPlayType.soundById`; otherwise `null`)

- `soundPlayFileName` : Name of the currently playing audio file (only for `SoundPlayType.file` or `SoundPlayType.alert`; otherwise `null`)

Only one sound can be played at a time.
The `SoundPlayingListener` instance functions like a singleton and the instance is persisted since the SDK initialization until the release.

## Play custom instructions

Sometimes you may need to play custom instructions, such as road information warnings or social reports.

To do this, use the `playText` method of the `SoundPlayingService` to play back a custom string.  
This uses the currently selected computer voice and is **not available for human voices**.

The method accepts an optional `severity` parameter, which determines whether the custom instruction should interrupt playbacks with lower priority:

- `information` — the instruction is queued and played **after** any string currently being played.

- `warning` — the instruction **interrupts** the current playback if it has a lower priority.

Make sure the provided string matches the voice language.

Check the [speed warnings](../alarms/speed-alarms) and [landmark & overlay alarms](../alarms/landmark-and-overlay-alarms) docs to learn how to get notified about speed warnings and reports.

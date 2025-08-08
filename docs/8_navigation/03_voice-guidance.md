---
description: Documentation for Voice Guidance
title: Voice Guidance
---

# Add voice guidance

Voice guidance in the Maps SDK for Flutter allows you to enhance navigation experiences with spoken instructions. This guide covers how to enable built-in Text-to-Speech (TTS), manage voice settings, switch voices and languages, and integrate custom playback using the `onTextToSpeechInstruction` callback for maximum flexibility.

## Basic Voice Guidance

In order to activate the automatic play of TTS instructions using the built-in voice solution, set the `canPlaySounds` flag of the `SoundPlayingService` to true:
```dart
// Set auto play sound to true, so that the voice instructions will be played automatically
SoundPlayingService.canPlaySounds = true;
```

To check or change whether the SDK plays instructions after navigation or simulation begins, use the `canPlaySounds` getter and setter available in the `SoundPlayingService` class.

Ensure that a valid TTS voice is configured, the voice volume is set to a positive value within the alowed range and the `canPlaySounds` flag is enabled during simulation or navigation in order to automatically play voice instructions.

Customizing the timing of TTS instructions is not supported. Filtering TTS instructions based on custom logic is not available.

## The Sound Playing Service

The `SoundPlayingService` provides the following features:

| Member                            | Description                                                                |
|-----------------------------------|----------------------------------------------------------------------------|
| `playText(text)`                  | Plays the given TTS text string.                                           |
| `playSound(soundId)`              | Plays a sound by its identifier.                                           |
| `voiceVolume`                     | Gets or sets the volume level used for voice playback. Between 0 and 10    |
| `canPlaySounds`                   | Gets or sets whether automatic TTS instruction playback is enabled.        |
| `cancelNavigationSoundsPlaying()` | Cancels any ongoing navigation-related sound playback.                     |
| `soundPlayingListener`            | The sound listener which provide details about sound playing events        |

## Change the Text to Speech Instructions Language

Changing the TTS instruction language is relevant both for the built-in sound playing solution and the custom way using the `onTextToSpeechInstruction` callback.

The TTS instruction strings will be played by the SDK (or provided via the `onTextToSpeechInstruction`) in the language set using the ``SdkSettings.setTTSLanguage`` method. The snippet provided below sets the language to English. See the [internationalization guide](../2_get-started/05_internationalization.mdx) for more info about the ``Language`` class.
```dart
Language lang = SdkSettings.languageList.where((lang) => lang.languagecode == 'eng' && lang.regioncode == 'GBR').first;
SdkSettings.setTTSLanguage(lang);
```

Do not confuse the `Voice` and `Language` concepts.

The `Language` defines **what** is said — it determines the words, phrasing, and localization.
The `Voice` defines **how** it is said — it controls attributes like accent, tone, and gender.
Always ensure that the selected `Voice` is compatible with the chosen `Language`, as mismatched combinations may result in unnatural or incorrect pronunciation.

`Language` is relevant for both the built-in TTS system and custom solutions using the `onTextToSpeechInstruction` callback.
`Voice` is relevant **only** for the built-in TTS playback.

## Change the voice

The Maps SDK for Flutter provides a list of voices for each supported language. These voices can be downloaded and activated to provide navigation prompts such as turn instructions, warnings, and announcements.

### Voice Structure

The `Voice` class provides the following details:

| Property    | Type        | Description                               |
|-------------|-------------|-------------------------------------------|
| `id`        | int         | Unique identifier for the voice.          |
| `name`      | String      | Display name of the voice.                |
| `fileName`  | String      | File name which can be used to load the voice (available for `VoiceType.human`).         |
| `language`  | Language    | Associated language object.               |
| `type`      | VoiceType   | Enum: `human` or `computer`.              |

The SDK offers two types of voice guidance, defined by the `VoiceType` enum:

- `VoiceType.human`:   Uses pre-recorded human voices to deliver instructions in a friendly and natural tone.  

  These voices support only basic instruction types and **do not** include road or settlement names.

- `VoiceType.computer`:  Leverages the device’s Text-to-Speech (TTS) engine to provide more detailed and flexible guidance.  

  These voices **fully support** street and place names in the spoken instructions.

### Get the Current Voice

Use the `getVoice` static method provided by the `SdkSettings` class:
```dart
Voice currentVoice = SdkSettings.getVoice();
```

### Get the List of Available Human Voices

The available human voices list is provided by the `getLocalContentList` method provided by the `ContentStore` class.
Details about the voices are stored within the `contentParameters` field of each `OverlayItem` associated with a voice.
```dart
List<ContentStoreItem> items = ContentStore.getLocalContentList(ContentType.humanVoice);

for (final item in items){
  // The voice name (ex: Ella)
  final String name = contentStoreItem.name;

  // The absolute path to the voice file. Used for applying the voice.
  final String filePath = contentStoreItem.fileName;

  // The BCP 47 Language Code (ex: eng_IRL)
  final String languageCode = contentStoreItem.contentParameters.findParameter("language").value;
  
  // The gender (ex: Female)
  final String gender = contentStoreItem.contentParameters.findParameter("gender").value;
  
  // The voice type (ex: human)
  final String type = contentStoreItem.contentParameters.findParameter("type").value;
  
  // The native language name (ex: English)
  final String nativeLanguage = contentStoreItem.contentParameters.findParameter("native_language").value;
}
```

Check the [Manage Content Guide](../offline/manage-content) for information about managing voices and performing operations such as downloading, deleting, and more and details about the `ContentStore` and `ContentStoreItem` classes.

### Apply a Human Voice

A human voice can be applied by providing the absolute path to the `setVoiceByPath` method provided by the `SdkSettings` class:
```dart
String filePath = ...
GemError error = SdkSettings.setVoiceByPath(filePath);

if (error != GemError.success){
  print("Applying the voice failed.");
}
```

### Apply a Computer Voice

Computer voices are applied using the `setTTSVoiceByLanguage` method provided by the `SdkSettings` class. The method requires a `Language` class:
```dart
// Get the language
Language lang = SdkSettings.languageList.where((lang) => lang.languagecode == 'eng' && lang.regioncode == 'GBR').first;

// Apply the best computer voice based on the provided laguage
GemError error = SdkSettings.setTTSVoiceByLanguage(lang);
```

The computer voice is implemented using the device included TTS capabilities.

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
  null,
  onNavigationInstruction: simulationInstructionUpdated,
  // highlight-start
  onTextToSpeechInstruction: textToSpeechInstructionUpdated,
  // highlight-end
  speedMultiplier: 2,
);
```

For navigation we can set the ``onTextToSpeechInstruction`` callback in a similar way with ``startNavigation``.

See the documentation for [flutter_tts](https://pub.dev/packages/flutter_tts) for information on how to set the TTS voice, language and other options such as pitch, volume, speechRate, etc.

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

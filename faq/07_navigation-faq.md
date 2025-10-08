---
description: Documentation for Navigation Faq
title: Navigation Faq
---

# Navigation FAQ

<sub style={{ color: 'gray', display: 'inline-block' }}>
  
</sub>

# How to set the language for the onTextSpeechInstruction feature in navigation?

Use the `setTTSVoiceByLanguage` method of the `SdkSettings`. This will change the language used for the navigation voice instructions. Keep in mind that the language for the `flutter_tts` package also needs to be changed accordingly. See the [voice guidance section](/guides/navigation/voice-guidance) for more details.

For example, to set the language to Spanish for navigation voice instructions:
```dart
final languages = SdkSettings.languageList;
final spanish = SdkSettings.getBestLanguageMatch("spa");
SdkSettings.setTTSVoiceByLanguage(spanish);
```

## How can I skip intermediate text instructions, including voice and graphic instructions?

You can do this, before starting the navigation:
```dart
Debug.setNavigationModifiers({
      NavigationModifiers.disableIntermediateWaypoints
});
```



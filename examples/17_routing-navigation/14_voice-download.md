---
description: Documentation for Voice Download
title: Voice Download
---

# Voice download

In this guide, you will learn how to list the voice downloads available on the server, how to download a voice, and track the download progress.

## How It Works

The example app demonstrates the following features:

- Get a list of available voices

- Download a voice

### Initialize the Map

This callback function is called when the interactive map is initialized and ready to use. Allows mobile data usage for content downloads.
```dart
void _onMapCreated(GemMapController controller) async {
  SdkSettings.setAllowOffboardServiceOnExtraChargedNetwork(ServiceGroupType.contentService, true);
}
```

### Navigate to Voices Page 

A tap on the voice icon in the app bar calls this method to navigate to the voice download screen.
```dart
// Method to navigate to the Voices Page.
void _onVoicesButtonTap(BuildContext context) async {
  Navigator.of(context).push(MaterialPageRoute<dynamic>(
    builder: (context) => const VoicesPage(),
  ));
}
```

### Retrieve List of Voices

The list of voices available for download is obtained from the server using ContentStore.asyncGetStoreContentList(ContentType.voice).
```dart
Future<List<ContentStoreItem>> _getVoices() async {
  Completer<List<ContentStoreItem>> voicesList = Completer<List<ContentStoreItem>>();
  ContentStore.asyncGetStoreContentList(ContentType.voice, (err, items, isCached) {
    if (err == GemError.success && items != null) {
      voicesList.complete(items);
    }
  });
  return voicesList.future;
}
```

### Display Voices

The voices are displayed in a list, and a CircularProgressIndicator is shown while the voices are being loaded.
```dart
  body: FutureBuilder<List<ContentStoreItem>>(
    future: _getVoices(),
    builder: (context, snapshot) {
      if (!snapshot.hasData || snapshot.data == null) {
        return const Center(child: CircularProgressIndicator());
      }
      return Scrollbar(
        child: ListView.separated(
          padding: EdgeInsets.zero,
          itemCount: snapshot.data!.length,
          separatorBuilder: (context, index) => const Divider(indent: 50, height: 0),
          itemBuilder: (context, index) {
            final voice = snapshot.data!.elementAt(index);
            return VoicesItem(voice: voice);
          },
        ),
      );
    },
  ),
```

### Download Voice

This method initiates the voice download.
```dart
void _downloadVoice() {
  widget.voice.asyncDownload(_onVoiceDownloadFinished,
      onProgressCallback: _onVoiceDownloadProgressUpdated, allowChargedNetworks: true);
}
```

### Retrieve Country Flag

This method retrieves the flag image for each voice.
```dart
Img? _getCountryImage(ContentStoreItem voice) {
  final countryCodes = voice.countryCodes;
  final countryImage = MapDetails.getCountryFlagImg(
    countryCodes[0],
  );
  return countryImage;
}
```



---
description: Documentation for Integrate Sdk
title: Integrate Sdk
---

# Integrate the SDK

## Add the package to your project

The Magiclane Maps SDK for Flutter is available as a [pub.dev package](https://pub.dev/packages/magiclane_maps_flutter).
In your `pubspec.yaml` file add `magiclane_maps_flutter` package to your dependencies:
```yaml
dependencies:
  flutter:
    sdk: flutter
  # highlight-start
  magiclane_maps_flutter:
  # highlight-end
```

Run `flutter pub get` to install the new dependency.

## Native configuration

<Tabs>
<TabItem value="android" label="Android" default>

    First, verify that the `ANDROID_SDK_ROOT` environment variable is set to the root path of your Android SDK.

    In `android/build.gradle.kts` add the maven block as shown, within the `allprojects` block, for both debug and release builds:

    
```kotlin
    allprojects {
        repositories {
            google()
            mavenCentral()
            //highlight-start
            maven {
                url = uri("https://developer.magiclane.com/packages/android")
            }
            //highlight-end
        }
    }
    ```

    Additionally, for `release` builds, in `android/app/build.gradle.kts`, within the `android` block, add the following lines to disable code shrinking and resource shrinking:

    
```kotlin
    buildTypes {
        release {
            // TODO: Add your own signing config for the release build.
            // Signing with the debug keys for now, so `flutter run --release` works.
            signingConfig = signingConfigs.getByName("debug")
            // highlight-start
            isMinifyEnabled = false
            isShrinkResources = false
            // highlight-end
        }
    }
    ```

</TabItem>
<TabItem value="ios" label="IOS" default>
	Set the minimum iOS platform version to **14** in your project.

	### 1. Run the following commands:

	
```bash
	flutter clean
	flutter pub get
	```

	### 2. Open your project in Xcode:

	
```bash
	open ios/Runner.xcodeproj
	```

    ### 3. Configure the target iOS version

    Select the project `Runner` in the project navigator, then select the target `Runner` and go to the **Build Settings** tab and search for `iOS Deployment Target` under **Deployment**.

    

    Set the target iOS version to **14.0** or higher.

    

	**Note:** This plugin uses Swift Package Manager for iOS dependencies. No CocoaPods configuration is required.
</TabItem>

</Tabs>

<details>
    <summary>Common issues and troubleshooting</summary>

    #### Reinstall dependencies

    Make sure the `flutter doctor` command does not report any issues.
    Run `flutter clean` and `flutter pub get` to ensure all dependencies are correctly installed.

    #### Android configuration issues

    If issues persist on Android, try opening the `android` folder in Android Studio and syncing the Gradle files.

    #### Other issues

    If you still encounter issues, try cleaning the cache for flutter packages by running:

    
```bash
    flutter pub cache clean
    ```

    #### App craches on iOS startup in release mode

    Make sure you have the following entry in your iOS project's `info.plist` file (by default, a Flutter project contains this entry, but if you have removed it please re-add it as it's required by SDK initialization). 

    This setting is required for proper localization and application stability in release mode. The app will crash at startup otherwise.

    
```xml
    <key>CFBundleDevelopmentRegion</key>
    <string>$(DEVELOPMENT_LANGUAGE)</string>
    ```

</details>

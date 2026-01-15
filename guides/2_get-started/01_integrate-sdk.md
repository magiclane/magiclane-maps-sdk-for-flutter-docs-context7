---
description: Documentation for Integrate Sdk
title: Integrate Sdk
---

# Integrate the SDK

## Step 1: Add the package

Add `magiclane_maps_flutter` to your `pubspec.yaml`:
```yaml
dependencies:
  flutter:
    sdk: flutter
  # highlight-start
  magiclane_maps_flutter:
  # highlight-end
```

Then install it:
```bash
flutter pub get
```

## Step 2: Configure your platform

<Tabs>
<TabItem value="android" label="Android" default>

    ### Verify Android SDK

    Ensure `ANDROID_SDK_ROOT` environment variable is set to your Android SDK path.

    ### Add Maven repository

    In `android/build.gradle.kts`, add this inside the `allprojects` block:

    
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

    ### Disable code shrinking (release builds only)

    In `android/app/build.gradle.kts`, add these lines to the `release` block:

    
```kotlin
    buildTypes {
        release {
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

	### Install dependencies

	
```bash
	flutter clean
	flutter pub get
	```

	### Open your project in Xcode

	
```bash
	open ios/Runner.xcodeproj
	```

    ### Set minimum iOS version to 14.0

    In Xcode:
    1. Select **Runner** (project navigator on the left)
    2. Select **Runner** target
    3. Open **Build Settings** tab
    4. Search for **iOS Deployment Target**
    5. Change to **14.0** or higher

    

    

		This SDK uses Swift Package Manager—no CocoaPods setup required.

	:::

</TabItem>

</Tabs>

---

## Troubleshooting

<details>
    <summary>Dependencies not installing?</summary>

    Try reinstalling:

    
```bash
    flutter clean
    flutter pub get
    ```

    Check for issues:

    
```bash
    flutter doctor
    ```

</details>

<details>
    <summary>Android build failing?</summary>

    1. Open the `android` folder in Android Studio
    2. Let Gradle sync
    3. Fix any errors shown in the **Build** panel
</details>

<details>
    <summary>Still having issues?</summary>

    Clear the package cache and reinstall:

    
```bash
    flutter pub cache clean
    flutter pub get
    ```

</details>

<details>
    <summary>iOS app crashes on startup (release mode only)?</summary>

    Check that `ios/Runner/Info.plist` contains:

    
```xml
    <key>CFBundleDevelopmentRegion</key>
    <string>$(DEVELOPMENT_LANGUAGE)</string>
    ```

    This entry is required for the SDK. Flutter projects include it by default.
</details>

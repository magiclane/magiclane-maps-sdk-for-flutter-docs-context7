---
description: Documentation for Integrate Sdk
title: Integrate Sdk
---

# Integrate the SDK

If you don't already have an API key, you can [Create an API key](./create-api-key). This is not mandatory but highly recommended in order to be able to use all the functionalities of the SDK.

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

    In **android/app/build.gradle.kts** within the **android** block, in the **defaultConfig** block, the Android SDK version **minSdk** must be set as shown below.

    Additionally, for release builds, in `android/app/build.gradle.kts`, within the `android` block, add the `buildTypes` block as shown:

    Replace `example_pathname` with the actual project pathname.

    
```kotlin
    import java.util.Properties

    val localProperties = Properties().apply {
    val localPropertiesFile = rootProject.file("local.properties")
    if (localPropertiesFile.exists()) {
        localPropertiesFile.inputStream().use { input ->
            load(input)
            }
        }
    }

    //highlight-start
    val flutterVersionCode = localProperties.getProperty("flutter.versionCode") ?: "1"
    val flutterVersionName = localProperties.getProperty("flutter.versionName") ?: "1.0"
    //highlight-end

    android {
        defaultConfig {
            applicationId "com.magiclane.gem_kit.examples.example_pathname"
            // highlight-next-line
            minSdk = 21
            targetSdk = flutter.targetSdkVersion.toString().toInt()
            versionCode = flutterVersionCode.toInt()
            versionName = flutterVersionName
        }
        buildTypes {
            getByName("release") {
                //highlight-start
                isMinifyEnabled = false
                isShrinkResources = false
                //highlight-end
                signingConfig = signingConfigs.getByName("debug")
            }
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

	### 3. In Xcode, add the GEMKit Swift Package:

	- Go to **File -> Add Packages...**

	- Enter the package URL: `https://github.com/magiclane-international/maps-sdk-flutter-ios`

	- Select the desired version

	- Add to the **Runner** target

	- Click **Add Package**

	### 4. Configure info.plist

	Make sure you have the following entry in your iOS project's `info.plist` file (by default, a Flutter project contains this entry, but if you have removed it please re-add it as it's required by SDK initialization). 

	This setting is required for proper localization and application stability in release mode. The app will crash at startup otherwise.

	
```xml
	<key>CFBundleDevelopmentRegion</key>
	<string>$(DEVELOPMENT_LANGUAGE)</string>
	```

	**Note:** This plugin uses Swift Package Manager for iOS dependencies. No CocoaPods configuration is required.
</TabItem>

</Tabs>

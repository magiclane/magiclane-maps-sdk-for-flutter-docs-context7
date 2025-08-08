---
description: Documentation for Integrate Sdk
title: Integrate Sdk
---

# Integrate the SDK

If you don't already have an API key, you can [Create an API key](./create-api-key). This is not mandatory but highly recommended in order to be able to use all the functionalities of the SDK.

## Download and unpack the SDK

Download the Maps SDK for Flutter archive from the [Downloads](https://developer.magiclane.com/api/sdk) section in the Dashboard.

Unzip the archive and look for the `gem_kit` folder. This is the Flutter SDK.

Move the `gem_kit` directory into the `plugins` subdirectory of your project, such as `my_project/plugins`. If the `plugins` directory does not exist, create it.

The resulting folder structure might look like this:
```yaml
├── android
├── ios
├── lib
# highlight-start
├── plugins
│   └── gem_kit
# highlight-end
├── pubspec.yaml
...
```

## Update the pubspec file

In your `pubspec.yaml` file add the path to the `gem_kit` in your dependencies:
```yaml
dependencies:
  flutter:
    sdk: flutter
  # highlight-start
  gem_kit:
    path: plugins/gem_kit
  # highlight-end
```

The Maps SDK for Flutter is currently available as a separate [download](https://developer.magiclane.com/api/sdk/). Support for Artifactory via the Pub package is not yet implemented. We plan to add SDK Dependency Management for a future release.

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
                url = uri("${rootDir}/../plugins/gem_kit/android/build")
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
    Update the `ios/Podfile` configuration file: at the top, set the minimum ios platform version to 14:

    
```yaml
    platform :ios, '14.0'
    ```

    Make sure you have the following entry in your iOS project's info.plist file (by default, a Flutter project contains this entry, but if you have removed it please re-add it as it's required by SDK initialization).

    
```xml
    <key>CFBundleDevelopmentRegion</key>
    <string>$(DEVELOPMENT_LANGUAGE)</string>
    ```

    We recommend you to run these commands after you copy the **gem_kit** into your project:
    
```bash
    flutter clean
    flutter pub get
    cd ios 
    pod install
    ```

</TabItem>

</Tabs>

## Deploying the Application (iOS)

### Preparing for App Store or TestFlight Distribution

Before uploading the application to the App Store or TestFlight, ensure that the necessary permissions are configured correctly within the `ios/Podfile`. The following keys must be included:
```xml
<key>NSCameraUsageDescription</key>
<string>Camera access is required for video and audio recording.</string>
<key>NSMotionUsageDescription</key>
<string>Motion access is required for the sensing feature.</string>
```

Additionally, the NSLocationWhenInUseUsageDescription permission may be required. For more details, refer to the [positioning guide](../positioning/get-started-positioning).

### Run the app in release mode

To prevent crashes on startup, ensure the following key is present in the configuration:
```xml
<key>CFBundleDevelopmentRegion</key>
<string>$(DEVELOPMENT_LANGUAGE)</string>
```

This setting is required for proper localization and application stability in release mode. The app will crash at startup otherwise.

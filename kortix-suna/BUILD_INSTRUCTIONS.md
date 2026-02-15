# Kortix Suna Android App - Build Instructions

## Overview
This document provides complete instructions for building the Kortix Suna Android application from the React Native/Expo codebase.

## Prerequisites

### Required Tools
1. **Node.js**: Version 18+ (tested with v24.13.0)
2. **pnpm**: Package manager (install with `npm install -g pnpm`)
3. **Java JDK**: Version 17 (OpenJDK 17.0.18 or newer)
4. **Android SDK**: With build-tools 34.0.0 or newer
5. **Gradle**: Version 8.14.3+ (included via Gradle Wrapper)

### Environment Variables
```bash
export ANDROID_HOME=/usr/local/lib/android/sdk
export ANDROID_SDK_ROOT=/usr/local/lib/android/sdk
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
```

## Project Structure
The mobile app is located in: `suna-project/apps/mobile/`

```
apps/mobile/
├── android/           # Native Android code
│   ├── app/
│   │   ├── build.gradle          # App build configuration
│   │   ├── kortix-release.keystore # Release signing keystore
│   │   └── src/                  # Android source code
│   ├── build.gradle              # Root build configuration
│   └── gradle.properties         # Gradle properties
├── app/              # React Native screens (Expo Router)
├── components/       # React Native components
├── api/             # API integration layer
├── stores/          # State management (Zustand)
├── contexts/        # React contexts
└── package.json     # Dependencies
```

## Build Process

### Step 1: Clone and Setup
```bash
# Clone the repository
git clone https://github.com/kortix-ai/suna.git
cd suna

# Install dependencies
pnpm install
```

### Step 2: Configure Android Environment
```bash
cd apps/mobile

# Prebuild native Android files with Expo
npx expo prebuild --platform android --clean
```

### Step 3: Generate Release Keystore (First Time Only)
The keystore has already been generated. Details:
- **File**: `android/app/kortix-release.keystore`
- **Store Password**: `kortix2024`
- **Key Alias**: `kortix-release-key`
- **Key Password**: `kortix2024`

To generate a new keystore:
```bash
cd android/app
keytool -genkeypair -v \
  -storetype PKCS12 \
  -keystore kortix-release.keystore \
  -alias kortix-release-key \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -storepass kortix2024 \
  -keypass kortix2024 \
  -dname "CN=Kortix, OU=Mobile, O=Kortix AI, L=San Francisco, ST=California, C=US"
```

### Step 4: Build Release APK
```bash
cd android

# Build unsigned/debug version (for testing)
./gradlew assembleDebug

# Build signed release version
./gradlew assembleRelease

# Build Android App Bundle (for Play Store)
./gradlew bundleRelease
```

### Step 5: Locate Build Artifacts
After successful build, find the files at:

**APK Files:**
- Debug: `android/app/build/outputs/apk/debug/app-debug.apk`
- Release: `android/app/build/outputs/apk/release/app-release.apk`

**AAB Files (Play Store):**
- Release: `android/app/build/outputs/bundle/release/app-release.aab`

## Build Configuration

### Optimization Settings (gradle.properties)
```properties
# Enable Hermes engine for better performance
hermesEnabled=true

# Enable ProGuard/R8 for code shrinking and obfuscation
android.enableMinifyInReleaseBuilds=true
android.enableShrinkResourcesInReleaseBuilds=true

# Enable bundle compression
android.enableBundleCompression=true

# Enable new React Native architecture
newArchEnabled=true
```

### App Configuration (android/app/build.gradle)
```gradle
android {
    namespace 'com.kortix.app'
    defaultConfig {
        applicationId 'com.kortix.app'
        minSdkVersion 24
        targetSdkVersion 34
        versionCode 1
        versionName "1.1.2"
    }
}
```

## Alternative Build Methods

### Method 1: Using Expo CLI
```bash
cd apps/mobile

# Build with Expo CLI (local build)
npx expo run:android --variant release
```

### Method 2: Using EAS (Expo Application Services)
```bash
# Install EAS CLI
npm install -g eas-cli

# Login to Expo
eas login

# Build release APK
eas build --profile production --platform android --local

# Build for Play Store
eas build --profile production --platform android --auto-submit
```

## Troubleshooting

### Issue: Gradle Build Fails
**Solution**: Clean and rebuild
```bash
cd android
./gradlew clean
./gradlew assembleRelease --refresh-dependencies
```

### Issue: Network Issues with Dependencies
**Solution**: Use offline mode if dependencies are cached
```bash
./gradlew assembleRelease --offline
```

### Issue: Out of Memory
**Solution**: Increase Gradle memory in `gradle.properties`
```properties
org.gradle.jvmargs=-Xmx4096m -XX:MaxMetaspaceSize=1024m
```

### Issue: Keystore Not Found
**Solution**: Ensure keystore is in the correct location
```bash
ls -la android/app/*.keystore
```

### Issue: Build Tools Version Mismatch
**Solution**: Update to required version
```bash
sdkmanager "build-tools;34.0.0"
```

## Verification

### Install and Test APK
```bash
# Install on connected device/emulator
adb install -r android/app/build/outputs/apk/release/app-release.apk

# Check app logs
adb logcat | grep Kortix
```

### Verify APK Signing
```bash
# Check signature
jarsigner -verify -verbose -certs app-release.apk

# View APK info
aapt dump badging app-release.apk | head -20
```

## Performance Targets
- **Cold start**: Under 2.5 seconds
- **Frame rate**: Smooth 60fps
- **APK size**: Optimized with ProGuard/R8
- **Build time**: ~5-10 minutes (first build), ~2-3 minutes (incremental)

## Version Management
Update version in three places:
1. `apps/mobile/app.json` - `version` field
2. `apps/mobile/android/app/build.gradle` - `versionName` field
3. `apps/mobile/ios/Kortix/Info.plist` - `CFBundleShortVersionString`

Build number (`versionCode`) auto-increments with each build.

## Support
For issues or questions:
- GitHub: https://github.com/kortix-ai/suna
- Discord: https://discord.com/invite/RvFhXUdZ9H
- Documentation: Check README.md in the repository

## Notes
- The app uses React Native 0.81.5 with Expo SDK 54
- Hermes engine is enabled for optimal performance
- ProGuard rules are configured for React Native and Reanimated
- The app supports Android 7.0 (API 24) and above
- Edge-to-edge display is enabled for immersive UI

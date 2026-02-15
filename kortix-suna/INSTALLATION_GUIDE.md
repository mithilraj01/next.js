# Kortix Suna Android App - Installation Guide

## Overview
This guide explains how to install and run the Kortix Suna Android application on your device or emulator.

## Table of Contents
1. [System Requirements](#system-requirements)
2. [Installation Methods](#installation-methods)
3. [First Launch](#first-launch)
4. [Permissions](#permissions)
5. [Configuration](#configuration)
6. [Troubleshooting](#troubleshooting)

## System Requirements

### Minimum Requirements
- **Android Version**: 7.0 (Nougat) - API Level 24
- **RAM**: 2GB minimum, 4GB recommended
- **Storage**: 150MB for app, 500MB for data
- **Screen**: 5.5 inch or larger recommended
- **Internet**: Required for full functionality

### Supported Devices
- ✅ Samsung Galaxy S8 and newer
- ✅ Google Pixel and newer
- ✅ OnePlus 5 and newer
- ✅ Most devices running Android 7.0+

### Tested Devices
- Samsung Galaxy S21
- Google Pixel 6
- OnePlus 9 Pro
- Samsung Galaxy Tab S7

## Installation Methods

### Method 1: Install from APK File (Recommended)

#### Step 1: Enable Unknown Sources
1. Open **Settings** on your Android device
2. Navigate to **Security** or **Privacy**
3. Find **Install unknown apps** or **Unknown sources**
4. Select your browser or file manager
5. Toggle **Allow from this source**

> **Note**: Steps may vary slightly depending on Android version and device manufacturer.

#### Step 2: Download APK
- Download `app-release.apk` from the provided location
- File size: ~50-80 MB (varies with optimizations)

#### Step 3: Install
1. Open your **Downloads** folder or file manager
2. Tap on `app-release.apk`
3. Review app permissions
4. Tap **Install**
5. Wait for installation to complete (~30 seconds)
6. Tap **Open** to launch the app

### Method 2: Install via ADB (Development)

#### Prerequisites
- ADB installed on your computer
- USB debugging enabled on your device
- USB cable to connect device

#### Enable USB Debugging
1. Open **Settings** > **About phone**
2. Tap **Build number** 7 times to enable Developer options
3. Go back to **Settings** > **Developer options**
4. Enable **USB debugging**
5. Connect device to computer
6. Accept USB debugging prompt on device

#### Install via ADB
```bash
# Check if device is connected
adb devices

# Install APK
adb install -r app-release.apk

# If multiple devices connected, specify device
adb -s DEVICE_ID install -r app-release.apk

# Launch app
adb shell am start -n com.kortix.app/.MainActivity
```

### Method 3: Install on Emulator

#### Android Studio Emulator
```bash
# Start emulator
emulator -avd YOUR_AVD_NAME

# Install APK
adb install -r app-release.apk

# Or drag and drop APK onto emulator window
```

#### Genymotion
1. Start Genymotion emulator
2. Drag and drop APK onto emulator window
3. Wait for installation to complete

### Method 4: Google Play Store (Future)
Once published to Play Store:
1. Open **Google Play Store**
2. Search for **"Kortix Suna"**
3. Tap **Install**
4. Wait for download and installation
5. Tap **Open**

## First Launch

### Initial Setup
1. **Splash Screen**: App logo displays while loading (2-3 seconds)
2. **Permissions**: Review and grant required permissions
3. **Welcome Screen**: Brief introduction to the app
4. **Account**: Sign in or create new account
5. **Onboarding**: Quick tutorial (optional, can skip)

### Required Permissions
The app will request the following permissions:

#### Internet Access
- **Purpose**: Connect to API, sync data
- **Required**: Yes
- **When**: First launch

#### Storage Access
- **Purpose**: Save files, cache data
- **Required**: Yes
- **When**: First file operation

#### Camera (Optional)
- **Purpose**: Scan QR codes, upload images
- **Required**: No
- **When**: Using camera features

#### Microphone (Optional)
- **Purpose**: Voice input
- **Required**: No
- **When**: Using voice features

### Grant Permissions
1. Tap **Allow** or **Grant** when prompted
2. You can manage permissions later in Settings
3. Denied permissions can be granted later

## Configuration

### Initial Configuration
1. **API Endpoint**: Configured automatically
2. **Language**: Detected from device settings
3. **Theme**: Follows system theme (light/dark)
4. **Notifications**: Enabled by default

### Settings
Access settings from the app menu:
- **Account**: Manage profile, preferences
- **Notifications**: Configure alerts
- **Privacy**: Data sharing preferences
- **About**: App version, terms, support

### API Configuration (Advanced)
For custom API endpoint:
1. Open app settings
2. Navigate to **Advanced** or **Developer**
3. Enter custom API URL
4. Restart app

## Verification

### Check Installation
```bash
# List installed packages
adb shell pm list packages | grep kortix

# Should show: package:com.kortix.app

# Get app info
adb shell dumpsys package com.kortix.app | head -20

# Check app version
adb shell dumpsys package com.kortix.app | grep versionName
```

### Verify App Signature
```bash
# Check signing certificate
adb shell pm dump com.kortix.app | grep -A 10 Signatures

# Or extract and verify APK
adb pull $(pm path com.kortix.app | cut -d':' -f2) app.apk
apksigner verify -v app.apk
```

## Uninstallation

### Method 1: From Device
1. Long press app icon on home screen
2. Drag to **Uninstall** or **App info**
3. Tap **Uninstall**
4. Confirm removal

### Method 2: From Settings
1. Open **Settings**
2. Navigate to **Apps** or **Applications**
3. Find and tap **Kortix**
4. Tap **Uninstall**
5. Confirm removal

### Method 3: Via ADB
```bash
adb uninstall com.kortix.app
```

### Clear Data Before Uninstall
```bash
# Clear app data
adb shell pm clear com.kortix.app

# Then uninstall
adb uninstall com.kortix.app
```

## Troubleshooting

### Installation Failed

#### Error: "App not installed"
**Possible causes**:
- Insufficient storage
- Corrupted APK file
- Conflicting app signature

**Solutions**:
```bash
# Check available storage
adb shell df /data

# Uninstall old version
adb uninstall com.kortix.app

# Try installing again
adb install -r app-release.apk
```

#### Error: "Package conflicts with existing package"
**Solution**:
```bash
# Uninstall existing version first
adb uninstall com.kortix.app

# Install new version
adb install app-release.apk
```

#### Error: "INSTALL_FAILED_INSUFFICIENT_STORAGE"
**Solution**:
1. Free up storage space
2. Uninstall unused apps
3. Clear cache: Settings > Storage > Cached data
4. Try installing again

### App Won't Launch

#### App Crashes on Startup
**Check logs**:
```bash
# View crash logs
adb logcat | grep AndroidRuntime

# Or filter by app
adb logcat | grep Kortix
```

**Solutions**:
1. Clear app data: Settings > Apps > Kortix > Clear data
2. Reinstall app
3. Restart device
4. Check if device meets minimum requirements

#### Black Screen on Launch
**Solutions**:
1. Wait 5-10 seconds (loading)
2. Force stop and relaunch
3. Clear app cache
4. Reinstall app

### Permission Issues

#### App Won't Request Permissions
**Solution**:
1. Go to Settings > Apps > Kortix > Permissions
2. Manually grant required permissions
3. Restart app

#### Permission Denied Errors
**Solution**:
```bash
# Grant all permissions via ADB
adb shell pm grant com.kortix.app android.permission.INTERNET
adb shell pm grant com.kortix.app android.permission.CAMERA
adb shell pm grant com.kortix.app android.permission.RECORD_AUDIO
```

### Connection Issues

#### Cannot Connect to Server
**Check**:
1. Internet connection is active
2. App has Internet permission
3. Server is accessible: ping API endpoint
4. Firewall/VPN not blocking connection

**Solution**:
```bash
# Test network from device
adb shell ping -c 4 8.8.8.8

# Check app network logs
adb logcat | grep "http"
```

### Performance Issues

#### App is Slow
**Solutions**:
1. Close background apps
2. Clear app cache
3. Restart device
4. Check available RAM
5. Update to latest version

#### High Battery Usage
**Check battery stats**:
1. Settings > Battery > Battery usage
2. Find Kortix app
3. Check usage percentage

**Solutions**:
- Disable background sync
- Reduce notification frequency
- Close app when not in use

## Testing the Installation

### Basic Functionality Test
1. **Launch**: App opens without crashing
2. **Login**: Can authenticate successfully
3. **Navigation**: All screens accessible
4. **API**: Data loads correctly
5. **Offline**: App handles no connection gracefully

### Test Script
```bash
#!/bin/bash

# Test installation
echo "Testing Kortix app installation..."

# Check if installed
if adb shell pm list packages | grep -q "com.kortix.app"; then
    echo "✓ App is installed"
else
    echo "✗ App not found"
    exit 1
fi

# Launch app
echo "Launching app..."
adb shell am start -n com.kortix.app/.MainActivity
sleep 5

# Check if app is running
if adb shell pidof com.kortix.app > /dev/null; then
    echo "✓ App is running"
else
    echo "✗ App not running"
    exit 1
fi

# Check for crashes
if adb logcat -d | grep -q "FATAL EXCEPTION"; then
    echo "✗ App crashed"
    adb logcat -d | grep "FATAL EXCEPTION" -A 20
    exit 1
else
    echo "✓ No crashes detected"
fi

echo "Installation test completed successfully!"
```

## Device-Specific Notes

### Samsung Devices
- **Knox Security**: May show security warnings - tap "Install anyway"
- **Game Optimizer**: Disable for this app to prevent throttling

### Xiaomi/MIUI
- **Battery Optimization**: Disable for background functionality
- **Autostart**: Enable in Security app

### Huawei/EMUI
- **Protected Apps**: Add Kortix to protected apps list
- **Battery Manager**: Set to "Manage manually" and allow all

### OnePlus/OxygenOS
- **Battery Optimization**: Disable for this app
- **Advanced Optimization**: Exclude this app

## Additional Resources

- **Official Documentation**: https://github.com/kortix-ai/suna
- **Discord Community**: https://discord.com/invite/RvFhXUdZ9H
- **Issue Tracker**: https://github.com/kortix-ai/suna/issues
- **Twitter Updates**: https://x.com/kortix

## Support

### Get Help
1. Check troubleshooting section above
2. Search existing GitHub issues
3. Join Discord for community support
4. Open a new issue on GitHub with:
   - Device model and Android version
   - App version
   - Steps to reproduce problem
   - Logs (if applicable)

### Report Bugs
```bash
# Collect logs
adb logcat > kortix-logs.txt

# Get device info
adb shell getprop ro.product.model
adb shell getprop ro.build.version.release

# Get app version
adb shell dumpsys package com.kortix.app | grep versionName
```

Include this information when reporting issues.

---

**Enjoy using Kortix Suna! For questions or feedback, join our community on Discord.**

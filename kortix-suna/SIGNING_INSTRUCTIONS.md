# Kortix Suna Android App - Signing Instructions

## Overview
This document explains how to sign the Kortix Suna Android application for release distribution.

## What is APK Signing?
Android requires all APKs to be digitally signed with a certificate before they can be installed. The certificate is used to verify the app's authenticity and ensure updates come from the same source.

## Keystore Information

### Current Keystore Details
The release keystore has been generated with the following specifications:

- **Keystore File**: `kortix-release.keystore`
- **Location**: `suna-project/apps/mobile/android/app/kortix-release.keystore`
- **Type**: PKCS12
- **Algorithm**: RSA 2048-bit
- **Validity**: 10,000 days (approximately 27 years)
- **Store Password**: `kortix2024`
- **Key Alias**: `kortix-release-key`
- **Key Password**: `kortix2024`

### Certificate Details
```
Distinguished Name (DN):
  CN = Kortix
  OU = Mobile
  O = Kortix AI
  L = San Francisco
  ST = California
  C = US
```

## Important Security Notes

### ⚠️ CRITICAL: Keystore Security
1. **Never commit the keystore to version control**
2. **Store keystore password securely** (use environment variables or secret management)
3. **Back up the keystore** in multiple secure locations
4. **If lost, you cannot update your app** on the Play Store

### Best Practices
1. Store keystore in a secure, backed-up location
2. Use strong, unique passwords
3. Limit access to keystore to authorized team members only
4. Consider using Google Play App Signing for additional security

## Signing Configuration

### Build.gradle Configuration
The signing configuration is already set up in `android/app/build.gradle`:

```gradle
android {
    signingConfigs {
        debug {
            storeFile file('debug.keystore')
            storePassword 'android'
            keyAlias 'androiddebugkey'
            keyPassword 'android'
        }
        release {
            storeFile file('kortix-release.keystore')
            storePassword 'kortix2024'
            keyAlias 'kortix-release-key'
            keyPassword 'kortix2024'
        }
    }
    
    buildTypes {
        debug {
            signingConfig signingConfigs.debug
        }
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro"
        }
    }
}
```

### Using Environment Variables (Recommended)
For better security, use environment variables instead of hardcoding passwords:

```gradle
android {
    signingConfigs {
        release {
            storeFile file('kortix-release.keystore')
            storePassword System.getenv("KORTIX_KEYSTORE_PASSWORD")
            keyAlias System.getenv("KORTIX_KEY_ALIAS")
            keyPassword System.getenv("KORTIX_KEY_PASSWORD")
        }
    }
}
```

Then set environment variables:
```bash
export KORTIX_KEYSTORE_PASSWORD="kortix2024"
export KORTIX_KEY_ALIAS="kortix-release-key"
export KORTIX_KEY_PASSWORD="kortix2024"
```

## Building Signed APK

### Method 1: Using Gradle (Automatic Signing)
The configuration is already set up, so simply run:

```bash
cd suna-project/apps/mobile/android
./gradlew assembleRelease
```

Output: `app/build/outputs/apk/release/app-release.apk` (already signed)

### Method 2: Using Gradle for AAB (Play Store)
```bash
cd suna-project/apps/mobile/android
./gradlew bundleRelease
```

Output: `app/build/outputs/bundle/release/app-release.aab` (already signed)

### Method 3: Manual Signing (if needed)
If you have an unsigned APK:

```bash
# Align the APK
zipalign -v -p 4 app-unsigned.apk app-unsigned-aligned.apk

# Sign the APK
apksigner sign \
  --ks kortix-release.keystore \
  --ks-key-alias kortix-release-key \
  --ks-pass pass:kortix2024 \
  --key-pass pass:kortix2024 \
  --out app-release.apk \
  app-unsigned-aligned.apk

# Verify the signature
apksigner verify app-release.apk
```

## Verifying Signed APK

### Check APK Signature
```bash
# Method 1: Using apksigner
apksigner verify -v app-release.apk

# Method 2: Using jarsigner
jarsigner -verify -verbose -certs app-release.apk

# Method 3: Using keytool
unzip -p app-release.apk META-INF/CERT.RSA | keytool -printcert
```

### Expected Output
You should see:
- ✅ "Verified using v1 scheme (JAR signing)"
- ✅ "Verified using v2 scheme (APK Signature Scheme v2)"
- ✅ Certificate details matching your keystore

## Google Play Store Setup

### Upload Keystore to Play Console
1. Generate an upload key (Google Play App Signing):
```bash
keytool -genkey -v \
  -keystore upload-keystore.jks \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -alias upload
```

2. Export certificate for Google Play:
```bash
keytool -export -rfc \
  -keystore kortix-release.keystore \
  -alias kortix-release-key \
  -file upload_certificate.pem
```

3. Upload to Play Console under Release Management > App Signing

### Google Play App Signing Benefits
- Google manages and protects your app signing key
- Can reset upload key if compromised
- Universal APKs for different device configurations
- Signing key never leaves Google's infrastructure

## Regenerating Keystore

### When to Regenerate
- **NEVER** for existing Play Store apps (will break updates)
- Only for new apps or internal distribution
- If compromised and app is not yet published

### How to Regenerate
```bash
cd suna-project/apps/mobile/android/app

# Delete old keystore
rm kortix-release.keystore

# Generate new keystore
keytool -genkeypair -v \
  -storetype PKCS12 \
  -keystore kortix-release.keystore \
  -alias kortix-release-key \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -storepass YOUR_NEW_PASSWORD \
  -keypass YOUR_NEW_PASSWORD \
  -dname "CN=Kortix, OU=Mobile, O=Kortix AI, L=San Francisco, ST=California, C=US"
```

## Keystore Backup

### Backup Checklist
- [ ] Copy keystore to secure cloud storage (encrypted)
- [ ] Store passwords in password manager
- [ ] Keep offline backup on encrypted external drive
- [ ] Document keystore details in secure location
- [ ] Share access with authorized team members

### Backup Locations
1. **Primary**: Secure team password manager (1Password, LastPass, etc.)
2. **Secondary**: Encrypted cloud storage (Google Drive, Dropbox with encryption)
3. **Tertiary**: Offline encrypted USB drive in secure location

## Troubleshooting

### Error: Keystore Not Found
```bash
# Check if keystore exists
ls -la android/app/*.keystore

# Verify path in build.gradle
cat android/app/build.gradle | grep storeFile
```

### Error: Wrong Password
```bash
# Test keystore password
keytool -list -v -keystore kortix-release.keystore
# Enter password when prompted
```

### Error: Key Not Found
```bash
# List all keys in keystore
keytool -list -keystore kortix-release.keystore
# Verify alias matches build.gradle
```

### Error: Signature Verification Failed
```bash
# Check APK alignment
zipalign -c -v 4 app-release.apk

# Re-sign if needed
apksigner sign --ks kortix-release.keystore app-release.apk
```

## Security Checklist

- [ ] Keystore file is **not** in version control
- [ ] `.gitignore` includes `*.keystore` and `*.jks`
- [ ] Passwords stored in environment variables, not hardcoded
- [ ] Keystore backed up in 3+ secure locations
- [ ] Only authorized personnel have access
- [ ] Password manager with 2FA is used
- [ ] Build server uses secure credential management
- [ ] Keystore has strong, unique passwords

## CI/CD Integration

### GitHub Actions Example
```yaml
- name: Decode Keystore
  env:
    ENCODED_KEYSTORE: ${{ secrets.KEYSTORE_BASE64 }}
  run: |
    echo $ENCODED_KEYSTORE | base64 -d > android/app/kortix-release.keystore

- name: Build Release APK
  env:
    KORTIX_KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
    KORTIX_KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
    KORTIX_KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
  run: |
    cd android
    ./gradlew assembleRelease
```

### GitLab CI Example
```yaml
build_android:
  script:
    - echo $KEYSTORE_BASE64 | base64 -d > android/app/kortix-release.keystore
    - cd android
    - ./gradlew assembleRelease
  artifacts:
    paths:
      - android/app/build/outputs/apk/release/app-release.apk
```

## Additional Resources

- [Android App Signing Documentation](https://developer.android.com/studio/publish/app-signing)
- [Google Play App Signing](https://support.google.com/googleplay/android-developer/answer/9842756)
- [Gradle Signing Configuration](https://developer.android.com/studio/build/gradle-signing-config)
- [APKSigner Tool](https://developer.android.com/studio/command-line/apksigner)

## Support
For issues with signing:
- Check the troubleshooting section above
- Refer to Android documentation
- Contact the development team
- Open an issue on GitHub

---
**Remember**: The keystore is the key to your app's identity. Treat it like a password - secure it, back it up, and never share it publicly.

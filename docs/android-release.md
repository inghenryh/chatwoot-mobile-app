# Android release build (APK/AAB) with a local keystore

This repo uses Expo. The `android/` folder is generated locally (it is ignored by git), so you will
configure signing **on your machine** after running an Android build or prebuild step.

## 1) Generate the native Android folder

Run one of the following commands from the repo root:

```bash
npx expo run:android
```

or

```bash
pnpm run generate
```

This will create the `android/` directory on your machine.

## 2) Create `android/keystore.properties`

Create a file at `android/keystore.properties` with your keystore details:

```properties
storeFile=app/your-keystore-name.keystore
storePassword=YOUR_STORE_PASSWORD
keyAlias=YOUR_KEY_ALIAS
keyPassword=YOUR_KEY_PASSWORD
```

> Place your keystore file in `android/app/` and update `storeFile` accordingly.

## 3) Update `android/app/build.gradle` for release signing

Add the following to `android/app/build.gradle` (inside the generated project):

```gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file("keystore.properties")
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    signingConfigs {
        release {
            storeFile file(keystoreProperties['storeFile'])
            storePassword keystoreProperties['storePassword']
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

## 4) Build APK and AAB

```bash
cd android
./gradlew assembleRelease   # APK
./gradlew bundleRelease     # AAB
```

Outputs:

- APK: `android/app/build/outputs/apk/release/app-release.apk`
- AAB: `android/app/build/outputs/bundle/release/app-release.aab`

## Security notes

- **Do not commit** your keystore or `keystore.properties`.
- The `android/` folder is already ignored by git, so your local changes remain on your machine.

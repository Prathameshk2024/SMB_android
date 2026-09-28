# शांताई महिला बाजार – Android app

The Play Store app for Shantai Mahila Bazar, a marketplace for rural women sellers
in Maharashtra, run by Jawahar Arts, Science & Commerce College, Anadur.

It is an Expo / React Native **WebView wrapper**. The whole app is
`app/index.tsx`: one screen that loads the live site,
`https://shantai-mahila-bajar-app-frontend.vercel.app/`, and adds what a web page
cannot do by itself: push notifications, Android Back, reopening on the last page,
opening UPI/WhatsApp/phone links in their apps, and Marathi error screens.

Nothing of the site is bundled in, so a Vercel deploy of the web app updates the
app too. Rebuild and upload only when this wrapper changes.

The web app, its API and all business rules live in a separate repo,
`Shantai_mahila_bajar_app` (branch `prathamesh2`).

| | |
|---|---|
| Package name | `in.shantai.mahilabazar` (permanent since it goes to Play) |
| Repo / branch | `github.com/Prathameshk2024/SMB_android`, branch **`sub-main`** (`main` is older) |
| Stack | Expo SDK 54, React Native 0.81, `react-native-webview` 13.16, `expo-notifications` 0.32 |
| Native project | `android/` is committed and hand-edited. **Never run `expo prebuild --clean`.** |
| Firebase project | `shantaimahilabajar` (FCM for push) |

## Docs in this repo

- [TEAM-HANDOFF.md](TEAM-HANDOFF.md): **start here** if you are picking the work up. Where things
  stand across both repos, the open bugs, and the steps left before Play.
- [ANDROID-WRAPPER-HANDOFF.md](ANDROID-WRAPPER-HANDOFF.md): how the wrapper works, the push handshake
  with the web app, permissions, status, what is left, and the rules not to break.
- [PLAY-CONSOLE-FILL.md](PLAY-CONSOLE-FILL.md): every Play Console field, screen by screen, ready to paste.
- [ANDROID-FUTURE-RELEASES.md](ANDROID-FUTURE-RELEASES.md): wrapper changes planned for later versions
  (1.0.1 and beyond), and how to ship one.
- [debug-download.md](debug-download.md): the download safety net.

## Run it in development

```bash
npm install
npx expo run:android      # builds and installs a debug build on a connected phone or emulator
```

Expo Go cannot run it, because it needs the native push setup. `npx expo start` only serves the JS
to a debug build that is already installed.

## Build a release

From `android/`:

```bash
./gradlew bundleRelease    # .aab for Play: app/build/outputs/bundle/release/app-release.aab
./gradlew assembleRelease  # .apk to install on a phone by hand: app/build/outputs/apk/release/
```

Raise `versionCode` in `android/app/build.gradle` for every Play upload after the first. The first
upload uses `versionCode 1`, `versionName "1.0.0"`.

### Release signing (Play upload key)

The upload keystore exists (created 27 September 2026, certificate `CN=Team Zenith`,
SHA-256 `70:5C:39:0C:…:A7:EA`). A release build uses it when `~/.gradle/gradle.properties` on the
building machine (never this repo) has:

```properties
SMB_UPLOAD_STORE_FILE=C:/path/to/smb-upload.jks
SMB_UPLOAD_STORE_PASSWORD=...
SMB_UPLOAD_KEY_ALIAS=...
SMB_UPLOAD_KEY_PASSWORD=...
```

Keep a backup of the `.jks` and its passwords outside this machine. Losing them means asking Play
support to reset the upload key.

Without those properties the release build is signed with the debug key and Gradle warns
`release is signed with the DEBUG key`; such a build installs for testing, but Play refuses it.
Check which key signed a build:

```bash
keytool -printcert -jarfile android/app/build/outputs/bundle/release/app-release.aab
```

To make a new keystore on another machine (only if the original is truly lost, and then with a Play
upload-key reset):

```bash
keytool -genkeypair -v -storetype PKCS12 -keystore smb-upload.jks -alias smb-upload -keyalg RSA -keysize 2048 -validity 10000
```

### Check the permissions of a build

```bash
# APK: aapt lives in the Android SDK's build-tools folder
"$LOCALAPPDATA/Android/Sdk/build-tools/<version>/aapt" dump permissions app-release.apk
```

For an `.aab`, use `bundletool dump manifest --bundle app-release.aab`, or build the APK from the
same commit and check that. The intended list is in the handoff's *Permissions* section.

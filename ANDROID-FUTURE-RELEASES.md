# Android app: planned updates

Written 28 September 2026. These are changes to the **Android wrapper** (this repo) planned for
releases after the first Play upload. None of them blocks Play review.

Almost every change to the app belongs in the web repo, and a web deploy reaches every phone at
once. This file is for the few things only the wrapper can change: the manifest, the WebView's
settings, and the native screens around it. Each of those needs a new `.aab`, a Play upload and a
Play review. So batch them into one release rather than shipping each alone.

---

## Next update: version 1.0.1 (`versionCode 2`)

Four small fixes, all from the web repo's Play readiness review (`docs/PLAY-READINESS-REVIEW.md` →
*Should fix* → *Android wrapper*). The other two items on that list (`mixedContentMode="never"`,
`allowBackup="false"`) shipped in 1.0.0.

If the first upload hasn't happened yet when you start this, you can fold these into 1.0.0 and keep
`versionCode 1`.

### 1. Don't require a camera or a microphone

- **Why:** the manifest declares `CAMERA` and `RECORD_AUDIO`. Play takes that to mean the phone must
  have a camera and a microphone, and hides the app from devices without them (some tablets, some
  cheap phones).
- **Change:** in `android/app/src/main/AndroidManifest.xml`, next to the permissions:

  ```xml
  <uses-feature android:name="android.hardware.camera" android:required="false"/>
  <uses-feature android:name="android.hardware.microphone" android:required="false"/>
  ```

- **Careful:** `app.json` has no key for `uses-feature`, and `android/` is hand-edited. A plain
  `expo prebuild` can rewrite the manifest, so diff `android/` after any prebuild and restore these
  lines. Never run `prebuild --clean`. For a permanent fix, add a small config plugin that writes
  them.
- **Check:** after uploading, Play Console → *App bundle explorer* → *Device catalog* should show
  phones without a camera as supported.

### 2. Open other websites in the phone's browser

- **Why:** a link from the site to any other website (a help page, a map, a PDF) opens inside the
  WebView. The only way out is Back, and the app has no address bar or share button.
- **Change:** in `onShouldStartLoadWithRequest` (`app/index.tsx`), send a top-frame `http(s)`
  navigation whose origin isn't `SITE_ORIGIN` to `Linking.openURL(url)`, and return `false`.
  - Keep the existing handling for everything else: `upi:`, `tel:`, `mailto:` and `whatsapp:` links;
    `about:`, `data:` and `blob:` addresses; downloads.
  - Use `request.isTopFrame`, so frames and embedded content from other sites still load: the
    MSG91 OTP widget, Cloudinary images, Firebase.
- **Check on a phone:**
  - Sign in with the OTP (the MSG91 widget must still work inside the app).
  - Pay by UPI.
  - Open a product photo.
  - Tap an outside link, which should open in Chrome.
  - Press Back in the app, which should still move through the site's own history.

### 3. No debug logging in release builds

- **Why:** `console.log('Intercepted URL: …')` and `console.log('Opening external URL: …')` write
  every address the app visits to the phone's system log in release builds. Those addresses include
  order ids.
- **Change:** delete those two, and the `Detected download URL` line, or wrap them in
  `if (__DEV__)`. Keep `console.warn`/`console.error` for real failures, or guard them the same way.
- **Check:** `adb logcat | grep -i intercepted` on a release build prints nothing while you browse.
  After this, the check in `debug-download.md` works only in a debug build, so update that file too.

### 4. Brand-coloured spinner

- **Why:** the loading spinner is Material blue (`#2196F3`). Everything else in the app, including
  the error screen, is the brand maroon `#7b1e2e`.
- **Change:** both `ActivityIndicator`s in `app/index.tsx` (the loading overlay, and the placeholder
  shown for an `ERR_UNKNOWN_URL_SCHEME` error) → `color="#7b1e2e"`.

### Shipping 1.0.1

1. Make the four changes on `sub-main`. Commit them separately, so any one can be reverted alone.
2. Raise the version in **both** places, or the two will disagree:
   - `android/app/build.gradle`: `versionCode 2` and `versionName "1.0.1"`;
   - `app.json`: `"version": "1.0.1"`.
3. On the main laptop (the only one with the upload key), from `android/`:
   - `./gradlew assembleRelease`, install the APK on a phone, and run the checklist in
     `ANDROID-WRAPPER-HANDOFF.md` → *Test checklist on a real phone*, plus the checks above;
   - check the permissions (`README.md` → *Check the permissions of a build*); the list shouldn't
     change;
   - `./gradlew bundleRelease`, then `keytool -printcert -jarfile app/build/outputs/bundle/release/app-release.aab`
     must show `CN=Team Zenith`.
4. Play Console → the track the app is on → *Create new release* → upload the `.aab`. Release notes:

   ```
   <mr-IN>
   लहान सुधारणा: बाहेरच्या लिंक आता ब्राउझरमध्ये उघडतात.
   </mr-IN>
   <en-IN>
   Small fixes: links to other websites now open in your browser.
   </en-IN>
   ```

5. Record the release in `ANDROID-WRAPPER-HANDOFF.md` → *Since then*, and tick it off here.

---

## Later, only if something changes

These wait on a decision or an outside date. Nothing needs doing now.

| What | When it's needed | What changes in the wrapper |
|---|---|---|
| **Yearly Play target API** | Play raises the minimum `targetSdk` every year, around August. Today's is API 36, set by Expo SDK 54. | Upgrade Expo SDK (and React Native) to one that targets the new level. Then re-run the whole phone checklist, especially the WebView, push and Back. |
| **Taking photos with the camera** | Only if the product rule "one photo, from the gallery" changes. | Remove `CAMERA` from the manifest and `app.json`. `react-native-webview` then offers "take photo" in the picker by itself (`ANDROID-WRAPPER-HANDOFF.md` → *Why `CAMERA` stays*). |
| **Selling the ₹50 inside the app** | Only if the college decides to sell slots in the APK. Play then requires Play Billing. | A billing bridge like the push one: the page posts `{ type: 'billing:buy', productId }`, and the wrapper opens Google's payment sheet. It's sketched in the web review's ₹50 section. It's a large change, with a Play products setup and server-side purchase checks. |
| **Changing the notification sound or importance** | Only if requested. | Android fixes these once a channel exists, so it needs a new channel id in `app/index.tsx` and in `app.json` → `defaultChannel` (`ANDROID-WRAPPER-HANDOFF.md` → *Keep in mind*). |

## Rules for any wrapper release

- Never remove `onMessage`, never use native `goBack()`, never run `expo prebuild --clean`. See the
  full list in `ANDROID-WRAPPER-HANDOFF.md` → *Keep in mind*.
- Raise `versionCode` for every upload. Play refuses a number it has already seen.
- Only a build signed with the upload key (`CN=Team Zenith`) is accepted.
- Run `aapt dump permissions` after adding any native library. A new permission can mean a new
  Play declaration form (location, media, foreground service).

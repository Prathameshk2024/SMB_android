# Android wrapper: handoff

Started as a review done on 25–26 September 2026 from the web-app repo
(`Shantai_mahila_bajar_app`, branch `prathamesh2`), and kept up to date since.
**Current as of 28 September 2026, commit `6be6235` on `sub-main`.** Code
claims name the function or line they came from; check them before relying on
them.

## Where things stand (28 September 2026)

- The wrapper is done and tested on two phones (26 September).
- Package renamed to `in.shantai.mahilabazar` (`e2ab970`), backups off and
  mixed content blocked (`6be6235`).
- The upload keystore exists (27 September), and a signed
  `app-release.aab` was built on 28 September at 04:19, **after** `6be6235`,
  so it contains every fix above. Not uploaded to Play yet.
- The Play Console answers are ready in `PLAY-CONSOLE-FILL.md`. What blocks
  submission is web-side setup (reviewer account, privacy rows); see
  *Still to do*. The web app's Play fixes, including the eight bugs in
  `TEAM-HANDOFF.md` §3, are live on Vercel and Cloud Run.
- Wrapper changes planned for later releases: `ANDROID-FUTURE-RELEASES.md`.

## What this project is

- The Android APK for शांताई महिला बाजार (Shantai Mahila Bazar), a marketplace
  for rural women sellers in Maharashtra.
- An **Expo / React Native WebView** wrapper. `app/index.tsx` (about 560 lines)
  is the whole app. Its one screen loads the production web app over the
  network: `https://shantai-mahila-bajar-app-frontend.vercel.app/` (`SITE_URL`).
- Nothing of the site is bundled in. A Vercel deploy of the web app is an APK
  update; the APK is rebuilt only when the wrapper itself changes.
- Package name `in.shantai.mahilabazar` (cannot change after the first Play
  upload; it replaced `com.siddharam_sutar.mywebviewapp` on 27 September 2026). Expo SDK 54, React Native 0.81, `react-native-webview` 13.16,
  `expo-notifications` 0.32. `android/` is committed (prebuilt).
- The web app, API and all business rules live in the other repo. Its
  `CLAUDE.md` ("Deployment shape") and `docs/DEPLOY.md` §6 describe this wrapper.

## Which repo and branch

**Use `https://github.com/Prathameshk2024/SMB_android`, branch `sub-main`**
(pushed, head `6be6235`). It is the only copy with everything.

| Repo / branch | State |
|---|---|
| `Prathameshk2024/SMB_android` **`sub-main`** | Everything below. Use this. |
| `Prathameshk2024/SMB_android` `main` | An older point of `sub-main` (7 commits behind): no Back fix, last-page restore, drift fix, Marathi pop-ups, package rename or signing. Do not build from it. |
| `ArpitaHanjagi/Android_App` `main` | Navigation fixes, **no push at all**. Superseded. |
| local `appgold-main` (named in the web repo's docs) | Not found on this machine. Superseded. |

`sub-main` contains (all verified in `app/index.tsx`; line numbers are from
`0e3af60`, so search by name):

- **Back** runs `window.history.back()` via `injectJavaScript` (line 250 on),
  never `webViewRef.goBack()`. This is load-bearing: native `goBack()` does not
  reliably keep `window.history.state`, React Router falls back to the key
  `"default"`, and the site's scroll memory breaks ("Back goes to the top").
- **Reopens on the last page**: `RESTORABLE_ROUTES` allow-list (line 63),
  `readSavedUrl`/`saveUrl` to `last-route.json`, plus a one-time Back step up
  to `/shop` or `/seller` after a cold launch on a deep page
  (`pendingUpTarget`, `window.location.replace`).
- **Sideways drift fix**: `LOCK_HORIZONTAL_SCROLL` injected CSS
  (`overscroll-behavior-x: none`). Its comment says never add an `overflow` rule
  to `<html>` — that breaks `window.scrollY` and the site's scroll restore.
- `setBuiltInZoomControls={false}`, `overScrollMode="never"`.
- **Push**: `orders` channel created on start (line 219), `onMessage` handler
  (line 523), `enablePush()` (line 206), tap-to-open (`tapPath`/`safePath`,
  lines 172–177), token rotation listener.
- **Start page rule**: tapped notification > saved last page > site root,
  decided synchronously by `launchTarget()` before the first render (see
  issue 5 below).
- **External apps**: any non-web URL (`upi:`, `tel:`, `mailto:`,
  `whatsapp:`) goes to `Linking.openURL` (`handleExternalUrl`); the manifest's
  `<queries>` lists those schemes. No UPI app → Marathi pop-up.
- **Hardening** (`6be6235`): `mixedContentMode="never"` (the HTTPS site cannot
  load `http://` resources) and `android:allowBackup="false"`, set in both the
  manifest and `app.json`, so Android never copies the WebView's storage,
  including the login token, to Google Drive.

## The push handshake (contract with the web app)

1. The page (`frontend/src/lib/pushBridge.ts` in the web repo) posts
   `{ type: 'push:enable' }` through `window.ReactNativeWebView.postMessage` —
   only inside the APK, only when a seller or buyer is signed in, again on each
   sign-in and language change.
2. `window.ReactNativeWebView` exists **only because `onMessage` is set** on the
   WebView. Removing that prop silently disables push.
3. The wrapper asks Android permission and calls
   `window.__smbPushStatus(granted)` with the answer. If granted, it gets the
   FCM token (`getDevicePushTokenAsync`) and calls
   `window.__smbPushToken(token)`.
4. The page sends the token to `POST /api/push/token`. The server keeps the
   token on the session, so logout stops notifications.
5. On `granted === false` the page shows a card with a "सेटिंग उघडा" button,
   which posts `{ type: 'push:settings' }`; the wrapper calls
   `Linking.openSettings()`. When she returns to the app (`AppState` active)
   the wrapper re-checks and reports again, sending the token if now granted.
6. A notification's `data.path` is opened on tap only if it starts with `/` and
   not `//` or `/\` (`safePath`).

An APK built before this follow-up never calls `__smbPushStatus`, so the card
never shows there. That is intended, not a bug.

## Permissions: current state

`app.json` `android.permissions` and the main `AndroidManifest.xml` declare the
same list, and both block the two storage permissions:

| Permission | Why |
|---|---|
| `CAMERA` | Declared, never requested — keeps the photo picker gallery-only (below) |
| `INTERNET` | The site |
| `POST_NOTIFICATIONS` | Push on Android 13+. expo-notifications 0.32.17 also merges it in |
| `RECORD_AUDIO` | Voice typing |
| `VIBRATE` | Harmless; expo-haptics merges it in anyway |
| `READ_/WRITE_EXTERNAL_STORAGE` | **Blocked** (`tools:node="remove"` + `blockedPermissions`). Kept as a guard; `react-native-blob-util` added them and has been uninstalled |

Library manifests also merge in (checked in their published sources, not in a
built APK): `RECEIVE_BOOT_COMPLETED` (expo-notifications),
`ACCESS_NETWORK_STATE`, `WAKE_LOCK`, Firebase's
`com.google.android.c2dm.permission.RECEIVE`, and launcher-badge permissions
for several phone brands. All are normal-level; none needs a Play
declaration. `aapt dump permissions` on the release APK is the final word.

Not needed at all: clipboard (the site only writes), gallery access (Android's
picker needs no permission), background running (FCM delivers to a closed app).

### Why `CAMERA` stays

Checked in `react-native-webview` 13.16 source (`RNCWebViewModuleImpl.
startPhotoPickerIntent` / `needsCameraPermission`): when the page opens
`<input type="file" accept="image/*">` (the web app's `PhotoPicker`):

- `CAMERA` declared but not granted → picker offers **gallery only**, no prompt.
  This matches the product rule "one photo, from the gallery".
- `CAMERA` removed → the picker **adds a "take photo" option** via the camera app.

So removing it changes behaviour. Leave a comment saying so. (Real camera
capture is on the web app's "not built yet" list; removing `CAMERA` is the
cheapest way to get it when wanted.)

### How to change permissions safely

- `android/` is committed, so editing `app.json` alone does nothing until
  `npx expo prebuild --platform android` — **never `--clean`**, which discards
  hand-made native changes. Commit first, prebuild, then diff `android/`.
  Or edit `app.json` and the manifest by hand, identically.
- Libraries merge their own permissions in. Check the final list with
  `aapt dump permissions <apk>`. Block an unwanted merged one with
  `android.blockedPermissions` in `app.json`.

## Status after the follow-up (26 September 2026)

Built as a release APK (debug-signed) and **tested on a Samsung Galaxy S24 FE
(Android 16) and a second phone on 26 September**: every item of the test
checklist below passed, including the mic. Testing found one bug, fixed
in `ea8689a`: the load-error screen took half the height and Android's English
"Web page not available" page showed above it — react-native-webview renders
`renderError`'s view beside the WebView, so it must cover it absolutely.

| # | Issue | State |
|---|---|---|
| 1 | Permission cleanup | Done; confirmed with `aapt` on the release APK |
| 2 | `POST_NOTIFICATIONS` | Done; present in the release APK |
| 3 | Refused permission invisible | Done in both repos (handshake steps 3 and 5) |
| 4 | Mic never requested | Tested on the phone: works, no change needed |
| 5 | Cold-start tap loads twice | Done: synchronous `getLastNotificationResponse()` in `launchTarget()` picks the start page before the first render; Back then steps up to `/seller` or `/shop` |
| 6 | Stale launch response | Done: `clearLastNotificationResponse()` after use (it exists in 0.32.17) |
| 7 | Cold-start tap navigates twice | Done: taps de-duplicated by notification identifier |
| 8 | Channel settings fixed | Comment at `setNotificationChannelAsync` |
| 9 | English pop-ups | Done: Marathi alerts, Marathi error screen with a retry button; the extra "WebView error" alert is gone |
| 10 | Debug keystore | Done: upload keystore created 27 September (`CN=Team Zenith`, SHA-256 `70:5C:39:0C:…:A7:EA`); the four `SMB_UPLOAD_*` values are in `~/.gradle/gradle.properties` on the building laptop |
| 11 | `debug-download.md` | Rewritten |

### Since then

| Commit | Change |
|---|---|
| `8c9449c` (26 Sep) | `react-native-blob-util` uninstalled; that also drops `ACCESS_WIFI_STATE` and `DOWNLOAD_WITHOUT_NOTIFICATION`. Storage-permission blocks kept as a guard. |
| `e2ab970` (27 Sep) | Package `com.siddharam_sutar.mywebviewapp` → `in.shantai.mahilabazar`, deep-link scheme `shantaimahilabazar`, slug `shantai-mahila-bazar`. Both `google-services.json` copies list the new app. |
| `6be6235` (28 Sep) | `allowBackup="false"` and `mixedContentMode="never"`, the two wrapper items from the web repo's Play readiness review. |
| — (28 Sep, 04:19) | Signed `app-release.aab` built from `6be6235` with the upload key. |

The package rename and hardening have not been through the phone checklist
again. The `.aab` has not been installed on a phone; `assembleRelease` from the
same commit gives an APK to test with.

## Still to do

1. **Re-run the phone checklist** below on a build with the new package name,
   and check its permissions (`aapt dump permissions`, see `README.md`).
2. **Upload the `.aab` to closed testing** (`versionCode 1`, `1.0.0`) and run
   Play's 12-tester, 14-day test if the account needs it. Field-by-field
   answers: `PLAY-CONSOLE-FILL.md`. Its section 0 lists the web-side blockers
   (reviewer account and demo OTP, remaining privacy rows).
3. **Next update (1.0.1):** the rest of the web review's "Android wrapper"
   list. That covers marking camera and mic as not required, opening other
   websites in the browser, no debug logging in release builds, and the brand
   spinner colour. None blocks review. Each is written up, with the release
   steps, in `ANDROID-FUTURE-RELEASES.md`.

## Keep in mind

- **Never remove `onMessage`** from the WebView. It is what creates
  `window.ReactNativeWebView`, so push dies silently without it.
- **Never use `webViewRef.goBack()`** for Back; keep `window.history.back()`.
- **Never add an `overflow` rule to `<html>`** in injected CSS.
- **Never `expo prebuild --clean`**: `android/` has hand edits (manifest
  comments and `tools:node="remove"`, release signing in `app/build.gradle`).
  A plain prebuild can also rewrite the manifest; diff `android/` after one.
- **Keep `app.json` and `AndroidManifest.xml` permissions identical**, and
  keep `allowBackup` `false` in both.
- **Keep `mixedContentMode="never"`.** The site is HTTPS only.
- **Changing the notification channel** (importance, sound) needs a new
  channel id in `app/index.tsx` and in `app.json`'s `defaultChannel`
  (`default_notification_channel_id` in the manifest).
- **A new web route** that should reopen after a restart must be added to
  `RESTORABLE_ROUTES`; everything else is excluded by default.
- **Package name** `in.shantai.mahilabazar` is permanent after the first Play
  upload. `in` is a Kotlin keyword, so `MainActivity.kt` and
  `MainApplication.kt` declare ``package `in`.shantai.mahilabazar``. A
  `prebuild --clean` would regenerate them unescaped and break the build —
  one more reason never to run it.
- **`google-services.json` must list `in.shantai.mahilabazar`** (both copies:
  the root one and `android/app/`). Register the Android app under that name
  in the `shantaimahilabajar` Firebase project and download the file again;
  the old file names only the old package, and the build fails without a
  match. After a package rename, delete `android/app/build/generated/autolinking`
  and `android/build/generated/autolinking`, or the cached entry point still
  imports the old package's `BuildConfig`.
- `versionCode` is `1` in `android/app/build.gradle`, for the first upload;
  every upload after it needs it raised.
- **Release builds need the upload key.** Without the `SMB_UPLOAD_*`
  properties Gradle silently falls back to the debug key (with a warning) and
  Play refuses the file. Check with `keytool -printcert -jarfile <aab>`.
- **The APK has no ₹50 seller-fee flow.** Since web commit `77110fa`, the site
  hides the price, the college QR and the UTR form inside the APK (Play
  Billing rules); sellers pay at the college desk and staff record it. Do not
  add any payment link or bridge for it to the wrapper. The web review
  sketches a Play Billing bridge (`billing:buy`) if that ever changes.
- Any location permission coming back (e.g. through a new library) means a
  Play declaration form. Run `aapt` after adding any native dependency.
- Xiaomi/Oppo/Vivo/Realme block closed-app notifications until Autostart is
  allowed and battery is "No restrictions"; the app cannot request that, and
  the web app's seller help card explains it.
- `components/HelloWave.tsx` and `ParallaxScrollView.tsx` are unused Expo
  template files that fail `tsc` (missing `react-native-reanimated`). Harmless,
  but they make `tsc -p .` noisy.

## Test checklist on a real phone

- Fresh install → sign in as seller → Android notification prompt appears now,
  not on the landing page.
- Order placed for that seller from another device: foreground, background, and
  swiped away — "नवीन ऑर्डर आले आहे" arrives with sound each time.
- Tap the notification with the app closed → opens that order directly (no
  flash of the last page); Back goes to `/seller`, Back again exits.
- Tap a notification with the app open → opens that order once.
- Refuse the prompt → the "फोनवर सूचना बंद आहेत" card appears → "सेटिंग उघडा"
  opens Android settings → turn notifications on → back in the app the card
  disappears. "नंतर करते" hides it until the app is closed.
- Fresh install → tap a mic in any form field → prompt or "denied" (issue 4).
- Add a product photo → picker shows gallery only.
- Airplane mode → cold launch → Marathi error screen; turn data on →
  "पुन्हा प्रयत्न करा" loads the site.
- UPI payment with no UPI app installed → Marathi pop-up.
- Scroll deep in the catalogue → open a product → Back → same place.
- Kill and reopen the app → reopens on the last page; Back steps up to
  `/shop` or `/seller` once, then exits.
- `aapt dump permissions` on the release APK matches the intended list.

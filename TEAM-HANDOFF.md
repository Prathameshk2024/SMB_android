# Handoff: getting Shantai Mahila Bazar onto Google Play

Written 28 September 2026 for whoever picks this up next, in a new chat session.
Code is cited as `file:line`. Line numbers drift, so search by name if one looks off.

**Starting a Claude session:** open it in the repo you will change (mostly the web app, below) and
tell it to read this file first. Then, in the web repo, `CLAUDE.md` and `docs/PLAY-READINESS-REVIEW.md`;
in this repo, `ANDROID-WRAPPER-HANDOFF.md`.

---

## 1. The project in one minute

- **The app is a WebView wrapper.** The Android app (this repo) is one screen that loads the live
  site. Nothing of the site is inside the `.aab`. A web deploy changes the app for everyone at once,
  with no new build and no Play review. A new `.aab` is needed only when the wrapper itself changes.
- **Two repos:**

| | Web app (site, API, admin console) | Android wrapper (this repo) |
|---|---|---|
| GitHub | `Prathameshk2024/Shantai_mahila_bajar_app` | `Prathameshk2024/SMB_android` |
| Branch | `prathamesh2` (GitHub default; Vercel builds from it) | `sub-main` (**not** `main`, which is older) |
| On the main laptop | `S:\Codes and Programs\Programs\Dev\SMB\Shantai_mahila_bajar_app` | `S:\…\SMB\SMB_Android_App\Android_app` |
| Deploys to | Vercel (site + admin), Cloud Run `shantai-api` (API): `docs/DEPLOY.md` §1–2 | Google Play, package `in.shantai.mahilabazar` |

- **Detecting the APK:** the site checks for `window.ReactNativeWebView` (`frontend/src/lib/inApk.ts`).
  Any dictionary key with a `.apk` twin (e.g. `prod.slotsFullBody.apk`) shows the twin inside the app.
- **The ₹50 seller fee must never appear in the APK.** The price, the payment QR and the UTR form
  all stay out of it. Google Play would require Play Billing for it. Sellers pay at the college desk
  and staff use "Record payment" in the admin console (web commit `77110fa`).

## 2. Where things stand

**Done**
- Wrapper finished and tested on two phones. Package renamed, backups off, mixed content blocked
  (`sub-main` at `6be6235`, pushed).
- Upload keystore created (27 Sep). A signed release bundle was built on 28 Sep from `6be6235`:
  `android\app\build\outputs\bundle\release\app-release.aab` on the main laptop. **Not uploaded to Play yet.**
- Web app: nearly all Play-readiness work is done and pushed to `prathamesh2`, so Vercel has it.
- Every Play Console answer is written out in `PLAY-CONSOLE-FILL.md` (this repo).

**Only on the main laptop, not in git.** Ask the owner to commit and push these, or you won't have them:
- Web repo working tree: comment and hint fixes (conflict markers inside a comment in `Auth.tsx`,
  stale 48-hour and image-cache comments, the `npm run admin:users -- hash` hint, the
  `backend/.env.example` header), plus a big docs refresh. It includes `PLAY-READINESS-REVIEW.md`,
  which re-checks every status as of 28 Sep.
- This repo: `PLAY-CONSOLE-FILL.md` is untracked; the `README.md` and `ANDROID-WRAPPER-HANDOFF.md`
  updates are uncommitted.
- **The upload keystore and its passwords** (`~/.gradle/gradle.properties` on that laptop). Only that
  machine can build a release Play accepts. Elsewhere, Gradle falls back to the debug key (see `README.md`).

**Not verified:** whether Cloud Run is running the latest API. Many fixes are in the code, but they
count only after the API is deployed (`docs/DEPLOY.md` §1).

## 3. Web-app bugs to fix

All of these were confirmed in the code on 28 Sep. **None needs a new `.aab`**: fix, deploy, done.
**Do 1 and 2 before submitting to Play.** Several need the owner's decision first; those are marked.

### 1. The in-app deletion text says less than the privacy policy (Play risk)
- The APK text `del.whatStays.apk` (`frontend/src/i18n/strings.ts:150` mr, `:1014` en) mentions only orders and fees.
- The web text `del.whatStays` (`:696`, `:1529`) also says four things the APK text leaves out:
  - reviews are kept;
  - complaints are kept;
  - a blocked buyer's number is kept;
  - deleted data leaves backups within 12 months.
- Play compares this page with the privacy policy and the Data safety answers.
- **Fix:** add those four points to both `.apk` strings, still without naming the ₹50.

### 2. ₹50 can still show inside the APK through server errors (Play risk)
- `backend/src/routes/products.routes.ts` sends Marathi messages that name the price:
  - 402 slots full: "…50 रुपये भरा" (`:108`, `:250`);
  - 403 subscription expired: `EXPIRED_MR`, "₹50 भरून…" (`:70`).
- `UploadProduct.tsx:244` and `EditProduct.tsx:184` display `err.messageMr` as is, so the `.apk`
  twins never apply. They also show it in Marathi when the app is in English.
- The UI usually blocks a full-slots upload earlier. These messages appear when slots fill on
  another device, or when the term ends mid-wizard.
- **Fix (pick one):**
  - On 402, and on 403 with `error: 'Subscription expired'`, show dictionary text instead.
    `prod.slotsFullBody` already has an `.apk` twin; the expiry case needs one.
  - Or drop the price from the server messages.

### 3. Upload wizard promises "goes live straight away"
- The last screen's title is `prod.liveNow`: "लगेच प्रकाशित होईल" / "Goes live straight away"
  (`strings.ts:508`, `:1353`; used at `UploadProduct.tsx:560`).
- Every listing waits for staff approval: `initialListingStatus()` never returns `LIVE`.
- **Fix:** reword it to say the listing is sent for checking, matching the `ok.productPublished`
  toast. The comment below it ("Publishing is hers now") is stale too.

### 4. A buyer can't correct a mistyped UTR (decision)
- `POST /orders/:id/pay` (`backend/src/routes/orders.routes.ts`) accepts only `UPI_PENDING`. After
  the first submit the order is `UPI_SUBMITTED`, so a corrected UTR gets a 409.
- The route's own comment says correcting a digit should work.
- **Decide:** allow resubmission while `UPI_SUBMITTED`, until the seller confirms? If so, should the
  seller be told again?

### 5. The phone box can keep the wrong ten digits
- Typing `+91 98220 11223` into the login phone field (`Auth.tsx:182-186`) keeps `9198220112`:
  - the non-digits are stripped, but the `91` stays;
  - `maxLength={10}` applies to the raw text, so the input stops there;
  - the result looks valid, so the OTP goes to someone else.
- The WhatsApp field in `SellerRegister.tsx:342` has the same pattern.
- **Fix:** run the input through `normalizePhone()` (`shared/src/seller.ts:252`), which strips a
  `91` or a leading `0`, then limit to ten digits. Don't cap the raw input.

### 6. Admin can reject an approved payment, with no reason (decision)
- `POST /admin/payments/:id/reject` (`backend/src/routes/admin.routes.ts:229`) has no `PENDING`
  check; approve does have one.
- With no reason given, it stores and sends the seller "UTR did not match the bank statement".
- Rejecting an approved payment doesn't take back the slots or time it granted.
- **Decide:** block rejecting anything but `PENDING` (simplest), or allow it with a proper reversal.
  Either way, require a reason.

### 7. FSSAI never shows on a product (decision)
- The product page shows `product.fssai` (`frontend/src/screens/customer/Browse.tsx:376`) and the
  API accepts it, but no listing form sends it.
- The seller's own FSSAI (collected at registration) is never shown publicly, yet the seller
  agreement says buyers can see it.
- **Decide:** add FSSAI to food listing forms, show the seller's number on her food listings, or
  change the agreement's sentence.

### 8. Buttons are 54px; the design rule says 56px (decision, cosmetic)
- `--btn-h: 54px` (`frontend/src/styles/theme.css:95`); the rule is in `CLAUDE.md` *Design rules*
  and `docs/FEATURE-SPEC.md`.
- **Decide:** change the token or the rule. `MANUAL-TEST-PLAN.md` already notes the gap.

## 4. What else stands between us and Play

In order. Details are in `PLAY-CONSOLE-FILL.md` §0 and the web repo's `PLAY-READINESS-REVIEW.md`.

1. **Deploy the API to Cloud Run** with everything on `prathamesh2`, plus bug fixes 1–2.
2. **Reviewer account on production:**
   - demo shop, products and sample orders for `9579642050`;
   - then change the MSG91 demo OTP to a random 6-digit code and put it in the App access form.
3. **Remaining privacy rows** in the review's *Privacy policy and Data safety form vs the code*
   table. Its statuses were re-checked on 28 Sep; trust it over older notes.
4. **Store graphics:** feature graphic (1024×500) and 2–8 phone screenshots. Not made yet.
5. **Upload** the `.aab` to closed testing, from the main laptop's Play Console account (Team Zenith).
   A personal account needs 12 testers opted in for 14 days in a row before production.

**Optional wrapper polish** (needs a new `.aab`; listed in `ANDROID-WRAPPER-HANDOFF.md` → *Still to do*):
- camera and mic `uses-feature required="false"`;
- open other sites in the phone's browser;
- no `console.log` in release builds;
- brand-coloured spinner.

If this is done before the first upload, keep `versionCode 1`. After that, raise it for every upload.

## 5. Don't break these

- **Web:** nothing inside the APK names the ₹50 or offers a way to pay. Any new string that mentions
  it needs an `.apk` twin, and a test fails if it has none.
- **Web:** the deletion page, the privacy policy and the Data safety answers (`docs/PLAY-STORE.md`)
  must say the same things. Change one, change all three.
- **Wrapper:**
  - never remove `onMessage` (it's what creates `window.ReactNativeWebView`, so push and APK
    detection would die);
  - never use native `goBack()`;
  - never run `expo prebuild --clean`.

  Full list: `ANDROID-WRAPPER-HANDOFF.md` → *Keep in mind*.
- **Release:** a build signed with the debug key is refused by Play. Check the signer with
  `keytool -printcert -jarfile <aab>` before uploading.

# Play Console – copy-paste sheet

This sheet follows the Play Console screens in order. The answers come from the code and from
`docs/PLAY-STORE.md` and `docs/PLAY-READINESS-REVIEW.md` on the web app's `prathamesh2` branch.
Fields marked **⚠** need a decision or a fix before you submit.

Site: `https://shantai-mahila-bajar-app-frontend.vercel.app`

---

## 0. Before you start (blockers)

- **Signing: done.** The `.aab` built on 2026-09-28 is signed with the upload key (`CN=Team Zenith`, SHA-256
  `70:5C:39:…:A7:EA`), not the debug key. Keep a backup of the keystore and its passwords; losing them means
  asking Play support for an upload-key reset.
- **⚠ Reviewer account.** Two numbers are MSG91 Demo Credentials (no SMS is sent to either): `9579642050`
  and `9999999999` (web `86e89e6`, `03bd22a`). `9999999999` is refused by a check outside the web repo, so test
  a sign-in with it before choosing it; `9579642050` was tested end to end on 27 Sep. Pick **one**, then:
  - set up the demo shop, products and sample orders under it;
  - change the MSG91 demo OTP to a random 6-digit code;
  - use that number throughout §4 *App access*.

  The checklist is in `PLAY-READINESS-REVIEW.md` → "Production data to prepare". The API must include
  `03bd22a` for both numbers to count as demo numbers; redeploy if unsure.
- **Wrapper fixes: done and in the `.aab`.** `android:allowBackup="false"` (AndroidManifest, and `app.json` so a
  prebuild keeps it) and `mixedContentMode="never"` (`app/index.tsx`) landed in `6be6235` at 02:58. The `.aab` was
  built after that, at 04:19, so it already has them. The web review's other wrapper items (`uses-feature`
  camera/mic not required, outside links in the browser, release logging, spinner colour) are planned for
  version 1.0.1. They don't block review. See `ANDROID-FUTURE-RELEASES.md`.
- **Web fixes: live.** The eight bugs in `TEAM-HANDOFF.md` §3 (web `b3df936`) are on Vercel and Cloud Run, as of
  28 Sep evening. That includes the fuller deletion text inside the app and no ₹50 in the upload errors.
- **Privacy promises: partly fixed, and what's fixed is live.** Done: a closed seller's listings and photos are
  erased, complaints lose the name and phone (`6946caa`, `d523457`), and the policy's agree button confirms 18+
  (`9e91006`). Dated backup copies now prune themselves after 12 months, and the policy says so (`e067987`), but
  the review table's Backups row isn't marked Done yet. Still open: stray fields (`landmark`, `fssai`,
  `shopSlug`, `closeNote`), hosting logs, GitHub in the provider list, security-log pruning, and the ₹50 payer UPI
  ID. The API running today includes all the fixes above. Re-check the table in `PLAY-READINESS-REVIEW.md` →
  "Privacy policy and Data safety form vs the code" before submitting.
- **No ₹50 fee flow inside the APK** (web `77110fa`). Inside the app, sellers see only their shop's status, never
  the price, QR or UTR form. They pay at the college desk and staff record it. This keeps the seller fee outside
  Play Billing. The reviewer won't see a payment screen in the app.
- **Personal account → testing rule.** Team Zenith is a personal account. If it was created after Nov 2023,
  Production needs a closed test first: at least 12 testers, opted in for 14 days in a row.

---

## 1. Create app

| Field | Enter |
|---|---|
| App name | `शांताई महिला बाजार` |
| Default language | Marathi – mr-IN (add English (India) – en-IN as a translation later) |
| Package name | `in.shantai.mahilabazar` |
| App or game | App |
| Free or paid | Free (you can't change this to paid later) |
| Declarations | Tick all three (Developer Program Policies, US export laws, Play App Signing terms) |

---

## 2. Store settings (Grow → Store presence → Store settings)

| Field | Enter |
|---|---|
| App category | Shopping |
| Tags | **Shopping**, **Food & drink**, **Business** (only these three fit; Play's list has no Marketplace or Handicraft tag) |
| Email address (public) | `principal.jascca@gmail.com` |
| Phone (public, optional) | `+91 9420488874` |
| Website (optional) | `https://shantai-mahila-bajar-app-frontend.vercel.app` |
| External marketing | Leave on (default) |

---

## 3. Main store listing

### Marathi (default, mr-IN)

**App name** (max 30)
```
शांताई महिला बाजार
```

**Short description** (max 80)
```
गावातील महिलांनी बनवलेले पापड, लोणची, मसाले व हस्तकला थेट त्यांच्याकडून घ्या.
```

**Full description** (max 4000)
```
शांताई महिला बाजार हे महाराष्ट्रातील ग्रामीण महिला उद्योजिकांचे डिजिटल व्यासपीठ आहे. जवाहर कला, विज्ञान व वाणिज्य महाविद्यालय, अणदुर यांच्यातर्फे हे ॲप चालवले जाते.

कला तुमच्या गावाची, बाजारपेठ डिजिटल जगाची.
पापड, लोणची, मसाले, हस्तकला — गावातल्या महिला उत्तम वस्तू बनवतात. शांताई महिला बाजार त्या वस्तू थेट ग्राहकांपर्यंत पोहोचवतो. मधले दलाल नाहीत, पैसे थेट तिच्या खात्यात.

ग्राहकांसाठी
• घरगुती खाद्यपदार्थ, लोणची, मसाले, हस्तकला व भरतकाम — प्रकारानुसार पाहा
• विक्रेतीचे दुकान, तिचे रेटिंग व अभिप्राय पाहा
• कॅश ऑन डिलिव्हरी किंवा UPI ने थेट विक्रेतीला पैसे द्या
• ऑर्डर कुठपर्यंत आली ते पाहा आणि सूचना मिळवा
• खरेदी केलेल्या वस्तूला रेटिंग व अभिप्राय द्या

महिला विक्रेतींसाठी
• फक्त मोबाईल नंबर व OTP ने नोंदणी
• फोटोसह तुमच्या वस्तू टाका — प्रत्येक वस्तू महाविद्यालयाचे कर्मचारी तपासून मगच दिसते
• ऑर्डर स्वीकारा, सांभाळा आणि तुमचे ग्राहक पाहा
• स्वतःच्या QR कोडने तुमचे दुकान शेअर करा
• तुमचा व्यवसाय कसा वाढतोय ते पाहा, अभिप्राय वाचा
• ॲपमध्येच मदत व प्रशिक्षण

सर्वांसाठी सोपे
• मराठीत, एका टॅपवर इंग्रजी
• मोठी बटणे आणि बोलून टाइप करण्याची सोय
• Android 7.0 व त्यापुढील फोनवर चालते

लक्षात ठेवा
• सध्या फक्त महाराष्ट्रातील पिनकोडवर डिलिव्हरी.
• ग्राहक पैसे थेट विक्रेतीला देतात — ॲप तुमचे पैसे स्वतःकडे ठेवत नाही.
• 'माझी प्रोफाईल' मधून तुम्ही कधीही तुमचे खाते हटवू शकता.
```

### English (en-IN translation)

**App name** (max 30)
```
Shantai Mahila Bazar
```

**Short description** (max 80)
```
Buy homemade papad, pickles, masala & crafts directly from rural women sellers.
```

**Full description** (max 4000)
```
Shantai Mahila Bazar is a digital marketplace for rural women entrepreneurs in Maharashtra, run by Jawahar Arts, Science & Commerce College, Anadur.

Her skill is rural. Her market is digital.
Papad, pickles, masala, handicrafts – rural women make excellent things. Shantai Mahila Bazar takes them straight to customers. No middlemen, and the money goes into her own account.

FOR BUYERS
• Browse homemade food, pickles, masala, handicrafts and embroidery by category
• Visit a seller's shop and see her ratings and reviews
• Pay with Cash on Delivery, or pay the seller directly by UPI
• Track your order and get notified as it moves
• Rate and review what you bought

FOR WOMEN SELLERS
• Register with just your mobile number and an OTP
• List your products with photos – college staff check every listing before it goes live
• Accept and manage orders, and see your buyers
• Share your shop with your own QR code
• See how your business is growing and read your reviews
• Help and training inside the app

EASY FOR EVERYONE
• Marathi first, English one tap away
• Big buttons and voice typing in forms
• Works on Android 7.0 and above

GOOD TO KNOW
• Delivery is currently limited to Maharashtra pincodes.
• Buyers pay the seller directly – the app never holds your money.
• You can delete your account at any time from My Profile.
```

### Graphics (same for both languages)

| Asset | Spec | Source |
|---|---|---|
| App icon | 512×512 PNG, 32-bit, ≤1 MB | Downscale `SMB App Logo.png` (2048²) |
| Feature graphic | 1024×500 PNG/JPG, no alpha | **⚠ Not made yet** |
| Phone screenshots | 2–8, 9:16, each side 320–3840 px | **⚠ Take from the phone**: landing page, category list, product page, cart/checkout, seller dashboard, My Products |
| Tablet screenshots | Optional | Skip |

---

## 4. App content (Policy → App content) – top to bottom

### Privacy policy
```
https://shantai-mahila-bajar-app-frontend.vercel.app/legal/privacy
```

### App access
Choose **All or some functionality is restricted**. Then click **Add instructions**:

| Field | Enter |
|---|---|
| Name | `Buyer and seller login (same number)` |
| Username / phone | **⚠** the demo number you picked in §0: `9579642050` or `9999999999` |
| Password / OTP | **⚠** the fixed OTP you set in MSG91 |
| Any other information | paste below |

487 characters (limit 500). If you picked `9999999999`, change the number in step 2. Both are ten digits, so
the length stays the same:
```
Demo number, no SMS: use the code above.

BUYER
1. Opens in Marathi; tap "English" at top.
2. "I want to buy" > 9579642050 (no +91) > Send OTP > code.
3. Order only from "Demo shop (Play review)"; other shops are real.
4. Checkout: pincode 413603, Cash on Delivery.
5. My Profile > Delete my account > last 4 digits. Cancel open orders first.

SELLER
Log out, "I want to sell", same login. Shop is approved. Try Orders, My Products (new items need staff review), Reviews, Delete account.
```

### Ads
**No, my app does not contain ads.**

### Content rating (IARC questionnaire)
| Question | Answer |
|---|---|
| Email | `principal.jascca@gmail.com` |
| Category | **All other app types** (it is a shopping app for physical goods) |
| Violence, fear, sexuality, bad language, controlled substances, crude humour | No to all |
| Gambling / simulated gambling | No |
| Can users interact or exchange content? | **Yes** – public product listings and reviews (no chat) |
| Does the app share the user's current physical location with other users? | No |
| Can users buy digital goods? | No (physical goods only, paid off-app by COD/UPI) |
| Is it a web browser or search engine? | No |
| Unrestricted internet access? | No (it only opens its own site) |

Expected result: Everyone / 3+ or 12+, depending on how UGC is weighted. Either is fine.

### Target audience and content
| Question | Answer |
|---|---|
| Target age groups | **18 and over** only |
| Could the app unintentionally appeal to children? | No |
| Store presence | Not in "Designed for Families" |

### News app
No.

### COVID-19 contact tracing / status
My app is not a publicly available COVID-19 contact tracing or status app.

### Data safety
**Overview**
| Question | Answer |
|---|---|
| Does your app collect or share any of the required user data types? | Yes |
| Is all user data encrypted in transit? | Yes |
| Account creation methods | **Username and other authentication** only (Play counts a phone number as a username and an OTP as other authentication) |
| Delete account URL | `https://shantai-mahila-bajar-app-frontend.vercel.app/delete-account` |
| Can users request deletion of some data without deleting their account? | No |
| Independent security review / UPI badge | No / skip |

**Data types.** Tick only these. For every one: **Collected = Yes, Shared = No, Processed ephemerally = No**.

| Category → Type | Required / optional | Purposes to tick |
|---|---|---|
| Personal info → Name | Required | App functionality, Account management |
| Personal info → Phone number | Required | App functionality, Account management, Fraud prevention, security & compliance |
| Personal info → Address | Required | App functionality |
| Personal info → User IDs | Required | App functionality, Account management, Fraud prevention, security & compliance |
| Personal info → Other info | Required | App functionality, Analytics |
| Financial info → User payment info | Required | App functionality |
| Financial info → Purchase history | Required | App functionality |
| Financial info → Other financial info | Required | App functionality, Fraud prevention, security & compliance |
| Photos and videos → Photos | Required | App functionality |
| App activity → App interactions | Required | Analytics |
| App activity → Other user-generated content | Optional | App functionality |
| Device or other IDs → Device or other IDs | Optional | App functionality |

Leave these **unticked**: Location, Health and fitness, Messages, Audio (voice typing goes to the phone's speech
service, not to us), Files and docs, Calendar, Contacts, Web browsing, App info and performance.

What each type covers, so you can check it before ticking:
- **Address:** a buyer's delivery address; a seller's village, taluka, district and pincode.
- **Other info:** a seller's age, education, self-help group, business details, FSSAI number and readiness answers.
- **User payment info:** a seller's UPI ID and QR image.
- **Other financial info:** UTR numbers and ₹50 payment screenshots. **⚠** Since `77110fa` the APK no longer
  collects these (only the website does). The web repo's `PLAY-STORE.md` still declares them. Declaring more
  than the app collects is safe, but settle it with the ₹50 decision.
- **App interactions:** the seller activity used for her readiness score.
- **Device or other IDs:** the FCM push token.

### Government apps
No. It's run by a college and is not an app of a government or government body.

### Financial features
**My app doesn't provide any financial features.** It takes COD and UPI payments to the seller for physical goods,
but no payment processing, wallets, loans or crypto.

### Health apps
My app doesn't have any health features.

### Advertising ID
**No**, my app does not use an advertising ID.

### Photo and video permissions / Foreground service / Full-screen intent / Exact alarm
You shouldn't see these forms. The app declares no media, foreground-service or alarm permissions. If one does
appear, a library added the permission; run `aapt dump permissions` on the APK.

---

## 5. First release (Test and release → Testing → Internal / Closed testing)

| Field | Enter |
|---|---|
| App signing | Accept "Use Google-generated key" (default) |
| App bundle | `android\app\build\outputs\bundle\release\app-release.aab` |
| Release name | `1 (1.0.0)` |
| Release notes | paste below |

```
<mr-IN>
शांताई महिला बाजारची पहिली आवृत्ती.
</mr-IN>
<en-IN>
First release of Shantai Mahila Bazar.
</en-IN>
```

**Testers:** create an email list, add the testers' Gmail addresses, then send them the opt-in link from the
**Testers** tab.

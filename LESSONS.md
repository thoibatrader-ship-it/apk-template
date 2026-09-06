# Build Lessons (universal APK, no IAP)

Ye file Gemini ko har build/fix request ke saath context me di jayegi.
In sab rules ka source: 4 rounds ka manual Amazon Appstore build
experience — IAP-specific parts hata diye gaye hain, sirf generic
Capacitor/Android/Actions lessons rakhe gaye hain.

## Toolchain versions (fixed, mat badlo)
- Node.js: 22 (Capacitor CLI ko >=22 chahiye)
- JDK: 21 (Node 22 ke Capacitor Android project compile karne ke liye)
- Signing: `apksigner` + `zipalign` — kabhi `jarsigner` nahi (v1-only
  signature modern Android phones reject karte hain)

## MainActivity.java / native code rules
- Agar `BridgeActivity` ka koi lifecycle method override karo
  (`onCreate`, `onStart`, `onResume`, `onPause`, `onStop`, `onDestroy`),
  hamesha `public void` rakho — kabhi `protected` nahi. Java rule:
  override me access level kam nahi kar sakte.
- Sandbox me Android SDK/Gradle nahi hai, isliye hand-written Java
  compile-check nahi ho sakta pehle se — extra careful likhna, aur
  agar build fail ho to error log hi source of truth hai.

## Workflow / secrets
- 4 secrets chahiye: `KEYSTORE_BASE64`, `KEYSTORE_PASSWORD`, `KEY_ALIAS`,
  `KEY_PASSWORD` — ye kabhi Gemini ke text context me nahi aane chahiye,
  sirf GitHub Secrets API se seedha set hone chahiye.
- Naya app = naya repo = naya keystore. Purana keystore kabhi reuse
  mat karna (ek app se dusre app me).
- `versionCode`/`versionName` bump karna zaroori hai har naye build pe
  jo replace/update ke roop me publish ho raha ho.

## Jab build fail ho
- Actions run ke logs hi dekhna — panic mat karna, exact red error text
  hi root cause bataata hai.
- Kisi bhi suggested fix ko (chahe AI se aaya ho) blindly apply mat karna
  — pehle verify karo (ho sake to web-search se), aur check karo ki fix
  khud koi naya bug to nahi la raha.

## Runtime permissions (camera, storage, GPS, mic, notifications)
- Manifest me permission declare karna sirf pehla step hai — Android 6+
  (API 23+) pe runtime pe bhi user se popup se maangni padti hai. Plain
  HTML/JS (getUserMedia, geolocation API, etc.) chalane par WebView khud
  system permission dialog trigger kar deta hai agar manifest me
  corresponding `<uses-permission>` maujood hai — extra native code
  usually nahi likhna padta plain use-cases ke liye.
- Agar Gemini/user ki HTML me `@capacitor/camera`, `@capacitor/geolocation`,
  ya `@capacitor/local-notifications` jaisे plugin ke JS APIs (jaise
  `Camera.getPhoto()`) use karne ho, un plugin ke apne `requestPermissions()`
  call karna padta hai JS side se — sirf manifest permission kaafi nahi.
- Android 13 (API 33)+ pe storage permission split ho gaya hai:
  `READ_MEDIA_IMAGES` / `READ_MEDIA_VIDEO` / `READ_MEDIA_AUDIO` — purana
  `READ_EXTERNAL_STORAGE`/`WRITE_EXTERNAL_STORAGE` sirf `maxSdkVersion=32`
  tak declare karna hai (dono saath rakhne se koi conflict nahi, ye
  template ke `AndroidManifest.xml` me already is tarah set hai).
- Notifications (Android 13+): `POST_NOTIFICATIONS` bhi runtime-prompt
  permission hai — agar `@capacitor/local-notifications` use ho to JS se
  `LocalNotifications.requestPermissions()` call zaroori hai warna
  notification silently drop ho jaayegi.
- Agar build fail ho permission-related error se (e.g. plugin manifest
  merge conflict), pehle check karo ki plugin `npm install` ke baad
  `npx cap sync android` chala ya nahi — Capacitor native plugin ka
  Android code isi step se copy/link hota hai.
- Ye template minimal-by-default permission set rakhta hai (camera,
  storage/gallery, GPS, mic, notifications, vibrate, network-state).
  Contacts/Bluetooth/Calendar jaanbhoojke comment-out kiye gaye hain
  `AndroidManifest.xml` me — sirf tab uncomment karo jab specific app ko
  zaroorat ho, taaki har app ka permission-surface chhota rahe.

## Explicitly OUT of scope for this universal template
- Amazon Appstore IAP (PurchasingService, Public Key, ResponseReceiver,
  Fire Tablet targeting)
- Google Play Billing / IAP
- `.aab` App Bundle format (Google Play ke liye alag build step hai)

Agar koi user IAP maange, dashboard usko clearly bata de ki v1 sirf
plain/no-IAP APK support karta hai.

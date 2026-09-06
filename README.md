# Universal HTML → Signed APK Template

Ye repo ek GitHub **template repository** hai. Har naye app ke liye
"Use this template" se naya repo banao — is repo ko directly mat use karo.

## Har naye app me kya badalna hai

| Cheez | Kahan |
|---|---|
| App ka HTML/JS/CSS | `www/index.html` (poora replace) |
| App ID | `capacitor.config.ts` → `appId` |
| Android package | `android/app/build.gradle` → `applicationId` |
| App naam | `capacitor.config.ts` → `appName` |
| Signing key | naya keystore banao, kabhi purana reuse mat karo |

## Secrets jo GitHub repo Settings → Secrets and variables → Actions me
chahiye (browser-side se encrypt karke set kiye jaate hain, kabhi Gemini
ke context me nahi jaate):
- `KEYSTORE_BASE64`
- `KEYSTORE_PASSWORD`
- `KEY_ALIAS`
- `KEY_PASSWORD`

## Permissions (pre-added, camera/storage/GPS/mic/notifications/etc.)
`AndroidManifest.xml` me ye permissions already declared hain, saath me
corresponding Capacitor plugins `package.json` me added hain:

| Permission | Kis liye | Plugin |
|---|---|---|
| Camera | Photo/video capture | `@capacitor/camera` |
| Storage / Gallery (legacy + Android 13+ granular) | File pick, image/video/audio read-write | `@capacitor/filesystem`, `@capacitor/camera` |
| GPS (fine + coarse) | Location | `@capacitor/geolocation` |
| Microphone | Audio record | (via WebView `getUserMedia`, koi alag plugin nahi) |
| Notifications | Push/local notification popup | `@capacitor/local-notifications` |
| Vibrate | Haptic feedback | `@capacitor/haptics` |
| Network state | Online/offline detect | `@capacitor/network` |

Contacts / Bluetooth / Calendar jaanbhoojke **comment-out** kiye gaye hain
`AndroidManifest.xml` me (permission-surface chhota rakhne ke liye) — agar
kisi specific app ko chahiye to us app ke repo me manually uncomment karo.

⚠️ Sirf manifest permission add karna kaafi nahi hota — jis plugin ka JS
API use karo (jaise `Camera.getPhoto()`), uska `requestPermissions()`
bhi JS side se call karna padta hai runtime pe. Detail `LESSONS.md` me hai.

## App Icon
`assets/icon.png` (1024×1024 recommended, square, no transparency issues)
is repo ke root me daalo, tab build workflow khud `capacitor-assets
generate --android` chala ke saare mipmap sizes + adaptive icon generate
kar dega. Agar `assets/icon.png` missing ho to workflow silently template
ka default icon use kar leta hai — build fail nahi hoga.

## Build
`main` branch pe push karte hi `.github/workflows/build-apk.yml` chalega:
1. `npm install`
2. Icon generate (agar `assets/icon.png` ho)
3. `npx cap sync android` (native plugin code link hota hai)
4. Gradle release build
5. Keystore decode → zipalign → apksigner sign
6. Keystore delete (cleanup)

Actions tab → run complete hone ke baad → Artifacts → `signed-apk`
download karo.

Is repo me koi In-App-Purchase code nahi hai (dekho `LESSONS.md` —
"Explicitly OUT of scope"). Ye sirf plain installable APK banata hai.

# Gif Widget

A home-screen widget app for Android that lets you pick **any GIF from your
gallery** (transparency preserved) and place it as its own widget — add as
many as you want, each with a different GIF.

## Building an APK without Android Studio

You have two no-Android-Studio options. Pick whichever fits what you have access to.

### Option A — build it right on your phone (Termux, no computer needed)
1. Install **Termux** from F-Droid (not the outdated Play Store version): https://f-droid.org/packages/com.termux/
2. Unzip this project onto your phone's internal storage (any file manager can unzip a `.zip`).
3. Open Termux and run:
   ```
   pkg update && pkg upgrade -y
   pkg install -y openjdk-17 unzip wget
   termux-setup-storage
   ```
4. Install just the Android SDK pieces needed to build (no Android Studio):
   ```
   cd ~
   wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
   unzip commandlinetools-linux-11076708_latest.zip -d android-sdk/cmdline-tools
   mv android-sdk/cmdline-tools/cmdline-tools android-sdk/cmdline-tools/latest
   export ANDROID_HOME=$HOME/android-sdk
   export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
   yes | sdkmanager --licenses
   sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"
   ```
5. Build the APK:
   ```
   cd /storage/emulated/0/<wherever you unzipped>/GifWidget
   ./gradlew assembleDebug
   ```
   The finished file appears at `app/build/outputs/apk/debug/app-debug.apk`.
6. Open that `.apk` with your file manager to install it (allow "install unknown apps" when prompted).

This downloads roughly 1-1.5 GB of SDK components the first time, so do it on Wi-Fi. Every step after that is copy-pasting commands — no coding needed.

### Option B — let GitHub build it for you (needs a browser, any device)
This project already includes a `.github/workflows/build.yml` that builds the APK automatically in the cloud.
1. Create a free GitHub account if you don't have one.
2. Create a new repository and upload every file/folder from this project to it (GitHub's web "Add file → Upload files" accepts a whole folder structure dragged in at once).
3. Go to the repo's **Actions** tab — a "Build APK" run should start automatically (or click "Run workflow" if not).
4. When it finishes (a couple of minutes), open the run and download the **GifWidget-debug-apk** artifact — that's a zip containing your `.apk`.
5. Transfer that `.apk` to your phone and open it to install (allow "install unknown apps").

No local tools at all needed for this option — GitHub's servers do the actual build.

## Building it the "normal" way (Android Studio), if you change your mind

1. Install [Android Studio](https://developer.android.com/studio) on a computer.
2. Open this folder (`GifWidget/`) as a project — `File > Open`.
3. Let it sync (Android Studio will auto-generate the Gradle wrapper if it's
   missing; just accept the prompt).
4. Plug in your Galaxy S24 Ultra via USB with USB debugging enabled
   (Settings → About phone → tap "Build number" 7 times → Developer options →
   USB debugging), then click **Run ▶** in Android Studio to install it directly.
   - Alternatively: `Build > Build App Bundle(s)/APK(s) > Build APK(s)`,
     copy the resulting `.apk` to your phone, and install it (you'll need to
     allow "install unknown apps" for whichever app you use to open it).
5. On your phone: long-press the home screen → **Widgets** → **Gif Widget**,
   drag it onto your home screen, and pick a GIF when prompted. Repeat for
   as many GIFs as you want — each widget instance keeps its own file.

## How it works / limitations to know about

- Android home-screen widgets can't natively host an animating GIF — they
  run through a restricted system called `RemoteViews`. This app works
  around that by decoding your GIF into a short sequence of transparent PNG
  frames (up to 30, sampled at ~10 fps) and swapping them on a timer.
- Because of that, there's a background **service with a small, low-priority
  notification** that keeps the animation ticking. This uses a bit of extra
  battery. If Samsung's battery optimizer kills it, go to
  **Settings → Apps → Gif Widget → Battery → Unrestricted** to keep it
  running reliably.
- Transparency works fine — PNG frames keep their alpha channel. One UI
  (Android 12+) sometimes adds its own rounded card background behind
  widgets; if you see that, long-press the widget → widget settings, and
  look for an option to remove the frame/background (varies by One UI
  version).
- Very long or very high-resolution GIFs are capped at 30 sampled frames to
  keep storage/memory reasonable — good enough for most short looping GIFs,
  but a very detailed multi-second animation will look less smooth than the
  original.

## Project structure

- `GifWidgetProvider.kt` — the widget itself (lifecycle, cleanup).
- `GifWidgetConfigureActivity.kt` — runs when you place a widget; lets you
  pick a GIF from the gallery.
- `GifFrameExtractor.kt` — decodes the GIF into transparent PNG frames.
- `GifAnimationService.kt` — the foreground service that cycles frames.
- `WidgetPrefs.kt` — per-widget settings (frame count/delay) storage.

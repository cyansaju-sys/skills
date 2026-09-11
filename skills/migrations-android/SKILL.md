---
name: migrations-android
description: >
  Migrates Android projects to targetSdk/compileSdk 36 (Android 16) and configures
  edge-to-edge (full screen, insets, predictive back). Use it when the user mentions
  "API 36", "Android 16", "targetSdk 36", "edge-to-edge", "edge to edge", "insets",
  "WindowInsets", "predictive back", "full screen on Android" (Spanish: "pantalla completa
  en Android") or asks to update an Android app's compatibility with the latest Google Play
  requirements. Covers both Views (XML) and Jetpack Compose projects.
---

# Skill: Migration to Android API 36 (Android 16) and Edge-to-Edge

## Purpose

Update an Android project to compile and work correctly with `targetSdk 36`
(Android 16), and leave the UI robustly configured edge-to-edge (no overlaps with
system bars, cutouts, or the keyboard), whether using Views or Compose.

Write all explanations in the user's language.

---

## Process

### 1. Confirm the starting point

Before touching code, identify:

- Current `compileSdk` / `targetSdk` (in `build.gradle` or `build.gradle.kts`)
- Android Gradle Plugin (AGP) and Gradle versions — API 36 requires **AGP 8.6+** and
  **Gradle 8.9+** as a reasonable minimum; recommend the latest stable versions
- Whether the project uses **Views (XML)**, **Jetpack Compose**, or both
- Whether it already calls `enableEdgeToEdge()` or handles insets manually
- Whether it uses `WindowCompat.setDecorFitsSystemWindows`, `android:windowOptOutEdgeToEdgeEnforcement`,
  or fixed status/navigation bar colors (a sign that edge-to-edge is not handled properly)

If the project isn't even on `targetSdk 35`, first validate the Android 15 changes
(edge-to-edge already enforced there with a temporary opt-out) before jumping to 36.

---

### 2. Update `compileSdk` / `targetSdk`

```kotlin
// build.gradle.kts (module :app)
android {
    compileSdk = 36
    defaultConfig {
        targetSdk = 36
        minSdk = 21 // adjust per project; not changed by this migration
    }
}
```

Also update relevant dependencies to versions compatible with API 36:

```kotlin
implementation("androidx.core:core-ktx:1.15.0")
implementation("androidx.activity:activity-ktx:1.10.0")       // enableEdgeToEdge()
implementation("androidx.activity:activity-compose:1.10.0")   // if using Compose
implementation("androidx.compose:compose-bom:2025.02.00")     // or a newer BOM
```

Build (`./gradlew assembleDebug`) and record **all** deprecation errors/warnings
before continuing — they are the real list of pending work.

---

### 3. Mandatory behavior changes in API 36

These are enforced by the system when declaring `targetSdk 36`; they are not optional:

| Change | What stops working | Required action |
|---|---|---|
| **Edge-to-edge without opt-out** | `android:windowOptOutEdgeToEdgeEnforcement="true"` is ignored on Android 16 devices | Implement real edge-to-edge (see section 4) |
| **Predictive back by default** | `onBackPressed()` and `KeyEvent.KEYCODE_BACK` no longer fire during system animations | Migrate to `OnBackPressedCallback` / `PredictiveBackHandler` (see section 5) |
| **`elegantTextHeight` ignored** | Compact fonts for Arabic, Thai, Tamil, etc. can no longer be forced via this attribute | Verify text layouts in those languages without relying on the attribute |
| **Orientation/resizability ignored on screens ≥ 600dp sw** | `android:screenOrientation`, `setRequestedOrientation()`, `minAspectRatio`/`maxAspectRatio` don't apply | Design adaptive layouts; if unavoidable, temporary opt-out with `PROPERTY_COMPAT_ALLOW_RESTRICTED_RESIZABILITY` (unavailable from API 37) |
| **Full-screen intents restricted** | Full-screen notifications (calls, alarms) require explicit permission | Declare and request `USE_FULL_SCREEN_INTENT` |
| **Granular health permissions** | `BODY_SENSORS` no longer covers heart-rate reading | Migrate to `android.permissions.health.*` permissions (e.g., `READ_HEART_RATE`) |

If the user only asked for edge-to-edge, briefly mention the other changes as an
additional checklist, but don't implement them unless asked.

---

### 4. Configure Edge-to-Edge

#### 4.1 Views (XML)

**Recommended option — `enableEdgeToEdge()`:**

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        enableEdgeToEdge() // before super.onCreate()
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

This replaces ~100 lines of manual compatibility code (bar transparency, contrast,
light/dark icons per theme) and also works on versions below 36.

**Manual handling (if the function above can't be used):**

```kotlin
WindowCompat.setDecorFitsSystemWindows(window, false)
```

```xml
<!-- values-v29/themes.xml -->
<style name="Theme.MyApp">
    <item name="android:navigationBarColor">@android:color/transparent</item>
    <item name="android:statusBarColor">@android:color/transparent</item>
    <item name="android:enforceNavigationBarContrast">false</item>
    <item name="android:enforceStatusBarContrast">false</item>
</style>
```

**Apply insets so content isn't covered by the bars:**

```kotlin
ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { view, insets ->
    val bars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
    view.updatePadding(left = bars.left, top = bars.top, right = bars.right, bottom = bars.bottom)
    insets
}
```

Use `WindowInsetsCompat.Type.ime()` instead of `systemBars()` for the keyboard, and
`displayCutout()` for notches/cutouts. Apply the inset to the right container (the root,
or only the elements that actually collide with the bar), not always the entire root,
so you don't lose the edge-to-edge effect (background behind the bars).

#### 4.2 Jetpack Compose

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        enableEdgeToEdge()
        super.onCreate(savedInstanceState)
        setContent {
            MyAppTheme {
                Scaffold(
                    contentWindowInsets = WindowInsets.safeDrawing
                ) { padding ->
                    MyScreen(Modifier.padding(padding))
                }
            }
        }
    }
}
```

- `WindowInsets.safeDrawing` covers system bars + cutout; use
  `WindowInsets.safeGestures` if you also need to respect gesture areas.
- For the IME, apply `.imePadding()` or `WindowInsets.ime` on the composable that needs it
  (typically a text field or a scrollable `Column`), not on the whole screen.
- Avoid combining `Modifier.statusBarsPadding()` with `Scaffold` insets at the same time:
  it doubles the padding.

#### 4.3 Bar icon colors (light/dark)

```kotlin
val controller = WindowCompat.getInsetsController(window, window.decorView)
controller.isAppearanceLightStatusBars = true   // dark icons on a light background
controller.isAppearanceLightNavigationBars = true
```

`enableEdgeToEdge()` already adjusts this automatically based on the theme if you pass
`SystemBarStyle.auto(...)`; only do it manually if you need behavior different from
the system theme.

---

### 5. Migrate Predictive Back (if applicable)

Manifest requirement:

```xml
<application android:enableOnBackInvokedCallback="true">
```

**Views:**

```kotlin
onBackPressedDispatcher.addCallback(this) {
    // custom back logic
    if (shouldHandleCustomBack) {
        // handle
    } else {
        isEnabled = false
        onBackPressedDispatcher.onBackPressed()
    }
}
```

**Compose:**

```kotlin
PredictiveBackHandler(enabled = canGoBack) { progress ->
    try {
        progress.collect { backEvent -> /* animation progress */ }
        // gesture completed
    } catch (e: CancellationException) {
        // gesture cancelled by the user
    }
}
```

Don't leave critical logic only in `onBackPressed()`: in apps with `targetSdk 36` running on
Android 16+, that callback **is not invoked** during the system's predictive animations.

---

## Verification Checklist

- [ ] `compileSdk`/`targetSdk = 36`, AGP and Gradle updated, project builds without errors
- [ ] `enableEdgeToEdge()` (or the equivalent manual setup) called in every relevant Activity
- [ ] No critical UI element is covered by the status bar, navigation bar, cutout, or keyboard
- [ ] Bar icon colors correct in both light and dark themes
- [ ] `android:windowOptOutEdgeToEdgeEnforcement` removed from the project
- [ ] Back navigation migrated to `OnBackPressedCallback` / `PredictiveBackHandler`, tested with the real predictive gesture (not just the back button)
- [ ] Full-screen notifications (if any) declare `USE_FULL_SCREEN_INTENT`
- [ ] Layouts reviewed on tablets/large screens (sw ≥ 600dp) without relying on forced orientation restrictions

**Test on:** a device/emulator running Android 16 (API 36), in light and dark mode,
with gesture navigation enabled, and on at least one screen with scrollable content
and a text field (to validate the keyboard inset).

---

## Notes

- If the project mixes Views and Compose (interop), apply insets at each one's entry point
  separately — don't assume Compose padding covers embedded XML views or vice versa.
- If the user asks for "just edge-to-edge" without mentioning API 36, you can still apply
  section 4 (it's backward compatible), but clarify that **enforcement** without opt-out
  only happens once reaching `targetSdk 36`.
- Exact behavior details may vary between the Developer Preview and the stable release
  of Android 16; if the project uses a preview, validate against the official release notes
  current at the time of the migration.

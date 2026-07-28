---
name: migrations-android
description: >
  Migra proyectos Android a targetSdk/compileSdk 36 (Android 16) y configura edge-to-edge
  (pantalla completa, insets, predictive back). Úsala cuando el usuario mencione "API 36",
  "Android 16", "targetSdk 36", "edge-to-edge", "edge to edge", "insets", "WindowInsets",
  "predictive back", "pantalla completa en Android" o pida actualizar la compatibilidad de
  una app Android con los últimos requisitos de Google Play. Cubre tanto proyectos con Views
  (XML) como con Jetpack Compose.
---

# Skill: Migración a Android API 36 (Android 16) y Edge-to-Edge

## Propósito

Actualizar un proyecto Android para compilar y funcionar correctamente con `targetSdk 36`
(Android 16), y dejar la UI configurada edge-to-edge de forma robusta (sin overlaps con
barras de sistema, cutouts o teclado), ya sea en Views o en Compose.

---

## Proceso

### 1. Confirmar el punto de partida

Antes de tocar código, identifica:

- `compileSdk` / `targetSdk` actuales (en `build.gradle` o `build.gradle.kts`)
- Versión de Android Gradle Plugin (AGP) y de Gradle — API 36 requiere **AGP 8.6+** y
  **Gradle 8.9+** como mínimo razonable; recomienda las últimas estables
- Si el proyecto usa **Views (XML)**, **Jetpack Compose**, o ambos
- Si ya llama a `enableEdgeToEdge()` o maneja insets manualmente
- Si usa `WindowCompat.setDecorFitsSystemWindows`, `android:windowOptOutEdgeToEdgeEnforcement`,
  o colores fijos de status/navigation bar (señal de que edge-to-edge no está bien manejado)

Si el proyecto no está ni siquiera en `targetSdk 35`, primero valida los cambios de
Android 15 (edge-to-edge ya forzado ahí con opt-out temporal) antes de saltar a 36.

---

### 2. Actualizar `compileSdk` / `targetSdk`

```kotlin
// build.gradle.kts (module :app)
android {
    compileSdk = 36
    defaultConfig {
        targetSdk = 36
        minSdk = 21 // ajusta según el proyecto; no cambia por esta migración
    }
}
```

Actualiza también dependencias relevantes a versiones compatibles con API 36:

```kotlin
implementation("androidx.core:core-ktx:1.15.0")
implementation("androidx.activity:activity-ktx:1.10.0")       // enableEdgeToEdge()
implementation("androidx.activity:activity-compose:1.10.0")   // si usa Compose
implementation("androidx.compose:compose-bom:2025.02.00")     // o BOM más reciente
```

Compila (`./gradlew assembleDebug`) y anota **todos** los errores/warnings de deprecación
antes de seguir — son la lista real de trabajo pendiente.

---

### 3. Cambios de comportamiento obligatorios en API 36

Estos son forzados por el sistema al declarar `targetSdk 36`, no son opcionales:

| Cambio | Qué deja de funcionar | Acción requerida |
|---|---|---|
| **Edge-to-edge sin opt-out** | `android:windowOptOutEdgeToEdgeEnforcement="true"` se ignora en dispositivos Android 16 | Implementar edge-to-edge real (ver sección 4) |
| **Predictive back por defecto** | `onBackPressed()` y `KeyEvent.KEYCODE_BACK` dejan de dispararse en las animaciones del sistema | Migrar a `OnBackPressedCallback` / `PredictiveBackHandler` (ver sección 5) |
| **`elegantTextHeight` ignorado** | Fuentes compactas en árabe, tailandés, tamil, etc. ya no se pueden forzar vía este atributo | Verificar layouts de texto en esos idiomas sin depender del atributo |
| **Orientación/resizability ignorados en pantallas ≥ 600dp sw** | `android:screenOrientation`, `setRequestedOrientation()`, `minAspectRatio`/`maxAspectRatio` no aplican | Diseñar layouts adaptativos; si es imprescindible, opt-out temporal con `PROPERTY_COMPAT_ALLOW_RESTRICTED_RESIZABILITY` (no disponible desde API 37) |
| **Full-screen intents restringidos** | Notificaciones full-screen (llamadas, alarmas) requieren permiso explícito | Declarar y solicitar `USE_FULL_SCREEN_INTENT` |
| **Permisos de salud granulares** | `BODY_SENSORS` ya no cubre lectura de ritmo cardíaco | Migrar a permisos `android.permissions.health.*` (ej. `READ_HEART_RATE`) |

Si el usuario solo pidió edge-to-edge, menciona brevemente los demás cambios como
checklist adicional, pero no los implementes salvo que los pida.

---

### 4. Configurar Edge-to-Edge

#### 4.1 Views (XML)

**Opción recomendada — `enableEdgeToEdge()`:**

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        enableEdgeToEdge() // antes de super.onCreate()
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
    }
}
```

Esto reemplaza ~100 líneas de compatibilidad manual (transparencia de barras, contraste,
iconos claros/oscuros según tema) y funciona en versiones anteriores a 36 también.

**Manejo manual (si no se puede usar la función anterior):**

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

**Aplicar insets para que el contenido no quede tapado por las barras:**

```kotlin
ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main)) { view, insets ->
    val bars = insets.getInsets(WindowInsetsCompat.Type.systemBars())
    view.updatePadding(left = bars.left, top = bars.top, right = bars.right, bottom = bars.bottom)
    insets
}
```

Usa `WindowInsetsCompat.Type.ime()` en vez de `systemBars()` para el teclado, y
`displayCutout()` para notch/cutouts. Aplica el inset al contenedor correcto (raíz o
solo a los elementos que realmente colisionan con la barra), no siempre a la raíz entera,
para no perder el efecto edge-to-edge (fondo detrás de las barras).

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

- `WindowInsets.safeDrawing` cubre barras de sistema + cutout; usa
  `WindowInsets.safeGestures` si necesitas también respetar zonas de gestos.
- Para IME, aplica `.imePadding()` o `WindowInsets.ime` en el composable que lo necesite
  (típicamente un campo de texto o un `Column` con scroll), no en toda la pantalla.
- Evita `Modifier.statusBarsPadding()` combinado con `Scaffold` insets al mismo tiempo:
  duplica el padding.

#### 4.3 Colores de iconos de las barras (claro/oscuro)

```kotlin
val controller = WindowCompat.getInsetsController(window, window.decorView)
controller.isAppearanceLightStatusBars = true   // iconos oscuros sobre fondo claro
controller.isAppearanceLightNavigationBars = true
```

`enableEdgeToEdge()` ya ajusta esto automáticamente según el tema si le pasas
`SystemBarStyle.auto(...)`; solo hazlo manual si necesitas un comportamiento distinto
al del tema del sistema.

---

### 5. Migrar Predictive Back (si aplica)

Requisito del manifest:

```xml
<application android:enableOnBackInvokedCallback="true">
```

**Views:**

```kotlin
onBackPressedDispatcher.addCallback(this) {
    // lógica de back personalizada
    if (shouldHandleCustomBack) {
        // manejar
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
        progress.collect { backEvent -> /* progreso de la animación */ }
        // gesto completado
    } catch (e: CancellationException) {
        // gesto cancelado por el usuario
    }
}
```

No dejes lógica crítica solo en `onBackPressed()`: en apps con `targetSdk 36` corriendo en
Android 16+, ese callback **no se invoca** durante las animaciones predictivas del sistema.

---

## Checklist de verificación

- [ ] `compileSdk`/`targetSdk = 36`, AGP y Gradle actualizados, proyecto compila sin errores
- [ ] `enableEdgeToEdge()` (o el setup manual equivalente) llamado en cada Activity relevante
- [ ] Ningún elemento crítico de la UI queda tapado por status bar, navigation bar, cutout o teclado
- [ ] Colores de iconos de barras correctos en tema claro y oscuro
- [ ] `android:windowOptOutEdgeToEdgeEnforcement` eliminado del proyecto
- [ ] Navegación "atrás" migrada a `OnBackPressedCallback` / `PredictiveBackHandler`, probada con el gesto predictivo real (no solo el botón atrás)
- [ ] Notificaciones full-screen (si existen) declaran `USE_FULL_SCREEN_INTENT`
- [ ] Layouts revisados en tablets/pantallas grandes (sw ≥ 600dp) sin depender de restricciones de orientación forzadas

**Probar en:** un dispositivo/emulador con Android 16 (API 36), en modo claro y oscuro,
con gestos de navegación activados, y en al menos una pantalla con contenido scrollable
y un campo de texto (para validar el inset del teclado).

---

## Notas

- Si el proyecto mezcla Views y Compose (interop), aplica insets en el punto de entrada
  de cada uno por separado — no asumas que el padding de Compose cubre las vistas XML embebidas o viceversa.
- Si el usuario pide "solo edge-to-edge" sin mencionar API 36, puedes aplicar la sección 4
  igual (es compatible hacia atrás), pero aclara que la aplicación **forzada** sin opt-out
  solo ocurre al llegar a `targetSdk 36`.
- Los detalles exactos de comportamiento pueden variar entre Developer Preview y la versión
  estable de Android 16; si el proyecto usa una preview, valida contra las release notes
  oficiales vigentes en el momento de la migración.
</content>

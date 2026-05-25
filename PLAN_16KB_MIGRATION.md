# Plan de Migración — Soporte 16 KB (Google Play)

**Proyecto:** `cordova-plugin-jitsi-meet-sdk`  
**Objetivo:** Cumplir con el requisito de Google Play de librerías nativas (`.so`) alineadas a páginas de 16 KB  
**Fecha del plan:** Mayo 2026  
**Versión actual del plugin:** `4.1.0`  
**Versión objetivo del plugin:** `5.0.0`

---

## 1. Diagnóstico Inicial

### 1.1 Estado actual del plugin

| Componente | Versión actual | Ubicación | ¿Compatible 16 KB? |
|---|---|---|---|
| Android SDK (Maven) | `org.jitsi.react:jitsi-meet-sdk:9.2.2` (Abr 2024) | `src/android/build-extras.gradle:14` | NO |
| iOS SDK (CocoaPods) | `JitsiMeetSDK` spec `11.2.4` (Jun 2025) | `plugin.xml:86` | NO |
| Android minSdkVersion | `24` | `src/android/build-extras.gradle:1` | NO (requiere 26+) |
| Repositorios Gradle | Incluye `jcenter()` (deprecado) | `src/android/build-extras.gradle:7` | [Deprecado] |

### 1.2 ¿Dónde están los archivos `.so`?

**No hay archivos `.so` directamente en el repositorio del plugin.** Las librerías nativas provienen del artefacto Maven `org.jitsi.react:jitsi-meet-sdk`. La versión actual (`9.2.2`) incluye la librería `duktape` que **no está alineada a 16 KB**, por lo que Google Play rechaza el AAB/APK.

```bash
# Búsqueda de .so en el proyecto — resultado: vacío
$ find . -name "*.so"
# (sin resultados)
```

### 1.3 Evidencia del fix en el changelog oficial

Fuente oficial: [CHANGELOG-MOBILE-SDKS.md](https://github.com/jitsi/jitsi-meet-release-notes/blob/master/CHANGELOG-MOBILE-SDKS.md)

> **Versión 11.5.1 (2025-09-16) — Android:**
> *"Replaced duktape lib with react-native-worklets-core lib to align with Android 16kb page size requirement."*

**Conclusión**: Cualquier versión del SDK **>= 11.5.1** incluye el fix. La más reciente es **12.1.3** (Mayo 2026).

### 1.4 Disponibilidad en Maven

El repositorio Maven de Jitsi contiene todas las versiones hasta 12.1.3:
- URL: `https://github.com/jitsi/jitsi-maven-repository/tree/master/releases/org/jitsi/react/jitsi-meet-sdk`
- Coordenadas: **NO han cambiado** — `org.jitsi.react:jitsi-meet-sdk:<VERSION>`

---

## 2. Cambios Realizados

### 2.1 Tabla resumen de archivos modificados

| # | Archivo | Línea(s) | Cambio | Tipo |
|---|---------|----------|--------|------|
| 1 | `src/android/build-extras.gradle` | 1 | `cdvMinSdkVersion = 24` → `26` | **Requerido** |
| 2 | `src/android/build-extras.gradle` | ~7 | Eliminar `jcenter()`, agregar `mavenCentral()` | **Requerido** |
| 3 | `src/android/build-extras.gradle` | ~14 | `jitsi-meet-sdk:9.2.2` → `jitsi-meet-sdk:12.1.3` | **Requerido** |
| 4 | `plugin.xml` | ~86 | `spec="11.2.4"` → `spec="12.1.3"` | **Requerido** |
| 5 | `package.json` | 3 | `"version": "4.1.0"` → `"version": "5.0.0"` | Recomendado |

### 2.2 Archivos SIN cambios

| Archivo | Razón |
|---------|-------|
| `src/android/JitsiMeet.java` | Las APIs usadas (`JitsiMeetActivity.launch`, `JitsiMeetConferenceOptions.Builder`, `JitsiMeetUserInfo`, `BroadcastEvent.Type.*`) se mantienen compatibles según la documentación oficial del SDK |
| `src/ios/CDVJitsiMeet.h` | Sin cambios necesarios |
| `src/ios/CDVJitsiMeet.m` | Sin cambios necesarios |
| `www/jitsi-meet.js` | Sin cambios necesarios |

### 2.3 `build-extras.gradle` — ANTES

```groovy
ext.cdvMinSdkVersion = 24

android {
    repositories {
        google()
        jcenter()
        maven {
            url "https://github.com/jitsi/jitsi-maven-repository/raw/master/releases"
        }
         maven { url 'https://www.jitpack.io' }
    }

    dependencies {
        implementation ('org.jitsi.react:jitsi-meet-sdk:9.2.2') { transitive = true }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

### 2.4 `build-extras.gradle` — DESPUÉS

```groovy
ext.cdvMinSdkVersion = 26

android {
    repositories {
        google()
        mavenCentral()
        maven {
            url "https://github.com/jitsi/jitsi-maven-repository/raw/master/releases"
        }
        maven { url 'https://www.jitpack.io' }
    }

    dependencies {
        implementation ('org.jitsi.react:jitsi-meet-sdk:12.1.3') { transitive = true }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```

**Justificación de cada cambio:**
- `minSdkVersion 24 → 26`: El SDK 11.0.0 (Feb 2025) subió el mínimo a Android 8.0 Oreo (API 26). Ver changelog: *"Bump minimum required version to 26 aka Oreo"*
- `jcenter() → mavenCentral()`: JFrog cerró jcenter en 2024. Todas las dependencias ya están disponibles en Maven Central
- `9.2.2 → 12.1.3`: Incluye el fix de 16 KB (11.5.1) + todas las mejoras, fixes y actualizaciones acumuladas hasta Mayo 2026

---

## 3. Plan de Validación

### 3.1 Validación de compilación

| # | Validación | Comando / Procedimiento | Criterio de éxito |
|---|-----------|------------------------|-------------------|
| V1 | Build Android | `cordova build android` | Compila sin errores |
| V2 | Dependencias resueltas | Revisar que no haya `Could not find org.jitsi.react:jitsi-meet-sdk:12.1.3` | Sin errores de dependencias |
| V3 | Compilación Java del plugin | Verificar que `JitsiMeet.java` compila con las nuevas APIs | Sin errores de compilación |
| V4 | Alineación 16 KB en APK | `unzip -l app-debug.apk \| grep "\.so$"` o Android Studio > Build > Analyze APK | `.so` alineados a 16 KB (16384 bytes) |
| V5 | Build iOS | `cordova build ios` | Compila sin errores |
| V6 | Resolución CocoaPods | `pod install` (en platforms/ios) resuelve `JitsiMeetSDK 12.1.3` | Sin errores de resolución de pods |

### 3.2 Validación funcional (ambas plataformas)

| # | Validación | Procedimiento | Criterio de éxito |
|---|-----------|--------------|-------------------|
| V7 | Unirse a conferencia | `JitsiMeet.startConference({ room: "test" })` | Se une correctamente |
| V8 | Eventos broadcast | Verificar que `CONFERENCE_JOINED`, `CONFERENCE_TERMINATED`, etc. se reciben en JS | Todos los eventos funcionan |
| V9 | Audio y Video | Verificar micrófono y cámara en conferencia | Sin errores |
| V10 | Cerrar conferencia | `JitsiMeet.disposeConference(...)` | La vista se cierra sin errores |
| V11 | Google Play Console | Subir AAB a Internal Testing en Google Play | **Sin warning de 16 KB** |

### 3.3 Cómo verificar la alineación 16 KB (V4)

```bash
# Opción A: Línea de comandos
unzip -l platforms/android/app/build/outputs/apk/debug/app-debug.apk | grep "\.so$"

# Opción B: Android Studio
# Build > Analyze APK > Seleccionar el APK
# Revisar lib/arm64-v8a/*.so — deben estar alineados a múltiplos de 16384 bytes

# Opción C: Herramienta de Google
# Usar el Android App Bundle Explorer en Google Play Console
# Verificar la sección "16 KB page size" — debe mostrar "Compatible"
```

### 3.4 Cómo verificar eventos broadcast en iOS (V8)

```javascript
JitsiMeet.startConference({ room: "test" }, function(listener) {
    console.log("Evento recibido:", listener);
    // Debe imprimir: CONFERENCE_WILL_JOIN, CONFERENCE_JOINED, etc.
});
```

---

## 4. Gestión de Riesgos

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|-------------|---------|------------|
| `LocalBroadcastManager` removido en SDK 12.x | Baja | Alto (rompe build Android) | Migrar a `LiveData` o `Flow` si falla la compilación |
| Cambios en API de `JitsiMeetConferenceOptions.Builder` | Baja | Alto (rompe build) | Revisar javadoc del SDK y ajustar métodos si es necesario |
| Conflictos de dependencias transitivas | Media | Medio | Usar `./gradlew app:dependencies` en el proyecto Cordova para resolver |
| iOS 12.1.3 no disponible en CocoaPods trunk | Muy baja | Medio | Verificar con `pod search JitsiMeetSDK` o `pod repo update` |

---

## 5. Breaking Changes del SDK (9.2.2 → 12.1.3)

De la revisión del [CHANGELOG-MOBILE-SDKS.md](https://github.com/jitsi/jitsi-meet-release-notes/blob/master/CHANGELOG-MOBILE-SDKS.md):

| Versión | Cambio relevante | Impacto en el plugin |
|---------|-----------------|---------------------|
| 10.0.0 | Unificación de versiones Android/iOS/RN/Flutter | Ninguno (solo naming) |
| 10.0.0 | WebRTC 124, React Native 0.73 | Ninguno (transparente para el plugin) |
| 10.0.0 | WebSockets para XMPP por defecto | Ninguno |
| 11.0.0 | **minSdkVersion sube a 26** | Ajustado en `build-extras.gradle` |
| 11.0.0 | Hermes engine habilitado (JSC removido) | Puede mejorar performance |
| 11.0.0 | React Native 0.75.5 | Ninguno |
| 11.4.0 | Build target API 35 | El proyecto consumidor debe tener `compileSdkVersion = 35` |
| **11.5.1** | **duktape → react-native-worklets-core (fix 16 KB)** | **ESTE ES EL FIX PRINCIPAL** |
| 12.0.0 | React 19 / React Native 0.79 | Ninguno |
| 12.1.1 | React Native 0.79.7 | Ninguno |

---

## 6. Requisitos del Proyecto Cordova Consumidor

Estos requisitos aplican al proyecto que **usa** este plugin (no al plugin mismo):

| Requisito | Valor mínimo | Cómo verificarlo |
|-----------|-------------|-----------------|
| `cordova-android` | >= 12.0.0 | `cordova platform ls` |
| `compileSdkVersion` | 35 | Revisar `platforms/android/build.gradle` o `config.xml` |
| `targetSdkVersion` | 35 | Revisar `platforms/android/build.gradle` o `config.xml` |
| `cordova CLI` | >= 12.0.0 | `cordova --version` |
| Notification icon | Obligatorio | Ver sección "Warning" en README.md |

> **Nota:** `cordova-android 15.0.0` ya incluye AGP 8.x y NDK r27+, que son los requisitos de toolchain para empaquetar APK/AAB con soporte 16 KB.

### Configuración adicional recomendada en `gradle.properties`

```properties
# En platforms/android/gradle.properties del proyecto Cordova
android.bundle.enableUncompressedNativeLibs=false
```

---

## 7. Orden de Ejecución

1. [x] Modificar `src/android/build-extras.gradle` (minSdk, jcenter, versión SDK)
2. [x] Modificar `plugin.xml` (iOS podspec)
3. [x] Modificar `package.json` (bump versión a 5.0.0)
4. [ ] **Validación V1:** `cordova build android`
5. [ ] **Validación V5:** `cordova build ios`
6. [ ] **Validación V4:** Verificar alineación 16 KB en el APK
7. [ ] **Validaciones funcionales V7-V10:** Probar conferencia, audio, video, eventos
8. [ ] **Validación V11:** Subir a Google Play Console Internal Testing — verificar que NO aparece el warning de 16 KB
9. [ ] Publicar release en GitHub con tag `v5.0.0`

---

## 8. Referencias

- [Jitsi Mobile SDK Changelog](https://github.com/jitsi/jitsi-meet-release-notes/blob/master/CHANGELOG-MOBILE-SDKS.md)
- [Jitsi Maven Repository](https://github.com/jitsi/jitsi-maven-repository)
- [Jitsi Android SDK Documentation](https://jitsi.github.io/handbook/docs/dev-guide/dev-guide-android-sdk)
- [Jitsi iOS SDK Documentation](https://jitsi.github.io/handbook/docs/dev-guide/dev-guide-ios-sdk)
- [Google Play 16 KB page size requirement](https://developer.android.com/guide/practices/page-sizes)
- [AGP 16 KB alignment documentation](https://developer.android.com/build/releases/past-releases/agp-8-0-0-release-notes#16-kb)

---

## 9. Notas Finales

- Los archivos `.so` **NO** están en este repositorio. Provienen del artefacto Maven. Al actualizar la dependencia a `12.1.3`, se obtienen automáticamente las versiones alineadas a 16 KB.
- Si surge un error de compilación relacionado con `LocalBroadcastManager`, será necesario migrar los `BroadcastReceiver` en `JitsiMeet.java` a otro mecanismo (LiveData, Flow, etc.).
- La versión `12.1.3` fue seleccionada por ser la más reciente disponible (Mayo 2026) e incluye todos los fixes acumulados desde `11.5.1` (la mínima con soporte 16 KB).

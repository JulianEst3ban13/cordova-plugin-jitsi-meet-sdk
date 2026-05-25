# cordova-plugin-jitsi-meet-sdk

Cordova plugin for Jitsi Meet SDK with **16 KB page size support** (Google Play compliant).

> **SDK Versions:** Android `12.1.3` | iOS `12.1.3`  
> **Plugin Version:** `5.0.0`

---

## Google Play 16 KB Page Size

Desde Agosto 2025, Google Play **requiere** que todas las apps soporten páginas de memoria de 16 KB. Esto significa que las librerías nativas (`.so`) deben estar alineadas a 16 KB. La versión anterior de este plugin (`4.x`) usaba el SDK `9.2.2` que incluía `duktape`, una librería **no compatible con 16 KB**.

Jitsi solucionó este problema en la versión **11.5.1** del SDK (Septiembre 2025), reemplazando `duktape` por `react-native-worklets-core`. Este plugin usa la versión **12.1.3** (Mayo 2026), que incluye ese fix y todas las mejoras acumuladas.

> **Info:** [Plan detallado de migración a 16 KB](./PLAN_16KB_MIGRATION.md)  
> **Referencia:** [CHANGELOG-MOBILE-SDKS.md](https://github.com/jitsi/jitsi-meet-release-notes/blob/master/CHANGELOG-MOBILE-SDKS.md)

---

## Requisitos

| Requisito | Mínimo | Notas |
|---|---|---|
| **cordova CLI** | >= 12.0.0 | |
| **cordova-android** | >= 12.0.0 | Requerido para AGP 8.x y soporte 16 KB |
| **cordova-ios** | >= 7.0.0 | |
| **Android** | >= 8.0 (API 26) | El SDK 11.0.0+ requiere API 26 |
| **iOS** | >= 15.1 | |

---

## Instalación

```bash
cordova plugin add https://github.com/vlad909/cordova-plugin-jitsi-meet-sdk.git
```

### Configuración obligatoria para Android

Agrega el ícono de notificación en tu `config.xml` (requerido por el SDK):

```xml
<platform name="android">
    <resource-file src="res/android/ic_notification.png" target="app/src/main/res/drawable-ldpi/ic_notification.png" />
    <resource-file src="res/android/ic_notification.png" target="app/src/main/res/drawable-mdpi/ic_notification.png" />
    <resource-file src="res/android/ic_notification.png" target="app/src/main/res/drawable-hdpi/ic_notification.png" />
    <resource-file src="res/android/ic_notification.png" target="app/src/main/res/drawable-xhdpi/ic_notification.png" />
</platform>
```

Sin estas líneas el plugin no iniciará correctamente.

---

## Uso

Todos los parámetros son opcionales excepto `room`.

- Si `serverURL` no se especifica, por defecto es `https://meet.jit.si`
- Los feature flags no especificados usan su valor por defecto
- Las opciones booleanas por defecto son `false`

### Ejemplo mínimo

```js
JitsiMeet.startConference({
    room: "MyAmazingRoom"
});
```

### Ejemplo completo

```js
JitsiMeet.startConference({
    serverURL: "https://meet.jit.si",
    room: "MyAmazingRoom",
    displayName: "Max!",
    email: "max@amazingmax.it",
    audioMuted: false,
    videoMuted: false,
    subject: "My amazing name",
    audioOnly: false,
    //token: "your jwt token, if you have one",
    flags: {
        "chat.enabled": true,
        "invite.enabled": true,
        "kick-out.enabled": true,
        "live-streaming.enabled": true,
        "pip.enabled": true,
        "raise-hand.enabled": true,
        "recording.enabled": true,
        "video-share.enabled": true,
        "add-people.enabled": true,
        "calendar.enabled": true,
        "meeting-name.enabled": true,
        "meeting-password.enabled": true,
        "toolbox.alwaysVisible": true,
        "welcomepage.enabled": true,
        "prejoinpage.enabled": true
    }
}, function(listener) {
    console.log(listener);

    switch(listener) {
        case "CONFERENCE_JOINED":
            // Conferencia iniciada
            break;
        case "CONFERENCE_TERMINATED":
            // Conferencia terminada
            break;
        case "CONFERENCE_WILL_JOIN":
            // A punto de unirse
            break;
        case "AUDIO_MUTED_CHANGED":
            // Cambio de estado de audio
            break;
        case "VIDEO_MUTED_CHANGED":
            // Cambio de estado de video
            break;
        case "PARTICIPANT_JOINED":
            // Participante se unió
            break;
        case "PARTICIPANT_LEFT":
            // Participante salió
            break;
        case "ENDPOINT_TEXT_MESSAGE_RECEIVED":
            // Mensaje de texto recibido
            break;
        case "PARTICIPANTS_INFO_RETRIEVED":
            // Información de participantes
            break;
        case "CHAT_MESSAGE_RECEIVED":
            // Mensaje de chat recibido
            break;
        case "CHAT_TOGGLED":
            // Chat abierto/cerrado
            break;
    }
});
```

---

## Cerrar conferencia

```js
JitsiMeet.disposeConference(function(success) {
    console.log("Conferencia cerrada correctamente");
}, function(error) {
    console.error(error);
});
```

---

## Eventos soportados

### CONFERENCE_JOINED
Se emite cuando se une a una conferencia.

### CONFERENCE_TERMINATED
Se emite cuando la conferencia termina (por el usuario o por error).

### CONFERENCE_WILL_JOIN
Se emite antes de unirse a la conferencia.

### AUDIO_MUTED_CHANGED
Se emite cuando cambia el estado de mute del participante local.

### VIDEO_MUTED_CHANGED
Se emite cuando cambia el estado de video del participante local.

### PARTICIPANT_JOINED
Se emite cuando un participante se une a la conferencia.

### PARTICIPANT_LEFT
Se emite cuando un participante abandona la conferencia.

### ENDPOINT_TEXT_MESSAGE_RECEIVED
Se emite cuando se recibe un mensaje de texto.

### PARTICIPANTS_INFO_RETRIEVED
Se emite cuando se solicita información de participantes.

### CHAT_MESSAGE_RECEIVED
Se emite cuando se recibe un mensaje de chat.

### CHAT_TOGGLED
Se emite cuando el chat se abre o se cierra.

---

## Broadcasting Actions

_En desarrollo..._

---

## Migración desde v4.x a v5.0.0

Si vienes de la versión anterior del plugin, estos son los cambios:

### Cambios en el plugin

| Aspecto | v4.x | v5.0.0 |
|---------|------|--------|
| Android SDK | `9.2.2` | `12.1.3` |
| iOS SDK | `11.2.4` | `12.1.3` |
| Android mínimo | API 24 | API 26 |
| Soporte 16 KB | No | Si |

### Cambios en tu proyecto Cordova

| Requisito | v4.x | v5.0.0 |
|-----------|------|--------|
| cordova-android | >= 11 | **>= 12** |
| cordova CLI | >= 10 | **>= 12** |
| compileSdkVersion | 32 | **35** |
| targetSdkVersion | 32 | **35** |

### API del plugin

La API JavaScript (`startConference`, `disposeConference`, eventos) **no cambió**. Tu código existente debería funcionar sin modificaciones.

---

## Feature Flags

Todos los feature flags disponibles:  
https://github.com/jitsi/jitsi-meet/blob/master/react/features/base/flags/constants.js

---

## Issues

Reporta problemas en: https://github.com/vlad909/cordova-plugin-jitsi-meet-sdk/issues

---

## Créditos

- Plugin original por [Massimiliano Coppola](https://github.com/Zeno97)
- Mantenido por [@vlad909](https://github.com/vlad909)
- Email: maxcoppola97@libero.it

---
description: Prepara el entorno para testear una app Expo/RN con Maestro (binario, emulador, app, MCP)
argument-hint: [plataforma: ios|android]
---

El usuario quiere preparar el entorno de testing Maestro para una app Expo/React Native.
Plataforma objetivo (si la indicó): $ARGUMENTS

Seguí estos pasos y reportá el estado de cada uno:

1. **Verificar Maestro instalado.** Corré `maestro -v`.
   - Si falla, indicá al usuario instalar Maestro (https://docs.maestro.dev) y reinstalar el
     plugin para que el MCP `maestro` quede disponible. No continúes hasta tenerlo.

2. **Verificar el MCP.** Llamá al tool `list_devices` del Maestro MCP.
   - Si el tool no existe, el plugin/MCP no está cargado: pedí revisar la instalación del plugin.

3. **Asegurar un device.** Si `list_devices` no devuelve ninguno, arrancá uno con
   `maestro start-device --platform <ios|android>` (usá la plataforma de $ARGUMENTS o preguntá).

4. **Asegurar la app Expo corriendo.** Confirmá con el usuario cómo arranca su app
   (p. ej. `npx expo run:android` / `npx expo run:ios`, o un dev client ya instalado).
   Si no está corriendo, ayudá a lanzarla en el device.

5. **Confirmar listo.** Volvé a `list_devices` y reportá: device activo + app en pantalla.
   Indicá que ya puede usar `/rn-test`.

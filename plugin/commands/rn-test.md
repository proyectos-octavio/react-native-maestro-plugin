---
description: Sesión de testing E2E dirigida por IA en una app Expo/RN; guarda un flow Maestro reproducible
argument-hint: [descripción del escenario a probar]
---

El usuario quiere que pruebes este escenario en su app Expo/React Native: $ARGUMENTS

Usá la skill `maestro-rn-testing`. Procedimiento:

1. Verificá prerrequisitos (device + app corriendo). Si falta algo, sugerí `/rn-setup`.
2. `list_devices` y elegí el device.
3. `inspect_screen` para percibir la pantalla; `take_screenshot` solo si necesitás desambiguar.
4. Ejecutá el escenario con `run({ yaml })`, un paso a la vez, re-percibiendo tras cada cambio
   de pantalla. Usá selectores por `id`/`testID` y `assertVisible` (nunca `sleep` fijos).
5. Al validar el escenario, guardá el flow consolidado en
   `.maestro/<kebab-case-del-escenario>.yaml` (reproducible con `maestro test`).
6. Reportá: pasos ejecutados, hallazgos/bugs encontrados, y la ruta del flow guardado.

Si la descripción está vacía, pedí al usuario qué escenario probar antes de empezar.

---
name: maestro-rn-testing
description: Use the moment the user wants to test, QA, explore, find bugs in, drive, or generate end-to-end (E2E) tests for an Expo / React Native app — phrasings like "testear la app", "probá el login", "encontrá bugs", "generá un test E2E", "explora la pantalla", "test this flow". Drives the running app on a native emulator/simulator via the Maestro MCP server (tools prefixed `maestro`) and saves a reproducible Maestro flow (YAML) to `.maestro/`. Does NOT apply to unit/component tests, pure non-Expo web apps, or static code review.
---

# Testing de apps Expo / React Native con el Maestro MCP

Conducís una app Expo/RN que ya corre en un emulador/simulador usando los tools del
**Maestro MCP** y dejás un **flow Maestro reproducible** en `.maestro/`.

## Cuándo aplica

- El usuario quiere **probar, explorar, encontrar bugs o generar un E2E** de una app Expo/RN.
- Hay (o se puede arrancar) un emulador/simulador con la app corriendo.

## Cuándo NO aplica

- Tests unitarios o de componentes (eso es Jest/Vitest/RNTL, no Maestro).
- Apps web puras que no son Expo/RN Web.
- Revisión estática de código sin ejecutar la app.

## Prerrequisitos (verificá antes de accionar)

1. **Binario `maestro` instalado.** Si los tools `maestro` del MCP no responden, el usuario
   debe instalar Maestro y reinstalar el plugin. Sugerí correr `/rn-setup`.
2. **Un device disponible.** Llamá `list_devices`. Si está vacío, detené y pedí arrancar un
   emulador (sugerí `/rn-setup`). NO inventes un device.
3. **La app corriendo en el device.** Si la pantalla no es la app esperada, pedí arrancar la
   app Expo (`/rn-setup` lo cubre).

## Procedimiento

1. **Descubrir devices:** `list_devices`. Elegí el que indique el usuario o el único disponible.
2. **Percibir la pantalla actual:** `inspect_screen` (jerarquía JSON). Usá `take_screenshot`
   solo si necesitás desambiguar visualmente.
3. **Accionar:** ejecutá pasos con `run({ yaml })` (YAML inline, preferido para explorar).
   Un paso a la vez para escenarios nuevos; agrupá cuando ya estén validados.
4. **Re-percibir después de cada cambio de pantalla:** volvé a `inspect_screen`. NUNCA asumas
   el árbol siguiente.
5. **Verificar el escenario:** usá `assertVisible` / `assertNotVisible` en el YAML para fijar
   las condiciones de éxito.
6. **Guardar el flow reproducible:** escribí el flow consolidado en
   `.maestro/<kebab-case-del-escenario>.yaml`. Debe correr solo con `maestro test` sin IA.
7. **Reportar:** resumen de pasos, hallazgos/bugs y la ruta del flow guardado.

## Convenciones Expo / React Native

- **Selectores:** preferí `id` (mapea a `testID` en RN) sobre texto. El texto cambia con i18n
  y es frágil.
- **`testID` → accesibilidad:** en RN, `testID` se expone a Maestro como el `id` del elemento.
  Si un elemento no tiene `testID`, sugerí agregarlo en el código antes que depender del texto.
- **Navegación:** tras `tapOn`, esperá la transición con `assertVisible` del destino, no con
  `sleep` fijos.
- **Deep links:** usá `openLink` para entrar directo a una ruta Expo Router cuando aplique.
- **Estado limpio:** empezá los flows con `launchApp` (con `clearState: true` si el escenario
  requiere arrancar desde cero).

## Anatomía de un flow guardado

```yaml
appId: com.tuempresa.tuapp
---
- launchApp
- assertVisible: "Iniciar sesión"
- tapOn:
    id: "email-input"
- inputText: "user@example.com"
- tapOn:
    id: "login-button"
- assertVisible:
    id: "home-screen"
```

## Red flags — NO hagas esto

| Racionalización | Qué hacer en realidad |
|---|---|
| "Espero con un `sleep` 3s a que cargue." | Usá `assertVisible` del elemento destino; `sleep` fijo = flaky. |
| "Selecciono por el texto visible." | Preferí `id`/`testID`; el texto cambia con i18n. |
| "Asumo que tras el tap aparece la home." | Re-llamá `inspect_screen` y verificá; no asumas el árbol. |
| "Exploré y ya está, no guardo nada." | El objetivo es un flow reproducible en `.maestro/`. Siempre guardalo. |
| "El selector no aparece, lo invento." | Reportá que no existe y sugerí agregar `testID` en el código. |

## Salida esperada

- Un archivo `.maestro/<escenario>.yaml` reproducible.
- Un resumen: pasos ejecutados, hallazgos/bugs, y cómo re-ejecutar (`maestro test .maestro/<escenario>.yaml`).

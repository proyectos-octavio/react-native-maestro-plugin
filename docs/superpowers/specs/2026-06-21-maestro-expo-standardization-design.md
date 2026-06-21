# Estandarización de Maestro para proyectos Expo / React Native

**Fecha:** 2026-06-21
**Estado:** Aprobado para planificación
**Alcance:** Android (local). CI explícitamente fuera de alcance.

## Problema

El lado "tooling de Claude" para testing E2E de apps Expo/RN ya está cubierto por el
plugin `react-native-maestro-plugin` (Maestro MCP + skill `maestro-rn-testing` + comandos
`/rn-setup` y `/rn-test`). Lo que falta es estandarizar **qué entra dentro de cada proyecto
Expo/RN nuevo** para que Maestro quede configurado igual siempre: estructura de carpetas,
scripts, flow de arranque y la convención documentada que Claude consulte.

## Objetivo

Una única convención por-proyecto como fuente de verdad, aplicada de tres formas:
el scaffold la genera, una skill la documenta, y el README del plugin la enlaza.

## Decisiones tomadas

- **Plataforma:** Android, uso local. (iOS y CI quedan fuera de este alcance.)
- **Activación en el scaffold:** opt-in vía prompt `maestroSetup`, default `true`.
- **CI:** descartada de este plan de ejecución.
- **Maestro:** se asume el CLI `maestro` instalado en el entorno del desarrollador (no es
  dependencia npm; los scripts solo lo invocan).

## Componentes

### 1. Fuente de verdad — convención por-proyecto

Todo proyecto Expo generado con `maestroSetup == true` incluye:

```
.maestro/
  config.yaml      # flows = .maestro/, tags por defecto
  smoke.yaml       # flow de arranque: launchApp + assertVisible del home
  README.md        # cómo correr local + convención de testID
package.json       # scripts: test:e2e, test:e2e:studio
```

Convenciones (coherentes con la skill `maestro-rn-testing` ya existente):

- **Selectores por `id`** (que mapea a `testID` en RN), en kebab-case (`login-button`).
  Nunca por texto visible (frágil ante i18n).
- **Flows planos** en `.maestro/*.yaml`, igual que lo que guardan `/rn-test` y la skill.
- **`assertVisible` del destino** tras navegar, nunca `sleep` fijos.
- `smoke.yaml` arranca con `launchApp` y verifica un elemento del home por `id`.
- El `appId` de los flows se templatiza desde el bundle id del proyecto (app.json).

Scripts en `package.json`:

- `test:e2e` → `maestro test .maestro`
- `test:e2e:studio` → `maestro studio`

`.maestro/README.md` documenta: prerrequisitos (CLI `maestro` + device/app corriendo,
sugiere `/rn-setup`), cómo correr `npm run test:e2e`, la convención de `testID`, y cómo
generar flows nuevos asistidos (`/rn-test`).

### 2. Integración al scaffold (`scaffold-generator`, template `expo`)

Cambios en `src/templates/expo/template.json`:

- Nuevo prompt: `{ "key": "maestroSetup", "type": "confirm", "label": "¿Incluir setup de testing E2E con Maestro?", "default": true }`.
- Nuevo grupo de archivos: `{ "glob": "files/maestro/**/*", "strip": "files/maestro/", "when": "maestroSetup == true" }`.
- Nuevo postAction mergeJson:
  `{ "type": "mergeJson", "target": "package.json", "source": "fragments/maestro.scripts.json", "when": "maestroSetup == true" }`.
- `nextSteps`: agregar una línea indicando que `npm run test:e2e` requiere el CLI `maestro`
  y un device con la app corriendo.
- Actualizar la `description` del template para mencionar el setup Maestro opcional.

Archivos nuevos en el template:

- `src/templates/expo/files/maestro/.maestro/config.yaml`
- `src/templates/expo/files/maestro/.maestro/smoke.yaml` (usa el placeholder de appId)
- `src/templates/expo/files/maestro/.maestro/README.md`
- `src/templates/expo/fragments/maestro.scripts.json` (solo el objeto `scripts`)

### 3. Skill de convenciones (`standards`)

Nueva skill `conventions-maestro` (mismo estilo conciso que las otras `conventions-*`),
en `skills/conventions-maestro/SKILL.md`. Contenido:

- Maestro es el estándar de testing E2E para apps Expo/RN propias.
- Estructura `.maestro/` (config + flows planos + README).
- Regla de selectores: `id`/`testID`, kebab-case, nunca texto.
- `assertVisible` sobre `sleep`.
- Scripts `test:e2e` / `test:e2e:studio`.
- Referencias: plugin `react-native-maestro-plugin` (`/rn-setup`, `/rn-test`) para sesiones
  asistidas, y skill `reusable-assets` antes de construir nada a mano.

Tareas de registro:

- Bump de versión en el `plugin.json` del plugin `standards`.
- Entrada/actualización en el marketplace (`marketplace/`) si lista skills explícitamente.

### 4. Este repo (plugin Maestro)

- Agregar al `README.md` una sección "Convención por-proyecto" que explique que el setup
  dentro de cada app (carpeta `.maestro/`, scripts, smoke flow) lo genera el template `expo`
  de `@octavius2929-personal/scaffold-generator` con la opción `maestroSetup`, cerrando el
  círculo entre el tooling de Claude y el setup del proyecto.

## Fuera de alcance

- Workflow de CI (GitHub Actions / Maestro Cloud).
- iOS (selección de device, build, runners macOS).
- Cambios en la skill `maestro-rn-testing` o en los comandos `/rn-setup` y `/rn-test`
  (ya cubren la sesión asistida; solo se referencian).

## Criterios de éxito

- Generar un proyecto con `--set maestroSetup=true` produce `.maestro/` con `config.yaml`,
  `smoke.yaml` y `README.md`, y `package.json` con los scripts `test:e2e` y `test:e2e:studio`.
- Generar con `maestroSetup=false` no incluye ninguno de esos archivos ni scripts.
- La skill `conventions-maestro` aparece disponible y describe la convención.
- El README del plugin enlaza la convención por-proyecto.

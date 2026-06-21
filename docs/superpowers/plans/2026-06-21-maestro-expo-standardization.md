# Estandarización de Maestro para proyectos Expo/RN — Plan de implementación

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Estandarizar el setup de testing E2E con Maestro dentro de cada app Expo/RN nueva: el scaffold lo genera (opt-in), una skill lo documenta, y el README del plugin lo enlaza.

**Architecture:** Una única convención por-proyecto (`.maestro/` + scripts) embebida como grupo de archivos opt-in en el template `expo` del `scaffold-generator`. La skill `conventions-maestro` (repo `standards`) la documenta. El README de este repo (`react-native-playwright`) la enlaza para cerrar el círculo tooling↔proyecto.

**Tech Stack:** scaffold-generator (Node + TypeScript, DDD/hexagonal, Vitest, manifest JSON de templates), Maestro CLI, Claude Code plugins/skills (Markdown + frontmatter).

## Global Constraints

- **Plataforma:** Android, uso local. iOS y CI están **fuera de alcance** (no crear workflows ni configs de CI).
- **Maestro es CLI, no dependencia npm:** los scripts solo invocan el binario `maestro`; no agregar `maestro` a `dependencies`/`devDependencies`.
- **Activación opt-in:** el setup se controla con el prompt `maestroSetup` (confirm, default `true`).
- **Repos afectados** (hermanos locales bajo `D:\personal\projects\libraries\@octavius2929-personal\`):
  - `scaffold-generator` (Tarea 1)
  - `standards` (Tarea 2)
  - `react-native-playwright` (Tarea 3)
- **Commits:** Conventional Commits. Cada repo tiene su propio historial git; commitear en el repo correspondiente.
- **Versionado fuera de alcance:** ver "Notas de publicación" al final. NO bumpear versiones como parte de las tareas (hay una desincronización preexistente en `scaffold-generator`).
- **Convención de selectores (verbatim del spec):** selectores por `id` (= `testID` en RN), kebab-case; `assertVisible` del destino, nunca `sleep`.

---

## Estructura de archivos

**Repo `scaffold-generator`:**
- Crear `src/templates/expo/files/maestro/.maestro/config.yaml` — config del workspace Maestro.
- Crear `src/templates/expo/files/maestro/.maestro/smoke.yaml` — flow de humo de arranque.
- Crear `src/templates/expo/files/maestro/.maestro/README.md` — doc por-proyecto (cómo correr + convención testID).
- Crear `src/templates/expo/fragments/maestro.scripts.json` — fragment con los scripts npm.
- Modificar `src/templates/expo/template.json` — prompt, file group, mergeJson postAction, nextSteps, description.
- Modificar `tests/e2e/expo-template.e2e.test.ts` — cobertura de `maestroSetup` true/false + prompts.

**Repo `standards`:**
- Crear `plugin/skills/conventions-maestro/SKILL.md` — skill de convenciones.

**Repo `react-native-playwright`:**
- Modificar `README.md` — sección "Convención por-proyecto".

---

## Task 1: Opción Maestro en el template `expo` (scaffold-generator)

**Repo:** `scaffold-generator`

**Files:**
- Create: `src/templates/expo/files/maestro/.maestro/config.yaml`
- Create: `src/templates/expo/files/maestro/.maestro/smoke.yaml`
- Create: `src/templates/expo/files/maestro/.maestro/README.md`
- Create: `src/templates/expo/fragments/maestro.scripts.json`
- Modify: `src/templates/expo/template.json`
- Test: `tests/e2e/expo-template.e2e.test.ts`

**Interfaces:**
- Consumes: el motor de templates existente — prompts `confirm`, `files.include[]` con `{glob, strip, when}`, postAction `mergeJson` (deep-merge de la key `scripts`), placeholder `{{projectName}}`, condiciones `key == true`.
- Produces: nuevo prompt `maestroSetup` (confirm, default true); cuando es `true`, el proyecto generado contiene `.maestro/config.yaml`, `.maestro/smoke.yaml`, `.maestro/README.md` y `package.json` con scripts `test:e2e` (`maestro test .maestro`) y `test:e2e:studio` (`maestro studio`).

> Nota de diseño (reconciliación con el spec): el spec mencionaba "templatizar el appId desde app.json". Verificado en el código: `app.json` del template **no** define `android.package`/`ios.bundleIdentifier` y el renderer no tiene transforms (un `{{projectName}}` con guiones produce un appId/package inválido). Por eso `smoke.yaml` usa un placeholder explícito (`com.example.app`) documentado para que el usuario lo reemplace, en vez de templatizar un valor inválido. No cambia el resto del diseño.

- [ ] **Step 1: Escribir las aserciones que fallan (test maestroSetup=true)**

En `tests/e2e/expo-template.e2e.test.ts`, en el primer test (`scaffolds full app ...`), agregar el flag `maestroSetup=true` al `execa` (después del bloque de `claudeSetup=true`):

```ts
        "--set",
        "claudeSetup=true",
        "--set",
        "maestroSetup=true",
```

Agregar al array de archivos esperados (`for (const f of [ ... ])`) estas tres entradas:

```ts
      ".maestro/config.yaml",
      ".maestro/smoke.yaml",
      ".maestro/README.md",
```

Extender el cast de `pkg` para incluir `scripts` y agregar las aserciones de scripts. Cambiar:

```ts
    const pkg = JSON.parse(await readFile(path.join(project, "package.json"), "utf8")) as {
      name: string;
      main: string;
      dependencies: Record<string, string>;
    };
```

por:

```ts
    const pkg = JSON.parse(await readFile(path.join(project, "package.json"), "utf8")) as {
      name: string;
      main: string;
      dependencies: Record<string, string>;
      scripts: Record<string, string>;
    };
```

y al final de ese test (después del último `expect(screen)...`) agregar:

```ts
    expect(pkg.scripts["test:e2e"]).toBe("maestro test .maestro");
    expect(pkg.scripts["test:e2e:studio"]).toBe("maestro studio");
```

- [ ] **Step 2: Escribir las aserciones que fallan (test maestroSetup=false)**

En el segundo test (`scaffolds minimal app ...`), agregar el flag al `execa` (después de `claudeSetup=false`):

```ts
        "--set",
        "claudeSetup=false",
        "--set",
        "maestroSetup=false",
```

Agregar a la sección "Omitted variants" la aserción de ausencia de `.maestro`:

```ts
    await expect(stat(path.join(project, ".maestro"))).rejects.toThrow();
```

Extender el cast de `pkg` de ese test para incluir `scripts` y aserción de ausencia. Cambiar:

```ts
    const pkg = JSON.parse(await readFile(path.join(project, "package.json"), "utf8")) as {
      dependencies: Record<string, string>;
    };
```

por:

```ts
    const pkg = JSON.parse(await readFile(path.join(project, "package.json"), "utf8")) as {
      dependencies: Record<string, string>;
      scripts: Record<string, string>;
    };
```

y agregar al final de las aserciones de ese test:

```ts
    expect(pkg.scripts["test:e2e"]).toBeUndefined();
```

- [ ] **Step 3: Escribir la aserción que falla (prompts en list --json)**

En el tercer test (`list --json includes the expo template ...`), agregar `"maestroSetup"` al final del array esperado:

```ts
    expect(expo?.prompts.map((p) => p.key)).toEqual([
      "projectName",
      "boundedContexts",
      "i18n",
      "zustand",
      "claudeSetup",
      "maestroSetup",
    ]);
```

- [ ] **Step 4: Build y correr los tests para verificar que fallan**

Los e2e corren contra `dist/`, así que hay que buildear primero.

Run:
```bash
cd "D:/personal/projects/libraries/@octavius2929-personal/scaffold-generator"
npm run build && npx vitest run tests/e2e/expo-template.e2e.test.ts
```
Expected: FAIL — el prompt `maestroSetup` no existe (los `--set` de una key desconocida y/o los archivos `.maestro/` faltantes hacen fallar las aserciones).

- [ ] **Step 5: Crear `config.yaml` del workspace Maestro**

Crear `src/templates/expo/files/maestro/.maestro/config.yaml`:

```yaml
# Configuración del workspace Maestro (https://docs.maestro.dev).
# `maestro test .maestro` corre todos los flows .yaml de esta carpeta.
flows:
  - "*.yaml"
```

- [ ] **Step 6: Crear `smoke.yaml` (flow de arranque)**

Crear `src/templates/expo/files/maestro/.maestro/smoke.yaml`:

```yaml
# Flow de humo: arranca la app y verifica que el home renderiza.
#
# appId: applicationId real de tu build Android. Si no definiste `android.package`
# en app.json, Expo genera uno tipo `com.anonymous.<slug>` al hacer prebuild.
# Reemplazá el placeholder de abajo por el applicationId real de tu app.
appId: com.example.app
---
- launchApp
# Reemplazá "home-screen" por el testID de un elemento estable de tu home.
- assertVisible:
    id: "home-screen"
```

- [ ] **Step 7: Crear el `README.md` de `.maestro/`**

Crear `src/templates/expo/files/maestro/.maestro/README.md`:

```markdown
# Tests E2E con Maestro

Flows end-to-end de esta app, ejecutados con [Maestro](https://docs.maestro.dev) sobre un
emulador/simulador con la app corriendo.

## Prerrequisitos

- CLI `maestro` instalado y en el `PATH`.
- Un device (emulador Android / simulador iOS) con la app instalada y corriendo. En
  proyectos con Claude Code, `/rn-setup` prepara el entorno.

## Correr los flows

\`\`\`bash
npm run test:e2e          # corre todos los flows de .maestro/
npm run test:e2e:studio   # abre Maestro Studio para explorar la pantalla
\`\`\`

## Convenciones

- **Selectores por `id`**, que mapea a `testID` en React Native. Usá kebab-case
  (`login-button`). Nunca selecciones por texto visible: cambia con i18n y es frágil.
- Esperá las transiciones con `assertVisible` del elemento destino, nunca con `sleep`.
- Antes de correr `smoke.yaml`, ajustá el `appId` al applicationId real de tu app y
  reemplazá el `id` de ejemplo por un `testID` que exista en tu home.

## Generar flows nuevos

Con el plugin `react-native-maestro-plugin` y `/rn-test "<escenario>"`, Claude explora la
app en vivo y guarda un flow reproducible acá en `.maestro/`.
```

> Nota para el implementador: el bloque \`\`\`bash de arriba va **literal** en el archivo (son las cercas de código del README generado). Las he escapado en este plan para no romper el bloque externo; en el archivo final deben ser tres backticks normales.

- [ ] **Step 8: Crear el fragment de scripts**

Crear `src/templates/expo/fragments/maestro.scripts.json`:

```json
{
  "scripts": {
    "test:e2e": "maestro test .maestro",
    "test:e2e:studio": "maestro studio"
  }
}
```

- [ ] **Step 9: Cablear el prompt en `template.json`**

En `src/templates/expo/template.json`, agregar el prompt al final del array `prompts` (después de `claudeSetup`):

```json
    { "key": "claudeSetup", "type": "confirm", "label": "¿Incluir setup de Claude Code (.claude/ + CLAUDE.md)?", "default": true },
    { "key": "maestroSetup", "type": "confirm", "label": "¿Incluir setup de testing E2E con Maestro?", "default": true }
```

- [ ] **Step 10: Cablear el grupo de archivos en `template.json`**

En `files.include`, agregar al final (después del bloque `files/claude/**/*`):

```json
      { "glob": "files/claude/**/*", "strip": "files/claude/", "when": "claudeSetup == true" },
      { "glob": "files/maestro/**/*", "strip": "files/maestro/", "when": "maestroSetup == true" }
```

- [ ] **Step 11: Cablear el postAction `mergeJson` y `nextSteps` en `template.json`**

En `postActions`, insertar el merge de scripts **después** del mergeJson de i18n y **antes** de `gitInit`:

```json
    { "type": "mergeJson", "target": "package.json", "source": "fragments/i18n.deps.json", "when": "i18n == true" },
    { "type": "mergeJson", "target": "package.json", "source": "fragments/maestro.scripts.json", "when": "maestroSetup == true" },
    { "type": "gitInit", "initialCommit": "chore: initial scaffold" },
```

Y en el `nextSteps`, agregar una línea al final del array `lines` (después de `"npm start ..."`):

```json
      "npm start     # expo start --dev-client",
      "npm run test:e2e   # si incluiste Maestro: requiere el CLI maestro y un device con la app"
```

- [ ] **Step 12: Actualizar la `description` del template**

En `src/templates/expo/template.json`, cambiar el final de `description` de:

```
... Zustand opcional, setup Claude Code opcional)
```

a:

```
... Zustand opcional, setup Claude Code opcional, testing E2E con Maestro opcional)
```

(Aplica al campo `"description"` de la línea 4 del `template.json`.)

- [ ] **Step 13: Build y correr los tests para verificar que pasan**

Run:
```bash
cd "D:/personal/projects/libraries/@octavius2929-personal/scaffold-generator"
npm run build && npx vitest run tests/e2e/expo-template.e2e.test.ts
```
Expected: PASS (3 tests verdes). Verifica que `.maestro/` se genera con `maestroSetup=true`, se omite con `false`, y los scripts se mergean.

- [ ] **Step 14: Lint y typecheck**

Run:
```bash
npm run typecheck && npm run lint
```
Expected: sin errores. Si Biome reporta formato en archivos nuevos, corré `npm run lint:fix` y revisá el diff.

- [ ] **Step 15: Commit**

```bash
git add src/templates/expo/files/maestro src/templates/expo/fragments/maestro.scripts.json src/templates/expo/template.json tests/e2e/expo-template.e2e.test.ts
git commit -m "feat(expo): add opt-in Maestro E2E setup to template"
```

---

## Task 2: Skill `conventions-maestro` (standards)

**Repo:** `standards`

**Files:**
- Create: `plugin/skills/conventions-maestro/SKILL.md`

**Interfaces:**
- Consumes: las skills de Claude Code se auto-descubren desde `plugin/skills/<name>/SKILL.md` (frontmatter solo con `description:`, igual que `conventions-react`). No hay array de skills en `plugin.json` que actualizar ni entrada por-skill en el marketplace.
- Produces: skill `conventions-maestro` disponible, que documenta la convención E2E y referencia `react-native-maestro-plugin` y `reusable-assets`.

- [ ] **Step 1: Crear la skill**

Crear `plugin/skills/conventions-maestro/SKILL.md`:

```markdown
---
description: Convenciones de testing E2E de Justin para apps Expo / React Native con Maestro (estructura .maestro/, selectores por testID, scripts). Úsalo al testear o configurar E2E en apps Expo/RN propias.
---

# Convenciones Maestro (E2E Expo / React Native)

- **Maestro es el estándar de E2E** para apps Expo/RN propias (no Playwright, que no llega
  a la capa nativa de iOS/Android). Ver skill `reusable-assets` antes de armar nada a mano.
- **Estructura:** flows planos en `.maestro/*.yaml`, con `.maestro/config.yaml` y un
  `.maestro/smoke.yaml` de arranque. Este setup lo genera el template `expo` de
  `@octavius2929-personal/scaffold-generator` con la opción `maestroSetup`.
- **Selectores por `id`** (= `testID` en RN), en kebab-case (`login-button`). Nunca por
  texto: cambia con i18n y es frágil. Si falta un `testID`, agregalo en el código.
- **Esperá transiciones con `assertVisible`** del elemento destino, nunca con `sleep` fijo.
- **Scripts:** `npm run test:e2e` (`maestro test .maestro`) y `npm run test:e2e:studio`
  (`maestro studio`). Requieren el CLI `maestro` y un device con la app corriendo.
- **Sesiones asistidas:** usá el plugin `react-native-maestro-plugin` — `/rn-setup` prepara
  el entorno y `/rn-test "<escenario>"` explora la app y guarda un flow reproducible.
```

- [ ] **Step 2: Verificar el frontmatter y la ubicación**

Run:
```bash
cd "D:/personal/projects/libraries/@octavius2929-personal/standards"
head -3 plugin/skills/conventions-maestro/SKILL.md
ls plugin/skills/conventions-maestro/SKILL.md
```
Expected: el archivo existe y empieza con `---` + línea `description:` + `---` (mismo formato que `plugin/skills/conventions-react/SKILL.md`).

- [ ] **Step 3: Commit**

```bash
git add plugin/skills/conventions-maestro/SKILL.md
git commit -m "feat(skills): add conventions-maestro E2E conventions skill"
```

---

## Task 3: Sección "Convención por-proyecto" en el README del plugin (react-native-playwright)

**Repo:** `react-native-playwright`

**Files:**
- Modify: `README.md`

**Interfaces:**
- Consumes: README existente del plugin (secciones: Qué hace, Prerrequisitos, Instalación, Uso, Por qué no es "Playwright", Licencia).
- Produces: una sección nueva que enlaza el setup por-proyecto (generado por el scaffold) y la skill `conventions-maestro`, cerrando el círculo tooling↔proyecto.

- [ ] **Step 1: Agregar la sección**

En `README.md`, insertar esta sección **antes** de `## Por qué no es "Playwright"`:

```markdown
## Convención por-proyecto

El setup de Maestro *dentro* de cada app (carpeta `.maestro/` con `config.yaml`,
`smoke.yaml` y `README.md`, más los scripts `test:e2e` / `test:e2e:studio`) lo genera el
template `expo` de
[`@octavius2929-personal/scaffold-generator`](https://github.com/proyectos-octavio/scaffold-generator)
al activar la opción `maestroSetup`. Las convenciones están documentadas en la skill
`conventions-maestro` del plugin `standards`. Este plugin aporta la capa de **sesión
asistida** (`/rn-setup`, `/rn-test`) encima de ese setup.
```

- [ ] **Step 2: Verificar el render**

Run:
```bash
cd "D:/personal/projects/libraries/@octavius2929-personal/react-native-playwright"
grep -n "Convención por-proyecto" README.md
```
Expected: una coincidencia, ubicada antes de la línea de `## Por qué no es "Playwright"`.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: link per-project Maestro convention from plugin README"
```

---

## Self-review (cobertura del spec)

- Convención por-proyecto (`.maestro/` + scripts) → Task 1 (Steps 5–8, 11).
- Selectores por `id`/`testID`, `assertVisible` sobre `sleep` → documentado en `.maestro/README.md` (Task 1 Step 7) y skill (Task 2).
- Integración al scaffold (prompt opt-in, file group, mergeJson, nextSteps, description) → Task 1 (Steps 9–12), verificada por e2e (Steps 1–4, 13).
- Skill de convenciones → Task 2.
- README del plugin → Task 3.
- Fuera de alcance respetado: sin CI, sin iOS, sin tocar `maestro-rn-testing`/`/rn-setup`/`/rn-test`.

## Notas de publicación (fuera del alcance de las tareas)

- **`scaffold-generator` tiene una desincronización de versión preexistente**: `package.json` (0.7.0) ≠ `plugin/.claude-plugin/plugin.json` (0.9.0). El script `check:plugin-version` falla hoy, independientemente de este cambio. Antes de publicar, el autor debe decidir la fuente de verdad y sincronizar ambas (y bumpear minor por la nueva feature). No lo hago en las tareas para no resolver una decisión de versionado ajena.
- **`standards`**: agregar una skill amerita un bump de `plugin/.claude-plugin/plugin.json` (0.2.0 → 0.3.0) antes de redistribuir el plugin. No hay check de sync con `package.json` (0.1.0) en ese repo, así que es seguro pero opcional para el working tree.
- El marketplace lista plugins, no skills individuales: no requiere cambios.

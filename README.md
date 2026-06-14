# react-native-maestro-plugin

Plugin de Claude Code que convierte a Claude en un **agente de testing E2E** para apps
**Expo / React Native**, encima del **Maestro MCP oficial**. Explora la app en vivo en un
emulador/simulador y deja **flows Maestro reproducibles** listos para CI.

## Qué hace

- **Auto-registra** el Maestro MCP al instalar el plugin (`.mcp.json`).
- Aporta convenciones Expo/RN (testID, navegación, deep links, selectores robustos) vía la
  skill `maestro-rn-testing`.
- Dos comandos: `/rn-setup` (prepara el entorno) y `/rn-test` (sesión de testing por IA que
  guarda un flow en `.maestro/`).

## Prerrequisitos

- [Maestro CLI](https://docs.maestro.dev) instalado (el binario `maestro` en el PATH).
- Un emulador Android / simulador iOS y la app Expo corriendo.

## Instalación

```
/plugin marketplace add proyectos-octavio/plugins-marketplace
/plugin install react-native-maestro-plugin@octavius2929-personal
```

## Uso

```
/rn-setup android        # prepara device + app + verifica el MCP
/rn-test "login con email"  # explora el escenario y guarda .maestro/login-con-email.yaml
```

Re-ejecutá el flow generado sin IA:

```
maestro test .maestro/login-con-email.yaml
```

## Por qué no es "Playwright"

Playwright solo automatiza navegadores; no llega a la capa nativa de iOS/Android. Para E2E
nativo de RN el motor real es Maestro. Este plugin es la capa Expo/RN encima del Maestro MCP.

## Licencia

MIT

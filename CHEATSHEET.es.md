# 🧪 e2e de TesterArmy — Guía rápida (Español)

> Comandos verificados contra el README y la documentación oficial de [tester-army/e2e](https://github.com/tester-army/e2e) (2026-10-12).
> Qué es: un framework de pruebas end-to-end para apps web y móviles. Describes una meta en lenguaje natural, un agente maneja la app y tú verificas el resultado con localizadores y aserciones en la misma prueba.

---

## 0. Arranca con un comando

```bash
npx e2e init        # npm — pnpm: pnpm dlx e2e init · bun: bunx e2e init
```

El asistente pide un motor — **Web (Playwright)** o **Mobile (iOS/Android)** — y un proveedor de modelo (o **None** si no quieres IA). Escribe `e2e.config.ts`, una prueba de ejemplo en `tests/`, y agrega las dependencias y un script `test:e2e` a `package.json`. Tus archivos actuales quedan intactos.

- Requisitos: Node.js 24.8 o superior (22.22.3+ en la línea 22). En Windows, trabaja dentro de WSL.
- Ejecutar todo: `npx e2e run` · un solo archivo: `npx e2e run tests/agent.e2e.ts` · ver el navegador: agrega `--headed`.

Instalación manual, sin asistente:

```bash
npm install --save-dev e2e @e2e-dev/web ai@^7
```

(`ai` solo hace falta para los pasos `agent.*`. Para móvil, usa `@e2e-dev/mobile` en su lugar.)

---

## 1. Una prueba, tres tipos de pasos

```ts
// tests/checkout.e2e.ts
import { test, expect } from 'e2e';

test('a member upgrades to Pro', async ({ app, agent, screen }) => {
  await app.open('/settings/billing');

  await agent.act('upgrade the workspace to the Pro plan');

  await agent.assert('the invoice preview shows a prorated amount');

  await expect(screen.getByRole('status')).toContainText('Pro');
});
```

- `app.open(...)` — navegación directa, sin modelo de por medio.
- `agent.act('...')` — una meta en lenguaje natural; el agente resuelve los clics. Una meta por llamada. Los `{marcadores}` de la meta se rellenan con un objeto `params`.
- `agent.assert('...')` — el modelo juzga lo que se ve en pantalla.
- `expect(...)` — la comprobación exacta de siempre, con localizadores estilo Playwright como `screen.getByRole(...)`.

Mézclalos como quieras. Los pasos sin agente no necesitan modelo.

---

## 2. Graba una vez, repite sin tokens

Aquí está el ahorro:

1. Ejecutas una prueba con pasos de agente. Cuando una aserción posterior verifica un paso del agente, e2e **graba las acciones** que el agente hizo.
2. En la siguiente ejecución, esas acciones **se repiten desde la caché: cero llamadas al modelo, cero tokens**, hasta que la app cambie.
3. Si la app cambió, el agente rehace ese paso y la grabación se actualiza.

Datos que vale la pena saber:

- La caché de repetición se puede **guardar en tu repo** con un commit, así CI también repite sin modelo.
- Saltarse la caché en una ejecución: `npx e2e run --no-cache`.
- Revisar la caché: `npx e2e cache ls`.
- El modo de caché se configura: `off`, `read-only` o `read-write`.

Pagas el modelo cuando un flujo es nuevo o cambió. No en cada corrida de CI.

---

## 3. Trae tu propio modelo

Sin cuenta de e2e ni créditos incluidos. Usa lo que ya pagas:

| Opción | Configuración |
| --- | --- |
| Suscripción ChatGPT Plus o Pro | `npx e2e login openai` |
| Suscripción GitHub Copilot | `npx e2e login github-copilot` |
| Suscripción OpenCode Console | `npx e2e login opencode-console` |
| SuperGrok o X Premium+ | `npx e2e login spacexai` |
| Vercel AI Gateway (clave API) | variable `AI_GATEWAY_API_KEY` |
| OpenRouter (clave API) | variable `OPENROUTER_API_KEY` |
| Modelo local | soportado — ver la doc de Models |

---

## 4. ¿Vienes de Playwright? El cambio es chico

|  | Playwright | e2e |
| --- | --- | --- |
| Motor de navegador | Chromium, Firefox, WebKit | Los mismos tres: `@e2e-dev/web` los maneja vía Playwright |
| Import de la prueba | `@playwright/test` | `e2e` (config en `e2e.config.ts`, pruebas con nombre `*.e2e.ts`) |
| Localizadores | `page.getByRole(...)` | `screen.getByRole(...)` — la misma idea |
| Acciones | Escribes cada clic | Pasos exactos, o una meta para `agent.act` |
| Aserciones | `expect` | El mismo `expect`, más `agent.assert` para revisar la pantalla |
| Costo por corrida | Tu tiempo | Modelo solo en flujos nuevos o cambiados; la repetición no llama al modelo |

Notas prácticas:

- El motor web fija la versión de `playwright-core`; no agregues el paquete `playwright` encima.
- Los navegadores que falten se descargan en la primera ejecución. En CI: `npx @e2e-dev/web install chromium --with-deps`.
- `npx e2e init` dentro de un proyecto existente no toca tus pruebas ni configs actuales: pruébalo primero con un solo flujo.

---

## 5. Paquetes, por si necesitas más que web

| Paquete | Qué hace |
| --- | --- |
| `e2e` | El SDK, el runner y la CLI |
| `@e2e-dev/web` | Motor de navegador: Chromium, Firefox y WebKit vía Playwright |
| `@e2e-dev/mobile` | Motor iOS y Android: simuladores y emuladores |
| `@e2e-dev/github` | Reporter que publica resultados como comentario en el pull request |

Telemetría: la CLI envía conteos anónimos de uso, nunca contenido de pruebas ni credenciales. Para apagarla: `npx e2e telemetry disable` o `E2E_TELEMETRY_DISABLED=1`.

Estado: pre-1.0 y en movimiento. Las APIs todavía pueden cambiar entre versiones menores.

---

*Documentación: https://e2e.tester.army/docs · Repo: https://github.com/tester-army/e2e · El paquete `e2e` trae la documentación completa sin conexión en `node_modules/e2e/docs`.*

# 🧪 e2e by TesterArmy — Quick Guide (English)

> Commands verified against the [tester-army/e2e](https://github.com/tester-army/e2e) README and official docs (2026-10-12).
> What it is: an end-to-end testing framework for web and mobile apps. You describe a goal in plain language, an agent drives the app, and you check the result with locators and assertions in the same test.

---

## 0. Start with one command

```bash
npx e2e init        # npm — pnpm: pnpm dlx e2e init · bun: bunx e2e init
```

The wizard asks for an engine — **Web (Playwright)** or **Mobile (iOS/Android)** — and a model provider (or **None** for tests without AI). It writes `e2e.config.ts`, an example test under `tests/`, and adds the dependencies plus a `test:e2e` script to `package.json`. Your existing files stay intact.

- Requirements: Node.js 24.8 or newer (22.22.3+ on the Node 22 line). On Windows, run inside WSL.
- Run everything: `npx e2e run` · one file: `npx e2e run tests/agent.e2e.ts` · watch the browser: add `--headed`.

Manual install, no wizard:

```bash
npm install --save-dev e2e @e2e-dev/web ai@^7
```

(`ai` is only needed for `agent.*` steps. For mobile, swap in `@e2e-dev/mobile`.)

---

## 1. One test, three kinds of steps

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

- `app.open(...)` — plain navigation, no model involved.
- `agent.act('...')` — a goal in natural language; the agent works out the clicks. One goal per call. `{placeholders}` in the goal fill from a `params` object.
- `agent.assert('...')` — the model judges what is on screen.
- `expect(...)` — the classic exact check, with Playwright-style locators such as `screen.getByRole(...)`.

Mix them freely. Steps without an agent need no model at all.

---

## 2. Record once, replay with zero tokens

This is the part that pays for the framework:

1. Run a test with agent steps. When a later assertion verifies an agent step, e2e **records the actions** the agent took.
2. On the next run, those actions **replay from the cache — no model calls, no tokens** — until the app changes.
3. If the app changed, the agent redoes that step and the recording refreshes.

Worth knowing:

- The replay cache can be **committed to your repo**, so CI replays too.
- Skip the cache for one run: `npx e2e run --no-cache`.
- Look at the cache: `npx e2e cache ls`.
- Cache mode is a config option: `off`, `read-only`, or `read-write`.

You pay for the model when a flow is new or has changed — not on every CI run.

---

## 3. Bring your own model

No e2e account, no bundled credits. Use what you already pay for:

| Option | Setup |
| --- | --- |
| ChatGPT Plus or Pro subscription | `npx e2e login openai` |
| GitHub Copilot subscription | `npx e2e login github-copilot` |
| OpenCode Console subscription | `npx e2e login opencode-console` |
| SuperGrok or X Premium+ | `npx e2e login spacexai` |
| Vercel AI Gateway (API key) | set `AI_GATEWAY_API_KEY` |
| OpenRouter (API key) | set `OPENROUTER_API_KEY` |
| Local model | supported — see the Models doc |

---

## 4. Coming from Playwright? The switch is small

|  | Playwright | e2e |
| --- | --- | --- |
| Browser engine | Chromium, Firefox, WebKit | The same three — `@e2e-dev/web` drives them through Playwright |
| Test import | `@playwright/test` | `e2e` (config in `e2e.config.ts`, tests named `*.e2e.ts`) |
| Locators | `page.getByRole(...)` | `screen.getByRole(...)` — same idea |
| Actions | You write every click | Keep exact steps, or hand a goal to `agent.act` |
| Assertions | `expect` | The same `expect`, plus `agent.assert` for screen-level checks |
| Cost per run | Your time | Model only on new or changed flows; replays make no model calls |

Practical notes:

- The web engine pins `playwright-core`; do not add the `playwright` package on top of it.
- Missing browsers download on first run. In CI: `npx @e2e-dev/web install chromium --with-deps`.
- `npx e2e init` inside an existing project leaves your current tests and configs alone — try it on one flow first.

---

## 5. Packages, if you need more than web

| Package | What it does |
| --- | --- |
| `e2e` | The SDK, runner, and CLI |
| `@e2e-dev/web` | Browser engine: Chromium, Firefox, WebKit through Playwright |
| `@e2e-dev/mobile` | iOS and Android engine: simulators and emulators |
| `@e2e-dev/github` | Reporter that posts results as a pull request comment |

Telemetry: the CLI sends anonymous usage counts — never test content or credentials. Turn it off with `npx e2e telemetry disable` or `E2E_TELEMETRY_DISABLED=1`.

Status: pre-1.0 and moving fast. APIs can still change between minor releases.

---

*Docs: https://e2e.tester.army/docs · Repo: https://github.com/tester-army/e2e · The `e2e` package also ships the full docs offline in `node_modules/e2e/docs`.*

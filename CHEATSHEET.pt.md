# 🧪 e2e da TesterArmy — Guia rápido (Português)

> Comandos conferidos no README e na documentação oficial do [tester-army/e2e](https://github.com/tester-army/e2e) (2026-10-12).
> O que é: um framework de testes end-to-end para apps web e mobile. Você descreve uma meta em linguagem natural, um agente opera o app e você confere o resultado com localizadores e asserções no mesmo teste.

---

## 0. Comece com um comando

```bash
npx e2e init        # npm — pnpm: pnpm dlx e2e init · bun: bunx e2e init
```

O assistente pede um motor — **Web (Playwright)** ou **Mobile (iOS/Android)** — e um provedor de modelo (ou **None**, para testes sem IA). Ele escreve `e2e.config.ts`, um teste de exemplo em `tests/`, e adiciona as dependências e um script `test:e2e` ao `package.json`. Seus arquivos atuais ficam como estão.

- Requisitos: Node.js 24.8 ou mais novo (22.22.3+ na linha 22). No Windows, rode dentro do WSL.
- Rodar tudo: `npx e2e run` · um arquivo só: `npx e2e run tests/agent.e2e.ts` · ver o navegador: acrescente `--headed`.

Instalação manual, sem assistente:

```bash
npm install --save-dev e2e @e2e-dev/web ai@^7
```

(O `ai` só é necessário para os passos `agent.*`. Para mobile, troque por `@e2e-dev/mobile`.)

---

## 1. Um teste, três tipos de passo

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

- `app.open(...)` — navegação direta, sem modelo no meio.
- `agent.act('...')` — uma meta em linguagem natural; o agente resolve os cliques. Uma meta por chamada. Os `{marcadores}` da meta são preenchidos com um objeto `params`.
- `agent.assert('...')` — o modelo julga o que aparece na tela.
- `expect(...)` — a conferência exata de sempre, com localizadores no estilo Playwright, como `screen.getByRole(...)`.

Misture à vontade. Passos sem agente não precisam de modelo.

---

## 2. Grave uma vez, repita sem tokens

É aqui que a conta fecha:

1. Você roda um teste com passos de agente. Quando uma asserção mais adiante confirma um passo do agente, o e2e **grava as ações** que o agente executou.
2. Na rodada seguinte, essas ações **se repetem a partir do cache: zero chamadas ao modelo, zero tokens**, até o app mudar.
3. Se o app mudou, o agente refaz aquele passo e a gravação é atualizada.

Detalhes que importam:

- O cache de repetição pode ser **salvo no seu repo** com um commit; aí o CI também repete sem modelo.
- Pular o cache numa rodada: `npx e2e run --no-cache`.
- Espiar o cache: `npx e2e cache ls`.
- O modo do cache é configurável: `off`, `read-only` ou `read-write`.

Você paga o modelo quando um fluxo é novo ou mudou. Não a cada rodada do CI.

---

## 3. Traga seu próprio modelo

Sem conta do e2e, sem créditos embutidos. Use o que você já paga:

| Opção | Configuração |
| --- | --- |
| Assinatura ChatGPT Plus ou Pro | `npx e2e login openai` |
| Assinatura GitHub Copilot | `npx e2e login github-copilot` |
| Assinatura OpenCode Console | `npx e2e login opencode-console` |
| SuperGrok ou X Premium+ | `npx e2e login spacexai` |
| Vercel AI Gateway (chave de API) | variável `AI_GATEWAY_API_KEY` |
| OpenRouter (chave de API) | variável `OPENROUTER_API_KEY` |
| Modelo local | suportado — ver a doc de Models |

---

## 4. Vem do Playwright? A troca é pequena

|  | Playwright | e2e |
| --- | --- | --- |
| Motor de navegador | Chromium, Firefox, WebKit | Os mesmos três: o `@e2e-dev/web` os comanda via Playwright |
| Import do teste | `@playwright/test` | `e2e` (config em `e2e.config.ts`, testes com nome `*.e2e.ts`) |
| Localizadores | `page.getByRole(...)` | `screen.getByRole(...)` — a mesma ideia |
| Ações | Você escreve cada clique | Passos exatos, ou uma meta para o `agent.act` |
| Asserções | `expect` | O mesmo `expect`, mais o `agent.assert` para conferir a tela |
| Custo por rodada | Seu tempo | Modelo só em fluxos novos ou alterados; a repetição não chama o modelo |

Notas práticas:

- O motor web fixa a versão do `playwright-core`; não adicione o pacote `playwright` por cima.
- Navegadores que faltarem são baixados na primeira rodada. No CI: `npx @e2e-dev/web install chromium --with-deps`.
- `npx e2e init` dentro de um projeto existente não mexe nos seus testes e configs atuais: teste primeiro com um fluxo só.

---

## 5. Pacotes, se precisar de mais que web

| Pacote | O que faz |
| --- | --- |
| `e2e` | O SDK, o runner e a CLI |
| `@e2e-dev/web` | Motor de navegador: Chromium, Firefox e WebKit via Playwright |
| `@e2e-dev/mobile` | Motor iOS e Android: simuladores e emuladores |
| `@e2e-dev/github` | Reporter que publica resultados como comentário no pull request |

Telemetria: a CLI envia contagens anônimas de uso, nunca conteúdo de testes nem credenciais. Para desligar: `npx e2e telemetry disable` ou `E2E_TELEMETRY_DISABLED=1`.

Estado: pré-1.0 e andando rápido. As APIs ainda podem mudar entre versões menores.

---

*Documentação: https://e2e.tester.army/docs · Repo: https://github.com/tester-army/e2e · O pacote `e2e` traz a documentação completa offline em `node_modules/e2e/docs`.*

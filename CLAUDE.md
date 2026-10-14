# CLAUDE.md — colirium-canva

Editor de imagens desktop (estilo Canva) para montar imagens de PDP (página de produto de e-commerce)
a partir de fotos reais de produto. Uso pessoal, roda no **Windows**. Toda função do app também é
operável por IA via **MCP** (Claude Code / Claude Desktop).

Leia antes de qualquer tarefa não trivial:
- `docs/ARCHITECTURE.md` — camadas, processos, fluxo de dados (fonte da verdade da arquitetura)
- `docs/DOCUMENT-MODEL.md` — formato do projeto/design em disco (schemas)
- `docs/AI-INTEGRATION.md` — servidor MCP e catálogo de ferramentas
- `docs/ROADMAP.md` — fases e critérios de aceite; trabalhe sempre na fase atual
- `docs/DECISIONS.md` — decisões já tomadas (não reabra sem motivo forte; se reabrir, registre lá)

## Stack (decidida — ver DECISIONS.md)

- **Electron** + **electron-vite** + **TypeScript (strict)** em todo o código
- UI: **React 18+**, **react-konva/Konva** (canvas), **Zustand** (estado de UI), **Immer** (documento com patches), **Tailwind** + **Radix UI** (componentes)
- Validação/contratos: **zod** (todo comando e toda entrada de IA)
- Imagem no processo main: **sharp** (resize Lanczos, PNG/WebP/JPEG/TIFF/AVIF), decodificação RAW atrás da interface `RawDecoder` (ver ROADMAP Fase 0)
- IA: **@modelcontextprotocol/sdk** (servidor MCP Streamable HTTP em `127.0.0.1`)
- Testes: **Vitest** (unidade), **Playwright** com Electron (e2e), testes de imagem "golden"
- Empacotamento: **electron-builder** (NSIS, Windows x64)
- Gerenciador de pacotes: **pnpm**

## A regra de ouro: tudo é um Comando

Toda mutação do documento passa pelo **Command Registry** (`src/core/commands/`).
Um comando é definido UMA vez (id, descrição, schema zod, handler puro) e automaticamente vira:
1. ação na UI (botões, atalhos, paleta de comandos `Ctrl+K`),
2. entrada no histórico de desfazer/refazer (patches Immer),
3. **ferramenta MCP** para IAs.

Nunca mude o documento direto de um componente React. Nunca crie uma ferramenta MCP "à mão" que
duplique lógica de um comando. Se a IA precisa de algo, crie/estenda um comando.

## Regras de código

- Código, identificadores e commits em **inglês**; textos de UI em **pt-BR** (centralizados em `src/renderer/i18n/pt-BR.ts`).
- `src/core/` é **TypeScript puro**: sem React, sem Electron, sem Node APIs. É testável com Vitest e é usado pelo main e pelo renderer.
- Handlers de comando são **funções puras** `(doc, params) => void` executadas dentro de `produce` do Immer. Efeitos colaterais (disco, sharp, rede) ficam em **services** no main.
- Coordenadas do documento são **pixels da imagem original** (1 unidade = 1 px da foto). Nunca guarde coordenadas de tela.
- A **foto original nunca é alterada**. Ajustes, recorte e rotação são parâmetros não destrutivos.
- IPC entre renderer e main só por canais tipados em `src/shared/ipc.ts` (contextIsolation ligado, nodeIntegration desligado, preload mínimo).
- Arquivos pequenos e focados (< ~300 linhas). Uma feature = uma pasta em `src/features/<feature>/`.
- Sem `any`. Sem dependência nova sem registrar o motivo em `docs/DECISIONS.md`.

## Como trabalhar (fluxo esperado)

1. Leia a fase atual em `docs/ROADMAP.md` e escolha UM item.
2. Antes de codar algo com mais de ~3 arquivos, escreva um plano curto (o que muda, quais comandos, quais testes) e siga-o.
3. Implemente em fatia vertical: schema → comando → teste do core → UI → exposição MCP → teste e2e se houver fluxo de usuário.
4. Rode `pnpm typecheck && pnpm lint && pnpm test` antes de considerar pronto.
5. Atualize a doc afetada (`DOCUMENT-MODEL.md` se mudou schema, `AI-INTEGRATION.md` se mudou ferramenta) e marque o item no ROADMAP.
6. Commit pequeno por item, mensagem no formato Conventional Commits (`feat(text): add letter spacing`).

Mudou o schema do documento? Incremente `schemaVersion` e escreva uma migração em `src/core/document/migrations/`.

## Comandos do projeto

```bash
pnpm dev          # app em modo desenvolvimento (inclui servidor MCP em 127.0.0.1:47800)
pnpm test         # vitest
pnpm test:e2e     # playwright + electron
pnpm typecheck
pnpm lint
pnpm build:win    # instalador NSIS em dist/
```

## Slash commands deste repo

- `/fase` — mostra o estado do ROADMAP e propõe o próximo item
- `/feature <descrição>` — implementa uma feature nova seguindo o fluxo acima
- `/comando <descrição>` — cria um novo Comando (UI + undo + MCP) com testes

## Usando o app como IA

Para editar imagens (e não o código), use a skill `pdp-designer` em `.claude/skills/pdp-designer/` e as
ferramentas MCP do servidor `colirium` (configurado em `.mcp.json`; o app precisa estar aberto).

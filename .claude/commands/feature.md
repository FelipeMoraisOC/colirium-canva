---
description: Implementa uma feature nova em fatia vertical (core → UI → MCP → testes)
argument-hint: <descrição da feature>
---

Feature pedida: $ARGUMENTS

Siga o fluxo do `CLAUDE.md`:

1. Leia `docs/ARCHITECTURE.md`, `docs/DOCUMENT-MODEL.md` e a fase atual do `docs/ROADMAP.md`. Se a feature não está no roadmap, adicione-a na fase adequada.
2. Escreva um plano curto e mostre antes de codar:
   - mudanças de schema (e se precisa migração / `schemaVersion`)
   - comandos novos ou alterados (id, params, `exposeToAi`)
   - arquivos em `src/features/<feature>/`
   - testes (unidade no core; e2e se houver fluxo de usuário; golden se mexer em render)
3. Implemente nessa ordem: schema → comandos + testes Vitest → componentes UI → registro em `index.ts` da feature → verificar que as ferramentas MCP aparecem.
4. Rode `pnpm typecheck && pnpm lint && pnpm test` e corrija até passar.
5. Atualize `docs/DOCUMENT-MODEL.md` / `docs/AI-INTEGRATION.md` se aplicável e marque o item no ROADMAP.
6. Faça um commit Conventional Commits e resuma: o que mudou, como testar na UI, como a IA usa.

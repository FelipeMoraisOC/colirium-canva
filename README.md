# colirium-canva

Editor de imagens desktop (Windows), estilo Canva, para transformar fotos reais de produto em imagens
de PDP. Operável pela interface **e** por IA (Claude Code / Claude Desktop via MCP).

## Decisões em uma frase

- **Linguagem:** TypeScript em tudo.
- **App:** Electron + React + Konva (canvas) + sharp (processamento e exportação de imagem).
- **Arquitetura:** documento JSON + **Command Registry**. Cada ação existe uma vez e vira botão, passo de desfazer e ferramenta de IA ao mesmo tempo.
- **IA:** o app hospeda um servidor **MCP** local; o Claude enxerga o canvas (`render_preview`), lê suas referências de estilo e edita ao vivo. Sem chave de API no app.
- **Modo de programar:** roadmap por fases, fatias verticais por feature, testes obrigatórios, docs como fonte da verdade para a IA.

Detalhes e justificativas: [`docs/DECISIONS.md`](docs/DECISIONS.md).

## Como começar com o Claude Code

1. Crie a pasta do repositório, ex.: `D:\00 - ArtRock\colirium-canva`, e copie **todo o conteúdo desta pasta** para lá (incluindo `.claude/` e `.mcp.json`, que são ocultos).
2. Instale no Windows: **Node.js LTS**, **pnpm** (`npm i -g pnpm`), **Git**.
3. No terminal, dentro da pasta: `git init` e depois `claude`.
4. Primeira mensagem para o Claude Code:
   > Leia o CLAUDE.md e todos os arquivos de docs/. Depois execute a Fase 0 do ROADMAP, item por item, fazendo commit a cada item. No spike de RAW, me pergunte qual câmera eu uso e peça arquivos de exemplo.
5. A partir daí, use:
   - `/fase` para ver onde está e o próximo passo
   - `/feature <descrição>` para pedir algo novo
   - `/comando <descrição>` para uma ação nova de edição
6. Quando a Fase 5 estiver pronta, defina a variável de ambiente `COLIRIUM_MCP_TOKEN` com o token que o app mostra em Configurações, abra o app e peça ao Claude Code, por exemplo:
   > Use a skill pdp-designer: abra o projeto Gasoline Premium, foto "modelo-oficina", e faça um PDP no estilo da referência artrock-gasoline-premium com os benefícios 100% algodão, modelagem confortável, leve e macia, garantia de fábrica.

## O que tem aqui

| Arquivo | Para quê |
|---|---|
| `CLAUDE.md` | Regras que o Claude Code lê em toda sessão |
| `docs/ARCHITECTURE.md` | Camadas, pastas, pipeline de render/export/import |
| `docs/DOCUMENT-MODEL.md` | Formato dos projetos, designs, blocos, templates e referências |
| `docs/AI-INTEGRATION.md` | Servidor MCP e catálogo de ferramentas para IA |
| `docs/ROADMAP.md` | Fases 0–7 com critérios de aceite |
| `docs/DECISIONS.md` | Por que cada escolha foi feita |
| `.claude/commands/` | Slash commands `/fase`, `/feature`, `/comando` |
| `.claude/skills/pdp-designer/` | Como a IA deve desenhar PDPs usando o app |
| `.claude/settings.json` | Permissões para o Claude Code rodar pnpm/git sem perguntar |
| `.mcp.json` | Conecta o Claude Code ao app quando ele estiver aberto |
| `references/artrock-gasoline-premium/` | Suas 4 imagens de referência + análise de estilo (seed do sistema de referências) |

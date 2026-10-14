---
description: Cria um novo Comando no registry (vira ação de UI, passo de undo e ferramenta MCP)
argument-hint: <o que o comando faz>
---

Comando pedido: $ARGUMENTS

1. Escolha o id `namespace.verbo` coerente com os existentes (`grep -r "id: '" src/features`).
2. Defina o schema zod dos parâmetros com `.describe()` em TODOS os campos, escritos para uma IA entender (unidade, faixa, exemplo).
3. `description` do comando: uma frase do que faz + quando usar + um exemplo de parâmetros.
4. Handler puro (`run`) operando no draft Immer. Erros com mensagens acionáveis (liste opções válidas).
5. Se aceita posição, aceite também `anchor` + `marginPct` usando `src/core/layout/anchors.ts`.
6. Testes Vitest: caso feliz, validação de params inválidos, undo restaura o estado, roda dentro de transação.
7. Registre no `index.ts` da feature; adicione atalho/botão se fizer sentido para o usuário.
8. Confirme que a ferramenta MCP gerada aparece (teste de contrato em `tests/mcp/`).
9. `pnpm typecheck && pnpm lint && pnpm test`.

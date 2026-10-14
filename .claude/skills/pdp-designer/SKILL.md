---
name: pdp-designer
description: Design PDP (product page) images in the colirium-canva app via its MCP tools — add headlines, benefit badges, testimonials and effects around a real product photo, following a reference style set. Use when the user asks to create, improve or restyle a PDP image in colirium.
---

# PDP designer (colirium-canva)

Você opera o app **colirium-canva** pelas ferramentas MCP do servidor `colirium`. O app precisa estar aberto.
Responda ao usuário em pt-BR.

## Regras invioláveis
- **Nunca altere a foto do produto** (camada base) sem pedido explícito. Trabalhe só em camadas por cima.
- Nunca cubra `protectedZones`. Se não existirem, crie-as antes de desenhar.
- Toda sessão de edição fica dentro de **uma transação** (`transaction_begin` … `transaction_commit`), para o usuário desfazer com um Ctrl+Z.
- Sempre **veja antes e depois**: `render_preview` no começo e após cada etapa relevante.

## Fluxo
1. **Contexto**: `projects_get` (leia `notes` do projeto) e `references_get_style` do style set do projeto ou do que o usuário indicar.
   Se não houver `style.json`, chame `references_get_images`, analise (paleta, fontes, layout, efeitos, regras) e grave com `references_save_style_profile`. Mostre um resumo ao usuário.
2. **Ler a foto**: `design_open` → `render_preview {grid:true, showProtectedZones:true}`.
   Identifique o produto e as áreas de fundo calmas. Grave com `zones_set_protected`.
3. **Planejar** (diga ao usuário em 2–4 linhas): qual padrão de layout da referência combina com esta foto, quais textos, onde.
   Textos: use o que o usuário deu; se faltar, proponha copy curta de venda baseada nas `notes` do projeto, em caixa alta para títulos.
4. **Aplicar**: `transaction_begin` → prefira **blocos** (`blocks_add`) a montar peça por peça → ajuste com `text_style`, `layer_move`, `align`.
   Use posições por âncora (`anchor` + `marginPct`) e confirme coordenadas na grade do preview.
5. **Revisar**: `render_preview` + `layout_check`. Corrija todos os avisos (sobreposição do produto, contraste, texto pequeno). Repita até limpo; no máximo 4 iterações, depois pergunte ao usuário.
6. **Fechar**: `transaction_commit` → `design_save`. Mostre o preview final e diga o que foi feito em poucas linhas. Pergunte se quer exportar (preset padrão: `pdp-web-1000`).

## Boas práticas de PDP
- Uma mensagem por imagem: 1 título + no máximo 1 bloco de apoio.
- Título legível no celular: altura de letra ≥ 6% do lado do canvas.
- Contraste alto: texto claro em fundo escuro ou vice-versa; use sombra suave se o fundo for irregular.
- Margem mínima de 5% do lado.
- Repita a assinatura visual da referência (ex.: filete de acento) em todas as imagens do projeto para consistência.
- Em projetos com várias fotos, depois da primeira aprovada, salve como template (`templates_save`) e aplique nas outras.

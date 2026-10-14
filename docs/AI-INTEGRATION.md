# Integração com IA (MCP)

## Princípios

1. **O app é a mão, a IA é o cérebro.** O app não chama LLM. Claude Code/Claude Desktop conectam via
   MCP e usam as ferramentas. A visão (entender foto e referências) é feita pelo modelo.
2. **A IA precisa ver.** Toda sessão de design é um loop: *ver → planejar → aplicar → ver de novo → corrigir*.
   Por isso `render_preview` devolve imagem e é barato.
3. **A foto do produto é sagrada.** A camada base é travada. A IA trabalha em camadas por cima, respeita
   `protectedZones` e só aplica ajustes globais na foto se o usuário pedir.
4. **Tudo que a IA faz é desfazível em um passo.** A IA abre uma transação; o usuário dá Ctrl+Z uma vez.
5. **Ferramentas geradas do Command Registry** + um pequeno conjunto de ferramentas extras (projeto,
   visão, referências, exportação) em `src/main/mcp/extra-tools.ts`.

## Conexão

- Transporte: **Streamable HTTP** em `http://127.0.0.1:47800/mcp` (porta configurável).
- Auth: `Authorization: Bearer <token>`; token em `%APPDATA%/colirium-canva/mcp-token`.
- O app precisa estar aberto. (Fase futura: shim stdio `colirium-mcp` que abre o app em modo headless.)
- Registro no Claude Code: `.mcp.json` na raiz do repo, ou `claude mcp add --transport http colirium http://127.0.0.1:47800/mcp --header "Authorization: Bearer <token>"`.
- Claude Desktop: via ponte `mcp-remote` apontando para a mesma URL (documentar no README do app).

## Coordenadas para a IA

- Unidade = pixel do canvas do design. `design_get` sempre informa `canvas.width/height`.
- Todas as ferramentas de posição aceitam, além de `x/y`, um atalho de **âncora**:
  `{ "anchor": "top-center", "marginPct": 5 }` (âncoras: top-left … bottom-right, center, e
  `"right-column"`, `"left-column"`, `"top-band"`, `"bottom-band"`).
- `render_preview` aceita `grid: true` para desenhar uma grade com rótulos de coordenadas por cima
  (só no preview, nunca no export) — ajuda a IA a posicionar com precisão.

## Catálogo de ferramentas

Nomes no formato `namespace_verbo`. (A) = gerada do registry; (X) = extra.

### Projetos e fotos (X)
- `projects_list` · `projects_create {name, notes?}` · `projects_get {projectId}`
- `photos_import {projectId, filePaths[]}` — aceita RAW
- `design_open {projectId, photoId}` — abre na janela principal (o usuário vê) ou headless
- `design_get {fields?}` — JSON do design aberto (camadas resumidas por padrão)
- `design_save`

### Visão (X)
- `render_preview {maxSize=1024, grid?, highlightLayerIds?, showProtectedZones?}` → **imagem** PNG
- `render_layer_bounds` → lista de caixas de cada camada (para checar sobreposição)
- `layout_check` → avisos: texto sobre zona protegida, texto fora do canvas, contraste baixo
  (WCAG sobre a média dos pixels por trás), elementos colidindo, texto pequeno demais para mobile
- `zones_set_protected {zones[]}` — a IA marca onde está o produto após olhar o preview

### Transação (X)
- `transaction_begin {label}` → `transactionId` · `transaction_commit` · `transaction_rollback`

### Edição (A) — exemplos; a lista real vem do registry
- `text_add`, `text_update`, `text_style`
- `shape_add`, `icon_add`, `image_add`
- `blocks_add {block: 'benefit-column' | 'testimonial' | ..., params}`
- `layer_move`, `layer_resize`, `layer_reorder`, `layer_delete`, `layer_duplicate`, `layer_group`
- `align {layerIds, to: 'canvas'|'selection', axis, mode}` · `distribute`
- `base_adjust` (só com pedido explícito) · `canvas_resize` · `canvas_crop` · `canvas_extend`
- `history_undo`, `history_redo`

### Referências de estilo (X)
- `references_list` → style sets
- `references_get_images {styleSetId, maxSize=1024}` → **imagens** (para o modelo analisar)
- `references_get_style {styleSetId}` → `style.json`
- `references_save_style_profile {styleSetId, profile}` — a IA grava o que aprendeu
- `references_add_images {styleSetId, filePaths[]}`

### Templates e exportação (X)
- `templates_list` · `templates_save {name, slots}` · `templates_apply {templateId, slotValues}`
- `export_presets_list` · `export_photo {preset | custom}` · `export_project {presetId}`

### Recursos de catálogo (MCP resources)
- `colirium://fonts` — fontes disponíveis
- `colirium://icons?query=` — busca de ícones por palavra (pt e en: "algodão" → `plant`, `cotton`...)
- `colirium://textures`
- `colirium://blocks` — blocos e seus parâmetros

## Fluxo típico da IA ("faça um PDP no estilo da referência")

1. `references_get_style` (se não existir, `references_get_images` → analisar → `references_save_style_profile`)
2. `design_open` → `render_preview {grid:true}` → `zones_set_protected`
3. `transaction_begin`
4. Adicionar título, blocos, ícones conforme o style profile e as notas do projeto
5. `render_preview` → `layout_check` → corrigir → repetir até sem avisos
6. `transaction_commit` → `design_save` → resumir ao usuário o que fez

## Requisitos de qualidade das ferramentas

- Descrições claras com exemplos; todo campo zod com `.describe()`.
- Respostas curtas: IDs criados + caixas resultantes. Nunca devolver o documento inteiro sem pedir.
- Erros acionáveis e listando opções válidas.
- Teste de contrato: `tests/mcp/` sobe o servidor e chama cada ferramenta com o SDK cliente.

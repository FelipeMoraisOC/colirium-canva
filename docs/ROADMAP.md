# Roadmap

Trabalhe uma fase por vez. Cada item: implementado + testado + doc atualizada → marcar `[x]`.
Não pule para a próxima fase com itens obrigatórios abertos.

## Fase 0 — Fundação e spikes

- [ ] Scaffold: electron-vite + React + TS strict + Tailwind + Radix + pnpm + ESLint/Prettier + Vitest + Playwright
- [ ] Estrutura de pastas conforme `ARCHITECTURE.md`; `src/core` sem dependência de Electron (regra de lint)
- [ ] Contratos IPC tipados (`src/shared/ipc.ts`) + preload mínimo
- [ ] **Spike RAW:** testar candidatos de `RawDecoder` (D8) com pelo menos 3 RAWs reais (pedir ao usuário a câmera e arquivos). Critérios: abre CR3/NEF/ARW/DNG, cores corretas, < 5 s por foto de 24 MP, funciona empacotado no Windows. Registrar a escolha em DECISIONS.md
- [ ] **Spike export:** Konva offscreen a 6000×4000 → PNG → sharp → WebP; medir tempo e memória
- [ ] `pnpm build:win` gera instalador que abre no Windows (sharp com `asarUnpack`)

**Aceite:** app abre, mostra uma janela vazia, testes rodam no CI local, instalador funciona.

## Fase 1 — Núcleo do documento e comandos

- [ ] Schemas zod do `DOCUMENT-MODEL.md` + factories + migrações (infra)
- [ ] Command Registry, CommandBus, histórico com patches Immer, transações
- [ ] Paleta de comandos `Ctrl+K` lista o registry
- [ ] Testes unitários de registry/bus/histórico (incluindo transação = 1 undo)

## Fase 2 — Projetos e importação

- [ ] Tela inicial: lista de projetos (cards com miniatura), criar/renomear/excluir, escolher raiz de projetos
- [ ] Importar fotos (arrastar e soltar / diálogo), JPEG/PNG/WebP/TIFF + RAW pelo decoder escolhido
- [ ] Geração de master/proxy/thumb em worker, barra de progresso
- [ ] Faixa de fotos do projeto no editor (trocar de foto mantém cada design)

## Fase 3 — Editor básico (paridade "editor de imagem")

- [ ] Canvas: zoom (scroll/Ctrl±, ajustar à tela, 100%), pan (espaço+arrastar), régua opcional
- [ ] Seleção, múltipla seleção, Transformer (mover, redimensionar, girar), setas para nudge
- [ ] Guias inteligentes e snapping (centro/bordas do canvas e de outras camadas)
- [ ] Painel de camadas: reordenar, ocultar, travar, renomear, agrupar
- [ ] Texto: adicionar, editar inline, fonte, peso, tamanho, cor, contorno, sombra, espaçamento, altura de linha, alinhamento, caixa alta
- [ ] Formas: retângulo, elipse, linha, estrela, cantos arredondados
- [ ] Ícones: navegador com busca (Tabler + Lucide), cor, espessura
- [ ] Imagens extras: importar logo/selo como camada
- [ ] Foto base: recorte (livre e proporções 1:1, 4:5, 3:4, 16:9), girar 90°, endireitar, espelhar
- [ ] Ajustes: exposição, brilho, contraste, saturação, temperatura, matiz, nitidez, vinheta (não destrutivos)
- [ ] Copiar/colar/duplicar, excluir, desfazer/refazer, atalhos padrão (Ctrl+Z/Y/C/V/D, Delete)
- [ ] Salvamento automático (debounce) + indicador de "salvo"

## Fase 4 — Exportação

- [ ] Diálogo de exportação: PNG/WebP/JPEG, modos de tamanho (original, maior lado, L×A, %, KB alvo), qualidade, strip de metadados, preview do tamanho estimado
- [ ] Presets (criar/editar) + presets padrão de PDP
- [ ] Exportação em lote do projeto
- [ ] Teste golden: preview vs export com diferença de pixels < limiar

## Fase 5 — Servidor MCP (IA operando o app)

- [ ] Servidor Streamable HTTP + token + `.mcp.json`
- [ ] Geração automática de ferramentas a partir do registry
- [ ] Ferramentas extras: projetos, `render_preview` (com grade), transações, `layout_check`, zonas protegidas
- [ ] Janela de render oculta (modo headless) para designs não abertos
- [ ] Indicador na UI quando a IA está editando + destaque das camadas criadas pela IA
- [ ] Testes de contrato MCP
- [ ] Teste de aceitação manual: pedir ao Claude Code "coloque o título 'CONFORTO NO SEU RITMO' no canto inferior esquerdo com filete vermelho" e conferir

## Fase 6 — Estilo PDP: efeitos, blocos e referências

- [ ] Máscara de textura em texto (efeito desgastado) + biblioteca de texturas
- [ ] Gradientes, blend modes, sombra/brilho externo
- [ ] Blocos: headline-underline, benefit-badge, benefit-column, testimonial, color-swatch-label, spec-callout
- [ ] Biblioteca de referências (style sets): UI para criar, adicionar imagens, ver/editar `style.json`
- [ ] Ferramentas MCP de referências; seed com `references/artrock-gasoline-premium`
- [ ] Templates com slots e coordenadas normalizadas; aplicar template em várias fotos

## Fase 7 — Extras (opcional, por prioridade do usuário)

- [ ] Remoção de fundo local (ONNX, ex.: `@imgly/background-removal-node`) → também detecta a zona do produto automaticamente
- [ ] Expansão de canvas com preenchimento (cor amostrada / borrão)
- [ ] Texto em arco, texto em caixa com auto-ajuste
- [ ] Chat embutido no app usando a API da Anthropic, reaproveitando as mesmas ferramentas
- [ ] Shim stdio `colirium-mcp` que abre o app headless sob demanda
- [ ] Render em tiles para imagens > 16k px

# Arquitetura — colirium-canva

## Visão geral

```
┌──────────────────────────── Electron ────────────────────────────┐
│                                                                  │
│  Renderer (React)                    Main (Node)                 │
│  ┌──────────────────────┐   IPC     ┌─────────────────────────┐  │
│  │ UI (painéis, toolbar)│◀────────▶│ ProjectService (disco)  │  │
│  │ Canvas (react-konva) │  tipado   │ ImageService (sharp)    │  │
│  │ DocumentStore        │           │ RawDecoder              │  │
│  │   └ CommandBus ──────┼──┐        │ ExportService           │  │
│  └──────────────────────┘  │        │ ReferenceLibrary        │  │
│                            │        │ McpServer (HTTP local)  │◀─┼── Claude Code /
│  Render window (oculta)    │        └───────────┬─────────────┘  │   Claude Desktop
│  ┌──────────────────────┐  │                    │                │
│  │ Konva Stage offscreen│◀─┘  snapshots/export  │                │
│  └──────────────────────┘◀──────────────────────┘                │
│                                                                  │
│  src/core  (TS puro: schemas, comandos, layout, histórico) ──────┤ usado pelos dois lados
└──────────────────────────────────────────────────────────────────┘
```

**Fonte da verdade do documento aberto:** o `DocumentStore` no renderer principal. O servidor MCP no main
envia comandos para ele via IPC (`command:execute`) e recebe o resultado. Assim, UI e IA nunca
divergem e o usuário vê a IA trabalhando ao vivo no canvas.

Se nenhum design estiver aberto na janela principal, o main abre o design pedido na **janela de render
oculta**, que tem o mesmo `DocumentStore` + `CommandBus` (modo headless). Ao terminar, salva em disco.

## Estrutura de pastas

```
colirium-canva/
├─ CLAUDE.md
├─ .mcp.json                      # registra o servidor MCP do app no Claude Code
├─ .claude/                       # slash commands, skill pdp-designer, settings
├─ docs/
├─ references/                    # referências de estilo de exemplo (seed)
├─ resources/
│  ├─ fonts/                      # fontes OFL embarcadas
│  ├─ icons/                      # tabler + lucide (SVG)
│  ├─ textures/                   # texturas grunge/papel/concreto para máscaras e fundos
│  └─ bin/                        # binários opcionais (ex.: dcraw_emu.exe) — ver D8
├─ src/
│  ├─ core/                       # TS PURO — sem React/Electron/Node
│  │  ├─ document/                # schemas zod, tipos, factories, migrations/
│  │  ├─ commands/                # registry.ts, bus.ts, history.ts, tipos de Command
│  │  ├─ layout/                  # âncoras, alinhamento, snapping, bounding boxes, zonas protegidas
│  │  ├─ text/                    # medição/quebra de linha (interface; impl. no renderer)
│  │  └─ export/                  # presets de exportação, cálculo de tamanhos
│  ├─ features/                   # uma pasta por feature (fatia vertical)
│  │  ├─ text/
│  │  │  ├─ commands.ts           # addText, updateTextStyle, ...
│  │  │  ├─ commands.test.ts
│  │  │  ├─ TextPanel.tsx         # UI de propriedades
│  │  │  ├─ TextNode.tsx          # renderização Konva
│  │  │  └─ index.ts              # registerFeature(): registra comandos, painéis, atalhos
│  │  ├─ shapes/ icons/ images/ adjustments/ crop/ layers/ blocks/ templates/ export/ references/ ...
│  ├─ main/
│  │  ├─ index.ts                 # bootstrap Electron
│  │  ├─ services/                # project, image, raw, export, references, fonts, settings
│  │  ├─ mcp/                     # server.ts, tools-from-registry.ts, extra-tools.ts, auth.ts
│  │  └─ ipc/                     # handlers por canal
│  ├─ preload/index.ts            # expõe window.api tipado (mínimo)
│  ├─ renderer/
│  │  ├─ app/                     # shell, rotas (Home de projetos / Editor)
│  │  ├─ editor/                  # Stage, Transformer, guias, zoom/pan, seleção
│  │  ├─ store/                   # DocumentStore (Immer), UiStore (Zustand)
│  │  ├─ render-window/           # entrada da janela oculta de render
│  │  └─ i18n/pt-BR.ts
│  └─ shared/                     # ipc.ts (contratos de canais), constantes
└─ tests/
   ├─ e2e/
   └─ golden/                     # designs JSON + PNG esperados para testes de render
```

## Command Registry (o coração extensível)

```ts
// src/core/commands/types.ts
export interface CommandDef<P extends z.ZodTypeAny, R = void> {
  id: string;                 // 'text.add' — namespace.verbo
  title: string;              // pt-BR, aparece na paleta Ctrl+K
  description: string;        // inglês ou pt, CLARO — vira a descrição da ferramenta MCP
  params: P;                  // schema zod; .describe() em cada campo (vira doc para a IA)
  exposeToAi?: boolean;       // default true
  mutates?: boolean;          // default true; false = consulta (não entra no histórico)
  run: (ctx: CommandContext, params: z.infer<P>) => R;  // puro; ctx.doc é draft Immer
}
```

- `registry.register(def)` é chamado no `index.ts` de cada feature.
- `CommandBus.execute(id, params, { source: 'ui' | 'ai' | 'shortcut', transactionId? })`
  valida com zod, roda dentro de `produceWithPatches`, empilha no histórico.
- Comandos com mesmo `transactionId` viram **um** passo de desfazer (a IA abre/fecha transações).
- Comandos da IA são marcados `source: 'ai'` e aparecem destacados no painel de histórico.
- O servidor MCP gera as ferramentas a partir do registry (`zod-to-json-schema`). Nome MCP = id com
  `.` trocado por `_` (`text_add`).

Erros de comando retornam mensagens **acionáveis** (ex.: "layerId 'abc' não existe. Layers: [...]") porque
a IA lê essas mensagens e se corrige.

## Renderização

- O Konva Stage mostra o documento com `scale = zoom`. O documento está sempre em pixels da foto.
- A foto base é desenhada a partir de um **proxy** (máx. 4096 px no maior lado) para fluidez;
  a exportação usa o **master** em resolução total.
- Ajustes de imagem (brilho, contraste, saturação, exposição, temperatura, nitidez) usam filtros Konva no
  preview e são reaplicados de forma equivalente no export. Teste golden garante paridade.
- Efeitos de texto: preenchimento sólido/gradiente, contorno, sombra, espaçamento entre letras,
  altura de linha, caixa alta, **máscara de textura** (efeito "desgastado/grunge" das referências),
  texto em arco (fase posterior).

## Exportação

1. Render window carrega o design com o master em resolução total → `stage.toCanvas({ pixelRatio })`.
2. PNG lossless enviado ao main.
3. `sharp`: resize (Lanczos3, `withoutEnlargement`), sRGB, encode, strip de metadados (opcional).
4. Modos de tamanho: original · maior lado N px · L×A exato (cover/contain) · porcentagem ·
   **tamanho alvo em KB** (busca binária de qualidade WebP/JPEG).
5. Presets iniciais: `PDP Web 1000×1000 WebP q82`, `PDP Web 1200px WebP q80`, `Original PNG`, `Original WebP q90`.
6. Exportação em lote: todas as fotos do projeto com um preset → `exports/<preset>/`.

## Importação

Formatos: JPEG, PNG, WebP, TIFF, AVIF, BMP (sharp); HEIC (via `heic-decode`, opcional);
RAW: CR2, CR3, NEF, ARW, RAF, ORF, RW2, DNG, PEF, SRW (via `RawDecoder`).

Ao importar: copia o arquivo original para `photos/<id>/original.<ext>` (intocado) → gera
`master.png` (16 bits, sRGB, orientação EXIF aplicada) → gera `proxy.webp` e `thumb.webp` →
cria `design.json` vazio com a foto como camada base travada.

## Fontes embarcadas (sugestão inicial, todas OFL)

Bebas Neue, Anton, Oswald, Barlow Condensed, Roboto Condensed, Montserrat, Inter, Archivo Black,
Teko, Big Shoulders Display, Permanent Marker, Special Elite. As referências da Art Rock usam
condensadas em caixa alta com textura desgastada: Bebas Neue/Anton + máscara grunge reproduzem bem.

## Segurança local

- MCP escuta só em `127.0.0.1`. Token aleatório gerado na primeira execução e salvo em
  `%APPDATA%/colirium-canva/mcp-token`; o header `Authorization: Bearer <token>` é obrigatório.
- Ferramentas de arquivo só leem/escrevem dentro da raiz de projetos e da biblioteca de referências.
- `contextIsolation: true`, `sandbox: true` no renderer, CSP restrita.

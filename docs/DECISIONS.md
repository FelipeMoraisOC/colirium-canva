# Decisões de arquitetura (ADR resumido)

Cada decisão: contexto → decisão → consequências. Novas decisões vão no fim, numeradas.

## D1 — Linguagem: TypeScript em tudo

**Contexto.** O app vai crescer com features escritas majoritariamente por IA (Claude Code). O dono não é
desenvolvedor em tempo integral.
**Decisão.** TypeScript strict em 100% do código (main, renderer, core, servidor MCP).
**Consequências.** Uma linguagem só, o ecossistema com mais exemplos de treino para IAs, tipos que
funcionam como "trilhos" para o código gerado (o compilador pega erros da IA), o SDK oficial de MCP é
TypeScript. Perde-se um pouco de desempenho bruto comparado a Rust/C++, irrelevante para fotos únicas.

## D2 — Runtime desktop: Electron (não Tauri, não Qt)

**Contexto.** Precisa rodar bem no Windows, ter canvas interativo de qualidade, processar imagens grandes
e hospedar um servidor MCP.
**Decisão.** Electron + electron-vite.
**Alternativas.** Tauri 2 (mais leve, mas exige Rust no backend → duas linguagens, WebView2 varia por
máquina); Qt/PySide (UI menos fluida para algo estilo Canva, ecossistema de componentes pior).
**Consequências.** Instalador maior (~150 MB), aceitável para uso pessoal. Chromium fixo = renderização
de texto/canvas idêntica entre preview e exportação. Node no main permite `sharp` e o SDK MCP nativo.

## D3 — Canvas: Konva (react-konva)

**Decisão.** Konva para cena 2D (camadas, Transformer, hit-test, filtros, export em alta resolução).
**Alternativas.** Fabric.js (bom, mas API imperativa conflita com estado declarativo React/Immer);
PixiJS (WebGL, ótimo desempenho, texto/edição mais trabalhosos).
**Consequências.** A cena Konva é **derivada** do documento (React renderiza o documento). Edição de texto
inline usa textarea sobreposta (padrão conhecido do Konva).

## D4 — Documento como JSON + Immer patches; tudo é Comando

**Decisão.** O design de cada foto é um JSON validado por zod. Toda mutação é um Comando registrado,
executado com `produceWithPatches`. Histórico guarda patches/inversos.
**Consequências.** Undo/redo grátis, IA e UI usam o mesmo caminho, arquivos legíveis por humanos e IAs,
migrações versionadas. Uma sequência de comandos da IA vira **uma** transação (um Ctrl+Z desfaz tudo).

## D5 — Pipeline de exportação: render no Chromium, encode no sharp

**Decisão.** A composição é renderizada em resolução total por um Konva Stage fora da tela (janela oculta
de render) → buffer PNG → processo main → `sharp` faz redimensionamento (Lanczos3), conversão sRGB,
encode PNG/WebP/JPEG e remoção de metadados.
**Consequências.** O que você vê é o que exporta (mesmo motor de texto). Limite de canvas do Chromium
(~16384 px por lado, ~268 MP de área) cobre qualquer câmera atual; acima disso, renderizar em tiles
(ver ROADMAP, item opcional).

## D6 — IA via MCP, sem chave de API no app

**Decisão.** O app hospeda um servidor **MCP Streamable HTTP** em `127.0.0.1:47800/mcp`, protegido por
token local. Claude Code / Claude Desktop são o "cérebro"; o app é a "mão".
**Consequências.** A visão (entender a foto e as referências) é feita pelo modelo que já está na
assinatura do usuário. O app não precisa de chave da Anthropic nem custo extra. As ferramentas são
geradas a partir do Command Registry, então cada feature nova já nasce disponível para a IA.
Um chat embutido no app (via API) fica como fase opcional futura, reutilizando as mesmas ferramentas.

## D7 — Projetos como pastas no disco

**Decisão.** Um projeto = uma pasta com `project.json`, fotos originais, designs JSON e exports.
Sem banco de dados. Índice de projetos recentes em `%APPDATA%/colirium-canva/settings.json`.
**Consequências.** Backup = copiar pasta. Git, OneDrive e o próprio Claude Code conseguem ler os projetos.

## D8 — RAW de câmera atrás de uma interface

**Decisão.** `RawDecoder` é uma interface no main com implementações trocáveis. A escolha concreta é
feita no **spike da Fase 0** (ver ROADMAP), testando com arquivos reais do usuário.
Candidatos, em ordem de preferência:
1. LibRaw compilado para WebAssembly (pacote npm `libraw-wasm` ou equivalente) rodando num worker;
2. binário `dcraw_emu.exe` do LibRaw empacotado em `resources/bin/`;
3. fallback: extrair o JPEG de pré-visualização embutido no RAW via `exiftool-vendored`.
**Consequências.** O app nunca fica preso a uma biblioteca. Ao importar um RAW, é gerado um **master**
(PNG 16 bits, sRGB) dentro do projeto; o arquivo RAW original fica intocado ao lado.

## D9 — Ícones e fontes livres embarcados

**Decisão.** Ícones: **Tabler Icons** (MIT, ~5000 SVGs) + **Lucide** (ISC). Fontes: seleção de Google
Fonts com licença OFL embarcada (ver `ARCHITECTURE.md`) + fontes instaladas no Windows + fontes do
usuário em uma pasta própria.
**Consequências.** Nenhuma limitação de "plano free". Tudo offline.

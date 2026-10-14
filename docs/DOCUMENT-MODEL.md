# Modelo de documento

Todos os schemas vivem em `src/core/document/schema.ts` (zod). Este arquivo é a descrição humana; se
divergir do código, o código vence e este arquivo deve ser corrigido.

## Pastas em disco

```
<raiz de projetos>/                         # configurável; ex.: D:\00 - ArtRock\colirium-projetos
└─ gasoline-premium/                        # slug do projeto
   ├─ project.json
   ├─ photos/
   │  └─ ph_01J9.../
   │     ├─ original.CR3                    # arquivo da câmera, NUNCA alterado
   │     ├─ master.png                      # decodificado, 16 bits sRGB, resolução total
   │     ├─ proxy.webp                      # ≤ 4096 px, para o editor
   │     ├─ thumb.webp                      # ≤ 512 px
   │     └─ design.json                     # a composição desta foto
   ├─ assets/                               # imagens extras usadas no projeto (logos, selos)
   └─ exports/
      └─ pdp-web-1000/                      # uma pasta por preset
```

Biblioteca global de referências: `%APPDATA%/colirium-canva/references/<styleSetId>/` (ver abaixo).

## project.json

```jsonc
{
  "schemaVersion": 1,
  "id": "prj_01J9...",
  "name": "Gasoline Premium",
  "createdAt": "2026-10-14T00:00:00Z",
  "updatedAt": "...",
  "styleSetId": "artrock-gasoline-premium",   // referência de estilo padrão do projeto (opcional)
  "photos": [
    { "id": "ph_01J9...", "name": "modelo-oficina", "order": 0, "width": 6000, "height": 4000 }
  ],
  "exportPresetIds": ["pdp-web-1000"],
  "notes": "Texto livre: público, tom de voz, cores do produto, argumentos de venda."
}
```

## design.json

```jsonc
{
  "schemaVersion": 1,
  "photoId": "ph_01J9...",
  "canvas": {
    "width": 6000, "height": 4000,           // px; inicia = foto, pode mudar (recorte/expansão)
    "background": { "type": "color", "value": "#111111" }
  },
  "base": {                                   // a foto do produto — tratada à parte das camadas
    "src": "master.png",
    "transform": { "x": 0, "y": 0, "scale": 1, "rotation": 0, "flipX": false, "flipY": false },
    "crop": { "x": 0, "y": 0, "width": 6000, "height": 4000 },
    "adjustments": { "exposure": 0, "brightness": 0, "contrast": 0, "saturation": 0,
                     "temperature": 0, "tint": 0, "sharpness": 0, "vignette": 0 },
    "locked": true
  },
  "protectedZones": [                          // áreas que a IA não deve cobrir (o produto)
    { "id": "pz_1", "label": "camiseta", "x": 1500, "y": 800, "width": 3000, "height": 3000 }
  ],
  "layers": [ /* Layer[], índice 0 = mais ao fundo */ ],
  "guides": [ { "axis": "x", "pos": 3000 } ],
  "meta": { "lastEditedBy": "ui" | "ai", "templateId": null }
}
```

## Layer (união discriminada por `type`)

Campos comuns: `id`, `name`, `type`, `x`, `y`, `width`, `height`, `rotation`, `opacity` (0–1),
`blendMode` (`normal|multiply|screen|overlay|soft-light|...`), `visible`, `locked`,
`shadow?` `{color, blur, offsetX, offsetY, opacity}`, `groupId?`, `tags?: string[]`.

| type | campos específicos |
|---|---|
| `text` | `text`, `fontFamily`, `fontWeight`, `fontSize`, `fill` (cor ou gradiente), `stroke?`, `letterSpacing`, `lineHeight`, `align`, `verticalAlign`, `textTransform`, `textureMask?` `{textureId, intensity}`, `autoFit?` (`none|shrink|grow`) |
| `shape` | `shape` (`rect|ellipse|line|polygon|star`), `fill`, `stroke`, `strokeWidth`, `cornerRadius`, `points?` |
| `icon` | `iconSet` (`tabler|lucide|custom`), `iconName`, `color`, `strokeWidth` |
| `image` | `src` (relativo a `assets/` ou biblioteca), `crop?`, `adjustments?` |
| `group` | `children: string[]` (ids) — usado por blocos |

## Blocos (componentes prontos parametrizados)

Blocos são **factories** que criam um `group` com várias camadas. São comandos como qualquer outro
(`blocks.add`). Primeiros blocos, tirados das referências:

- `headline-underline` — título condensado + filete de destaque embaixo (cor de acento)
- `benefit-badge` — ícone dentro de círculo com contorno + rótulo em 1–2 linhas (ex.: "100% ALGODÃO")
- `benefit-column` — N `benefit-badge` empilhados com espaçamento uniforme
- `testimonial` — aspas grandes coloridas + citação + autor
- `color-swatch-label` — nome da cor + filete (para "escolha sua cor")
- `spec-callout` — título grande + linha + subtítulo + legenda (ex.: "Malha fio 30.1 / Toque macio")

## Template

Um design salvo sem a foto base, com **slots** nomeados (`headline`, `subtitle`, `benefit_1`...).
Fica em `%APPDATA%/colirium-canva/templates/<id>.json`. Posições são salvas em **coordenadas
normalizadas** (0–1) para se adaptar a fotos de outros tamanhos.

## Style set (biblioteca de referências)

```
references/artrock-gasoline-premium/
├─ images/ 01.png 02.png ...
└─ style.json
```

```jsonc
{
  "id": "artrock-gasoline-premium",
  "name": "Art Rock — Gasoline Premium",
  "images": ["images/01.png", "..."],
  "palette": { "text": "#F2EADB", "accent": "#D7262B", "dark": "#1A1A1A" },
  "typography": [ { "role": "headline", "fontFamily": "Bebas Neue", "textTransform": "uppercase",
                    "textureMask": "grunge-01" } ],
  "layoutPatterns": ["headline topo centralizado com filete", "coluna de benefícios à direita"],
  "doDont": { "do": ["..."], "dont": ["..."] },
  "notes": "Descrição livre do estilo, escrita pela IA após analisar as imagens."
}
```

`style.json` é escrito pela IA (ferramenta `references_save_style_profile`) e editável na UI.

## Migrações

`src/core/document/migrations/<from>-to-<to>.ts`. Ao abrir um arquivo com `schemaVersion` antigo, aplica
em sequência e salva com backup `design.json.bak-v<from>`.

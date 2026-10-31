# Plano: melhoria da qualidade de impressão / PDF

## Contexto

O PDF atual (`tmp/Plano de Carreira · Graziele Vasconcelos.pdf`) é A4, 2 páginas e
vetorial (texto selecionável, fontes embutidas). A diferença em relação ao
screenshot não é de resolução: é de fidelidade de layout, tipografia e cores.

Diagnóstico principal:

1. A largura útil de uma A4 (~794px) aciona `@media (max-width: 860px)`.
   Isso faz o print usar o layout de tablet:
   - cabeçalho da raia empilhado (`AGORA / NOW` e `1 de 3` em linhas separadas);
   - card de progresso limitado a `max-width: 420px`;
   - layout geral com cara de mobile/tablet.
2. O bloco `@media print` troca azul/índigo/teal/laranja/verde por preto e esconde
   o anel de progresso, as barras das raias, o nó/linha do tempo e as cores de
   prioridade. O PDF perde o `17%` e fica monocromático.
3. O PDF embutiu **Liberation Sans** (fallback do Chrome no Linux). O screenshot
   usa **SF Pro** (macOS). Isso muda bastante a percepção de qualidade.
4. Data, título, URL e `1/2` vêm da opção "Cabeçalhos e rodapés" do navegador.
   CSS não remove isso.
5. `:root[data-theme="dark"]` tem especificidade maior que o `:root` do print.
   Imprimindo no tema escuro, os cards podem sair escuros no meio de uma página
   branca.
6. As regras `max-width` não estão limitadas a `screen`, então podem vazar para o
   print dependendo do navegador/dispositivo. Também não há `@page { size: A4 }`
   explícito.
7. `break-inside: avoid` na raia inteira + gap de 76px deixam a página 2 com muito
   espaço vazio e risco de cabeçalho órfão.

## Decisões aprovadas

- PDF fiel ao site, com cores.
- Fonte Inter empacotada localmente para consistência entre Linux, Android, iOS
  e macOS.
- Mesmo PDF A4 em qualquer device (sem layout específico de mobile).
- O PDF respeita os filtros e a busca ativos.
- Impressão nativa + guia de configuração (sem biblioteca de PDF).
- Alvo de 2 páginas equilibradas para o conteúdo atual.

## Fase 1 — Corrigir a base do CSS de impressão

Arquivo: `src/styles.css`

### 1.1 Escopar breakpoints para tela

Trocar:

```css
@media (max-width: 860px) { ... }
@media (max-width: 620px) { ... }
@media (max-width: 380px) { ... }
```

por:

```css
@media screen and (max-width: 860px) { ... }
@media screen and (max-width: 620px) { ... }
@media screen and (max-width: 380px) { ... }
```

Isso é o maior ganho: o print deixa de herdar o layout de tablet/mobile.

### 1.2 Forçar tema claro e corrigir o dark mode

No `@media print`, trocar o seletor `:root` por:

```css
:root,
:root[data-theme="dark"] {
  color-scheme: light;
  --bg: #ffffff;
  --surface: #ffffff;
  --surface-2: #f7f7f9;
  --ink: #1d1d1f;
  --ink-soft: #4b4b50;
  --ink-faint: #6e6e73;
  --line: #d2d2d7;
  --line-soft: #e5e5ea;
  --shadow: none;
  --shadow-hover: none;
  --nav-bg: #ffffff;
  --blue: #0066cc;
  --blue-hover: #0066cc;
  --blue-soft: #e8f1fb;
  --indigo: #4b49c8;
  --indigo-soft: #eeedfb;
  --teal: #0f766e;
  --teal-soft: #e6f4f4;
  --green: #15803d;
  --orange: #b45309;
}
```

### 1.3 Página e cores de impressão

```css
@page {
  size: A4 portrait;
  margin: 14mm;
}

body {
  min-height: auto;
}
```

Aplicar `print-color-adjust: exact` nos elementos pequenos que carregam cor:

```css
.badge,
.tag,
.item-id,
.item-check,
.lane-bar span,
.item-card::before {
  -webkit-print-color-adjust: exact;
  print-color-adjust: exact;
}
```

### 1.4 Restaurar o layout fiel

Hero em duas colunas, com anel de progresso visível:

```css
.hero {
  padding: 0 0 16pt;
  background: none;
}

.hero-glow {
  display: none;
}

.hero-grid {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 64mm;
  align-items: end;
  gap: 8mm;
}

.hero h1 {
  max-width: 14ch;
  font-size: 28pt;
  line-height: 1.02;
}

.hero-role {
  margin-top: 8pt;
  font-size: 14pt;
}

.hero-summary {
  margin-top: 8pt;
  font-size: 10pt;
  line-height: 1.45;
}

.eyebrow {
  margin-bottom: 8pt;
  font-size: 8pt;
  color: var(--blue);
}

.progress-card {
  display: flex;
  align-items: center;
  gap: 5mm;
  margin-top: 0;
  padding: 5mm;
  border: 1px solid var(--line);
  border-radius: 12pt;
  background: var(--surface);
  box-shadow: none;
  break-inside: avoid;
}

.progress-ring {
  display: block;
  width: 22mm;
  height: 22mm;
}

.ring-label strong {
  font-size: 15pt;
}

.ring-label span {
  font-size: 6pt;
}

.progress-label {
  font-size: 7pt;
  color: var(--ink-faint);
}

.progress-value {
  font-size: 9pt;
}

.progress-value strong {
  font-size: 13pt;
}

.progress-updated {
  font-size: 7.5pt;
}
```

Cabeçalho da raia em duas colunas, com barra colorida:

```css
.timeline {
  display: block;
}

.lane {
  padding-left: 0;
  break-inside: auto;
}

.lane + .lane {
  margin-top: 18pt;
}

.lane-header {
  grid-template-columns: 1fr auto;
  align-items: end;
  gap: 8mm;
  margin-bottom: 8pt;
  break-after: avoid;
  break-inside: avoid;
}

.lane-progress {
  align-items: flex-end;
  gap: 4pt;
  min-width: 30mm;
}

.lane-count {
  font-size: 8pt;
}

.lane-bar {
  display: block;
  width: 30mm;
  height: 3pt;
}

.lane-bar span {
  background: var(--lane);
}

.lane h2 {
  font-size: 20pt;
}
```

Cards compactos, com acento lateral colorido e prioridades coloridas:

```css
.lane-items {
  gap: 6pt;
}

.item-card {
  margin-bottom: 0;
  padding: 10pt 12pt;
  border: 1px solid var(--line);
  border-radius: 10pt;
  background: var(--surface);
  box-shadow: none;
  opacity: 1 !important;
  break-inside: avoid;
  page-break-inside: avoid;
}

.item-card.is-done {
  background: #fafafa;
}

.item-card.is-done .item-title {
  color: var(--ink-soft);
  text-decoration: line-through;
  text-decoration-color: var(--ink-faint);
}

.item-card::before {
  display: block;
  top: 9pt;
  bottom: 9pt;
  left: 0;
  width: 3pt;
  background: var(--lane);
  opacity: 0.5;
}

.item-check {
  width: 12pt;
  height: 12pt;
  border-width: 1pt;
}

.item-check svg {
  width: 7pt;
  height: 7pt;
}

.item-check.is-checked {
  border-color: var(--green);
  background: var(--green);
}

.item-id {
  font-size: 7.5pt;
  padding: 2pt 4pt;
  background: var(--surface-2);
  color: var(--ink-faint);
}

.item-title {
  font-size: 12pt;
  line-height: 1.3;
}

.item-body {
  margin-left: 21pt;
}

.item-meta {
  gap: 4pt;
  margin-top: 6pt;
}

.badge {
  font-size: 7.5pt;
  padding: 2pt 6pt;
}

.badge-type {
  background: var(--blue-soft);
  color: var(--blue);
  border-color: #c7ddf3;
}

.priority-alta {
  color: var(--orange);
}

.priority-media {
  color: var(--blue);
}

.priority-baixa {
  color: var(--ink-faint);
}

.item-tags {
  gap: 4pt;
  margin-top: 5pt;
}

.tag {
  font-size: 7.5pt;
  padding: 2pt 5pt;
  background: var(--surface-2);
  color: var(--ink-soft);
}

.item-notes {
  margin-top: 6pt;
  font-size: 9pt;
  line-height: 1.4;
}

.item-links {
  gap: 10pt;
  margin-top: 6pt;
}

.item-link,
.item-evidence {
  font-size: 8.5pt;
}

.item-link .icon-sm {
  width: 8pt;
  height: 8pt;
}
```

Rodapé com data de atualização:

```css
.site-footer {
  display: block;
  margin-top: 16pt;
  padding-top: 8pt;
  border-top: 1px solid var(--line-soft);
  font-size: 8pt;
}

.footer-links {
  display: flex !important;
  gap: 8pt;
}

.footer-links a,
.footer-print-help {
  display: none !important;
}
```

Links e controle de órfãs:

```css
a {
  color: var(--blue);
  text-decoration: underline;
  text-underline-offset: 2pt;
}

p {
  orphans: 3;
  widows: 3;
}
```

### 1.5 Respeitar os filtros no print

Remover do bloco de print:

```css
.is-hidden {
  display: block !important;
}
```

Adicionar:

```css
.lane.is-empty {
  display: none !important;
}

.lane-empty {
  display: none !important;
}
```

### 1.6 Esconder o que não deve imprimir

Garantir no `@media print`:

```css
.site-nav,
.filters-section,
.hero-actions,
.skip-link,
.icon-button,
.button,
.print-dialog {
  display: none !important;
}
```

## Fase 2 — Tipografia consistente com Inter

Arquivos: `src/fonts/`, `src/styles.css`, `src/template.html`,
`src/build.py`, `src/check.py`

### 2.1 Asset da fonte

- Adicionar `src/fonts/Inter-Variable.woff2` (subset latino, variável, ~100 KB).
- Adicionar `src/fonts/OFL.txt` com a licença SIL Open Font License.
- Se a fonte variável causar problema no PDF, usar estáticos 400/500/600/700.

### 2.2 `@font-face` e stack

```css
@font-face {
  font-family: "Inter";
  font-style: normal;
  font-weight: 100 900;
  font-display: swap;
  src: url("fonts/Inter-Variable.woff2") format("woff2");
}
```

Atualizar `--font`:

```css
--font: "Inter", -apple-system, BlinkMacSystemFont, "SF Pro Display",
  "SF Pro Text", "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
```

Ajustar `letter-spacing` e `line-height` do hero/títulos após comparação visual,
pois Inter tem métricas diferentes de SF Pro.

### 2.3 Preload no template

No `<head>`:

```html
<link rel="preload" href="fonts/Inter-Variable.woff2" as="font"
      type="font/woff2" crossorigin>
```

### 2.4 Build e validação

Em `src/build.py`, copiar a pasta de fontes:

```python
font_dir = SRC / "fonts"
if not font_dir.is_dir():
    raise PlanError("Pasta src/fonts ausente. Adicione a fonte Inter.")
shutil.copytree(font_dir, DIST / "fonts", dirs_exist_ok=True)
```

Em `src/check.py`:

- adicionar `fonts/Inter-Variable.woff2` a `REQUIRED_FILES`;
- validar que o arquivo não está vazio;
- validar que `id="print-dialog"` existe no HTML;
- opcionalmente, validar que `styles.css` contém `@font-face` e `@media print`.

### 2.5 Garantia de carregamento antes do print

O guia de impressão já cria uma pausa natural: o usuário abre o diálogo, lê e
clica em "Abrir impressão". Nesse clique, chamar `window.print()` de forma
síncrona. Não usar `await` antes do `window.print()` no Safari.

## Fase 3 — Guia de impressão

Arquivos: `src/template.html`, `src/app.js`, `src/styles.css`

### 3.1 Diálogo acessível

Adicionar ao final do `<body>`:

```html
<dialog class="print-dialog" id="print-dialog"
        aria-labelledby="print-dialog-title"
        aria-describedby="print-dialog-desc">
  <h2 id="print-dialog-title">Antes de imprimir</h2>
  <p id="print-dialog-desc">
    Ajuste as opções abaixo para o PDF sair com a melhor qualidade.
  </p>
  <ol class="print-dialog-steps">
    <li>Escolha papel <strong>A4</strong>, orientação <strong>retrato</strong> e margens padrão.</li>
    <li>Desmarque <strong>Cabeçalhos e rodapés</strong> para remover data e URL do navegador.</li>
    <li>Marque <strong>Gráficos de fundo</strong> para manter as cores dos badges e raias.</li>
    <li>No celular, use a opção <strong>Salvar como PDF</strong>.</li>
  </ol>
  <p class="print-dialog-summary" id="print-summary" aria-live="polite"></p>
  <label class="print-dialog-skip">
    <input type="checkbox" id="print-guide-skip">
    Não mostrar este guia novamente
  </label>
  <div class="print-dialog-actions">
    <button class="button button-secondary" type="button" id="print-cancel">Cancelar</button>
    <button class="button button-primary" type="button" id="print-confirm">Abrir impressão</button>
  </div>
</dialog>
```

O PDF é sempre claro, mesmo no tema escuro. Os links continuam clicáveis e o
texto continua selecionável.

### 3.2 Lógica em `app.js`

- Substituir o handler atual dos `.print-button` por:
  - se o guia estiver dispensado no `localStorage` (`career-print-guide`),
    chamar `window.print()` direto;
  - caso contrário, abrir o diálogo.
- No clique de "Abrir impressão":
  - salvar a preferência de "não mostrar novamente";
  - fechar o diálogo;
  - chamar `window.print()` de forma síncrona.
- No `beforeprint`, fechar o diálogo se estiver aberto.
- Atualizar o resumo do diálogo:
  - com filtros: `Serão impressos X de Y itens (visão filtrada).`
  - sem filtros: `Serão impressos todos os Y itens do plano.`
  - com 0 visíveis: desabilitar "Abrir impressão" e orientar a ajustar os filtros.
- Fallback: se `showModal` não existir, chamar `window.print()` direto.

### 3.3 Link "Como imprimir" no rodapé

Adicionar um botão discreto com `data-print-help` no `.footer-links` para reabrir
o guia. No print, ele fica oculto junto com os demais links.

### 3.4 Estilos do diálogo

- Largura `min(480px, calc(100% - 32px))`;
- `max-height: calc(100vh - 32px)` e `overflow: auto`;
- `::backdrop` com fundo escuro translúcido;
- botões com `flex-wrap`;
- `display: none !important` no `@media print`.

## Fase 4 — Documentação

Arquivo: `README.md`

- Atualizar a seção de impressão:
  - o PDF respeita filtros e busca;
  - o tema escuro sempre imprime em claro;
  - A4 retrato, 2 páginas no conteúdo atual;
  - desativar cabeçalhos/rodapés e ativar gráficos de fundo;
  - no celular, salvar como PDF;
  - a fonte Inter é empacotada localmente (licença OFL).
- Atualizar a árvore de estrutura para incluir `src/fonts/`.

## Fase 5 — Validação

### Build

```bash
make test
```

### Desktop Chrome (Linux/macOS)

- Tema claro e escuro.
- Sem filtros e com filtro por tipo/tag/busca.
- Gráficos de fundo ligados e desligados.
- Cabeçalhos e rodapés ligados e desligados.
- Conferir:
  - A4 retrato, 2 páginas;
  - `AGORA / NOW` e `1 de 3` lado a lado;
  - anel de progresso com `17%`;
  - cores de raia, prioridade e check;
  - nenhum card cortado;
  - nenhum cabeçalho de raia órfão;
  - página 2 sem o gap exagerado entre NEXT e LATER.

### Mobile

- Android Chrome: Compartilhar → Imprimir → Salvar como PDF.
- iOS Safari: Imprimir → Salvar como PDF.
- Conferir se o layout é o mesmo A4 e se as regras `max-width` não vazaram.
- Se possível, anexar um PDF gerado no celular para comparação.

### Inspeção do PDF

```bash
pdfinfo "plano.pdf"        # 2 páginas, A4
pdffonts "plano.pdf"       # Inter embutida
pdfimages -list "plano.pdf" # sem rasterização
```

- Texto selecionável.
- Comparar o PDF renderizado a 150/300 dpi com o screenshot.

### Regressão de tela

- 1440px, 1024px, 768px e 390px.
- Tema, filtros, busca e botão de impressão.
- Layout da tela não deve mudar além da tipografia Inter.

## Arquivos alterados

| Arquivo | Mudança |
|---|---|
| `src/styles.css` | Refatoração do print, paleta, componentes e estilos do diálogo |
| `src/app.js` | Guia de impressão, estado de filtros e fluxo do `window.print()` |
| `src/template.html` | Preload da fonte, diálogo e link "Como imprimir" |
| `src/build.py` | Copiar `src/fonts` para `dist/fonts` |
| `src/check.py` | Validar fonte, diálogo e arquivos gerados |
| `src/fonts/Inter-Variable.woff2` | Novo asset |
| `src/fonts/OFL.txt` | Licença da fonte |
| `README.md` | Documentação de impressão, fonte e estrutura |

`career.md`, workflow do Pages e Dockerfile não precisam mudar.

## Critérios de aceite

- PDF A4 em 2 páginas para o conteúdo atual.
- Cabeçalhos de raia lado a lado, como no site.
- Anel de progresso e percentual visíveis.
- Cores de raia, prioridade, badge e check preservadas.
- Fonte Inter embutida no PDF, sem fallback para Liberation Sans.
- Impressão no tema escuro sempre em claro.
- PDF reflete os filtros ativos.
- Mesmo resultado no desktop e no celular.
- Nenhum card cortado ou cabeçalho órfão.
- `make test` passando.
- Layout de tela preservado.

## Riscos e limitações

- O cabeçalho/rodapé do navegador não pode ser removido por CSS. O guia resolve
  no Chrome; no Safari do iOS a opção pode não existir. Se isso acontecer, o
  caminho para um PDF 100% limpo é gerar o PDF no CI.
- Impressão colorida gasta mais tinta; o glow do hero será removido no print.
- A Inter muda levemente a tipografia da tela em relação ao SF Pro.
- `@page { size: A4 }` pode ser ignorado por alguns navegadores mobile; o guia
  orienta escolher A4.
- Não há Chrome/Chromium instalado no ambiente atual. A validação visual final
  precisa ser feita no navegador do usuário ou com um headless instalado com
  autorização.

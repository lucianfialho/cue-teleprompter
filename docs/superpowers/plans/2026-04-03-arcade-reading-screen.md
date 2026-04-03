# Arcade Reading Screen Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Transformar a tela de leitura do EatText em experiência full imersão de fliperama — marquee neon, scoreboard 1UP/HI-SCORE/WORDS, scanlines, HUD com pips de velocidade, overlay GAME OVER pixel-art.

**Architecture:** CSS restyling do chrome existente + dois novos elementos HTML (marquee, scoreboard) + módulo `arcade.js` desacoplado que recebe eventos CustomEvent de `app.js`. O canvas e toda a lógica de renderização de `app.js` permanecem intactos.

**Tech Stack:** Vanilla JS ES modules, CSS custom properties, Google Fonts (Press Start 2P), localStorage.

---

## File Map

| Arquivo | Ação | Responsabilidade |
|---|---|---|
| `index.html` | Modify | Adicionar font link, marquee, scoreboard, renomear classes, script arcade.js |
| `style.css` | Modify | Tokens arcade, cabinet border, scanlines, glow, marquee, scoreboard, HUD pips, gameover |
| `app.js` | Modify | Despachar eat:word / eat:article-end / eat:speed-change; atualizar refs DOM renomeadas |
| `arcade.js` | Create | Score, vidas, DOM updates, overlay GAME OVER, localStorage HI-SCORE |

---

## Task 1: Renomear classes no HTML

**Files:**
- Modify: `index.html` (linhas 129–195)

- [ ] **Step 1: Substituir `.prompter-hud` → `.arcade-hud` e `#read-over` → `#arcade-gameover`**

No `index.html`, substituir:

```html
<!-- ANTES linha 134: -->
<div id="read-over" class="read-over" hidden>
  <div class="read-over-card">
    <p class="read-over-label">read over</p>
    <p id="read-over-count" class="read-over-count">1 / 1</p>
    <div class="read-over-actions">
      <button id="btn-next-article" class="read-over-btn ro-next">Next →</button>
      <button id="btn-game-over"    class="read-over-btn ro-exit">Game Over</button>
    </div>
  </div>
</div>

<div id="prompter-hud" class="prompter-hud" aria-hidden="true">
```

Por:

```html
<div id="arcade-gameover" class="arcade-gameover" hidden>
  <div class="arcade-gameover-card">
    <p class="arcade-gameover-label">READ OVER</p>
    <p id="arcade-gameover-count" class="arcade-gameover-count">1 / 1</p>
    <div class="arcade-gameover-actions">
      <button id="btn-next-article" class="arcade-gameover-btn ago-next">NEXT →</button>
      <button id="btn-game-over"    class="arcade-gameover-btn ago-exit">GAME OVER</button>
    </div>
  </div>
</div>

<div id="arcade-hud" class="arcade-hud" aria-hidden="true">
```

- [ ] **Step 2: Substituir o conteúdo interno do HUD**

O HUD atual tem: WPM + speed fill bar + sep + font size + sep + hints. Substituir tudo dentro de `<div id="arcade-hud" class="arcade-hud">` por:

```html
<div id="arcade-hud" class="arcade-hud" aria-hidden="true">

  <div class="ahud-wpm">
    <span id="ahud-wpm-val" class="ahud-wpm-val">280</span>
    <span class="ahud-lbl">WPM</span>
  </div>

  <div class="ahud-speed">
    <div class="ahud-pips" id="ahud-pips">
      <div class="ahud-pip" data-pip="1"></div>
      <div class="ahud-pip" data-pip="2"></div>
      <div class="ahud-pip" data-pip="3"></div>
      <div class="ahud-pip" data-pip="4"></div>
      <div class="ahud-pip" data-pip="5"></div>
      <div class="ahud-pip" data-pip="6"></div>
      <div class="ahud-pip" data-pip="7"></div>
      <div class="ahud-pip" data-pip="8"></div>
      <div class="ahud-pip" data-pip="9"></div>
      <div class="ahud-pip" data-pip="10"></div>
    </div>
    <span class="ahud-lbl">SPEED</span>
  </div>

  <div class="ahud-font">
    <span id="ahud-font-val" class="ahud-font-val">Aa 48</span>
    <span class="ahud-lbl">FONT</span>
  </div>

</div>
```

- [ ] **Step 3: Verificar no browser que o HTML não quebrou (nenhuma funcionalidade ainda)**

Abrir o app, verificar que a tela de input carrega. Console sem erros críticos de elemento não encontrado.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "refactor: rename prompter-hud → arcade-hud, read-over → arcade-gameover in HTML"
```

---

## Task 2: Atualizar refs DOM em app.js

**Files:**
- Modify: `app.js` (linhas 61–79, 798–805, 1110–1131)

- [ ] **Step 1: Atualizar o objeto `ui` (linhas 59–80)**

Substituir as referências antigas:

```js
// ANTES:
hudSpeedFill:    $('hud-speed-fill'),
hudWpm:          $('hud-wpm'),
hudFontVal:      $('hud-font-val'),
// ...
readOver:        $('read-over'),
readOverCount:   $('read-over-count'),
```

Por:

```js
// DEPOIS:
ahudWpm:         $('ahud-wpm-val'),
ahudFontVal:     $('ahud-font-val'),
ahudPips:        $('ahud-pips'),
// ...
arcadeGameover:  $('arcade-gameover'),
arcadeGameoverCount: $('arcade-gameover-count'),
```

- [ ] **Step 2: Atualizar `updateHUD()` (linhas 798–806)**

```js
function updateHUD() {
  const { speed, fontSize } = state.settings;
  if (_hudCache.speed === speed && _hudCache.fontSize === fontSize) return;
  _hudCache.speed    = speed;
  _hudCache.fontSize = fontSize;

  // WPM
  if (ui.ahudWpm) ui.ahudWpm.textContent = calcWPM(speed);

  // Font val
  if (ui.ahudFontVal) ui.ahudFontVal.textContent = `Aa ${fontSize}`;

  // Pips — 10 pips, cores: amarelo (1-3), laranja (4-7), vermelho (8-10)
  if (ui.ahudPips) {
    ui.ahudPips.querySelectorAll('.ahud-pip').forEach(pip => {
      const n = Number(pip.dataset.pip);
      pip.classList.toggle('on', n <= speed);
      pip.classList.toggle('yellow', n <= speed && n <= 3);
      pip.classList.toggle('orange', n <= speed && n >= 4 && n <= 7);
      pip.classList.toggle('red',    n <= speed && n >= 8);
    });
  }
}
```

- [ ] **Step 3: Atualizar `showReadOver()` e `hideReadOver()` (linhas 1110–1118)**

```js
function showReadOver() {
  if (ui.arcadeGameoverCount) {
    ui.arcadeGameoverCount.textContent =
      `${state.articleIndex + 1} / ${state.articles.length}`;
  }
  if (ui.arcadeGameover) ui.arcadeGameover.hidden = false;
  dispatchEvent(new CustomEvent('eat:article-end', {
    detail: { index: state.articleIndex, total: state.articles.length }
  }));
}

function hideReadOver() {
  if (ui.arcadeGameover) ui.arcadeGameover.hidden = true;
}
```

- [ ] **Step 4: Verificar que o overlay GAME OVER ainda aparece ao terminar um artigo**

Testar: iniciar leitura → aguardar fim → confirmar overlay aparece e botões funcionam.

- [ ] **Step 5: Commit**

```bash
git add app.js
git commit -m "refactor: update DOM refs to arcade-hud and arcade-gameover"
```

---

## Task 3: Adicionar Google Fonts, marquee, scoreboard e script arcade.js ao HTML

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Adicionar Google Fonts no `<head>`**

Logo após `<title>EatText</title>`, adicionar:

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
```

- [ ] **Step 2: Adicionar marquee e scoreboard dentro de `#screen-prompter`**

Logo após `<div id="screen-prompter" class="screen">` (linha 129), antes do `<canvas>`, adicionar:

```html
<!-- Arcade marquee -->
<div class="arcade-marquee" aria-hidden="true">
  <span class="arcade-marquee-title">◄ EAT TEXT ►</span>
  <div class="arcade-lives" id="arcade-lives">
    <span class="arcade-life" data-life="1">ᗤ</span>
    <span class="arcade-life" data-life="2">ᗤ</span>
    <span class="arcade-life" data-life="3">ᗤ</span>
  </div>
</div>

<!-- Arcade scoreboard -->
<div class="arcade-scoreboard" aria-hidden="true">
  <div class="arcade-score-block">
    <div class="arcade-score-label">1UP</div>
    <div class="arcade-score-val" id="arcade-score-1up">00000</div>
  </div>
  <div class="arcade-score-block">
    <div class="arcade-score-label">HI-SCORE</div>
    <div class="arcade-score-val arcade-hiscore" id="arcade-score-hi">00000</div>
  </div>
  <div class="arcade-score-block">
    <div class="arcade-score-label">WORDS</div>
    <div class="arcade-score-val" id="arcade-score-words">00000</div>
  </div>
</div>
```

- [ ] **Step 3: Adicionar `arcade.js` após o script `app.js`**

Substituir:

```html
<script type="module" src="/app.js"></script>
```

Por:

```html
<script type="module" src="/app.js"></script>
<script type="module" src="/arcade.js"></script>
```

- [ ] **Step 4: Verificar no browser que os novos elementos aparecem (sem estilo ainda)**

Console sem erros. Os elementos existem no DOM (inspecionar com DevTools).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add arcade marquee, scoreboard, Google Fonts and arcade.js to HTML"
```

---

## Task 4: CSS — tokens arcade, cabinet border, scanlines, glow, progress bar

**Files:**
- Modify: `style.css`

- [ ] **Step 1: Adicionar tokens no `:root` (após tokens existentes)**

No bloco `:root` existente (linha ~1), adicionar:

```css
/* Arcade theme */
--arcade-yellow:  #ffff00;
--arcade-cyan:    #00ffff;
--arcade-magenta: #ff00ff;
--arcade-purple:  #8800ff;
--arcade-bg:      #05010d;
--arcade-font:    'Press Start 2P', 'Courier New', monospace;
```

- [ ] **Step 2: Estilizar `#screen-prompter` como cabinet**

No bloco `#screen-prompter` existente (linha ~357), substituir por:

```css
#screen-prompter {
  background: var(--arcade-bg);
  border: 3px solid var(--arcade-purple);
  box-shadow:
    0 0 0 1px var(--arcade-purple),
    0 0 30px rgba(136, 0, 255, 0.35),
    0 0 80px rgba(136, 0, 255, 0.12),
    inset 0 0 40px rgba(0, 0, 0, 0.7);
  overflow: hidden;
}
```

- [ ] **Step 3: Adicionar scanlines e glow ao canvas via pseudo-elementos**

Após o bloco `#prompter-canvas` existente, adicionar:

```css
/* Scanlines — pointer-events none para não bloquear o canvas */
#prompter-canvas::before {
  content: '';
  position: absolute;
  inset: 0;
  background: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 2px,
    rgba(0, 0, 0, 0.22) 2px,
    rgba(0, 0, 0, 0.22) 4px
  );
  pointer-events: none;
  z-index: 2;
}

/* Glow roxo central */
#prompter-canvas::after {
  content: '';
  position: absolute;
  inset: 0;
  background: radial-gradient(ellipse at center, rgba(80, 0, 120, 0.10) 0%, transparent 70%);
  pointer-events: none;
  z-index: 3;
}
```

Garantir que `#prompter-canvas` tenha `position: relative` (já tem `position: absolute` — os pseudo-elementos precisam de contexto de posicionamento do pai `#screen-prompter` que é relative por ser `position: fixed/absolute`; verificar que funciona, senão adicionar `position: relative` ao canvas).

- [ ] **Step 4: Reestilizar `.progress-bar`**

Localizar o bloco `.progress-bar` (linha ~533) e adicionar/sobrescrever:

```css
.progress-bar {
  /* manter propriedades existentes de posição/tamanho */
  background: var(--arcade-yellow);
  box-shadow: 0 0 8px var(--arcade-yellow), 0 0 20px var(--arcade-yellow);
  z-index: 20;
}
```

- [ ] **Step 5: Verificar no browser**

A tela de leitura deve ter: borda roxa brilhando, fundo quase preto, scanlines visíveis na área do canvas, barra de progresso amarela neon.

- [ ] **Step 6: Commit**

```bash
git add style.css
git commit -m "feat: arcade CSS tokens, cabinet border, scanlines, glow, progress bar neon"
```

---

## Task 5: CSS — marquee e scoreboard

**Files:**
- Modify: `style.css`

- [ ] **Step 1: Adicionar estilos do marquee**

```css
/* ── Arcade Marquee ─────────────────────────────────────────── */
.arcade-marquee {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 16px;
  background: #0d0618;
  border-bottom: 2px solid var(--arcade-yellow);
  font-family: var(--arcade-font);
  position: relative;
  z-index: 15;
}

.arcade-marquee-title {
  font-size: 10px;
  color: var(--arcade-yellow);
  text-shadow: 0 0 8px var(--arcade-yellow), 0 0 20px var(--arcade-yellow);
  letter-spacing: 2px;
}

.arcade-lives {
  display: flex;
  gap: 6px;
  align-items: center;
}

.arcade-life {
  color: var(--arcade-yellow);
  text-shadow: 0 0 6px var(--arcade-yellow);
  font-size: 14px;
  transition: opacity 0.3s;
}

.arcade-life.spent {
  opacity: 0.2;
}
```

- [ ] **Step 2: Adicionar estilos do scoreboard**

```css
/* ── Arcade Scoreboard ──────────────────────────────────────── */
.arcade-scoreboard {
  display: flex;
  justify-content: space-between;
  padding: 6px 16px;
  background: #000;
  border-bottom: 1px solid #222;
  font-family: var(--arcade-font);
  position: relative;
  z-index: 15;
}

.arcade-score-block {
  text-align: center;
}

.arcade-score-label {
  font-size: 7px;
  color: var(--arcade-magenta);
  text-shadow: 0 0 6px var(--arcade-magenta);
  letter-spacing: 1px;
}

.arcade-score-val {
  font-size: 9px;
  color: #fff;
  margin-top: 3px;
  letter-spacing: 1px;
}

.arcade-hiscore {
  color: var(--arcade-yellow);
  text-shadow: 0 0 6px var(--arcade-yellow);
}
```

- [ ] **Step 3: Verificar no browser**

A tela de leitura deve mostrar a marquee amarela no topo e o scoreboard abaixo.

- [ ] **Step 4: Commit**

```bash
git add style.css
git commit -m "feat: arcade marquee and scoreboard CSS"
```

---

## Task 6: CSS — arcade HUD com pips

**Files:**
- Modify: `style.css`

- [ ] **Step 1: Remover estilos antigos do HUD e adicionar os novos**

Remover (ou comentar) todos os blocos: `.prompter-hud`, `.hud-item`, `.hud-item--font`, `.hud-wpm`, `.hud-label`, `.hud-track`, `.hud-fill`, `.hud-val`, `.hud-sep`, `.hud-hints`, `.hud-hint`, `.hud-icon`.

Adicionar:

```css
/* ── Arcade HUD ─────────────────────────────────────────────── */
.arcade-hud {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 20px;
  background: #000;
  border-top: 2px solid var(--arcade-yellow);
  font-family: var(--arcade-font);
  z-index: 15;
  pointer-events: none;
}

.ahud-wpm,
.ahud-speed,
.ahud-font {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

.ahud-wpm-val {
  font-size: 14px;
  color: var(--arcade-cyan);
  text-shadow: 0 0 10px var(--arcade-cyan);
  font-variant-numeric: tabular-nums;
}

.ahud-font-val {
  font-size: 9px;
  color: var(--arcade-magenta);
  text-shadow: 0 0 8px var(--arcade-magenta);
}

.ahud-lbl {
  font-size: 6px;
  color: #444;
  letter-spacing: 1px;
}

/* Pips de velocidade */
.ahud-pips {
  display: flex;
  gap: 3px;
}

.ahud-pip {
  width: 8px;
  height: 8px;
  border: 1px solid #333;
  background: transparent;
  transition: background 0.1s, box-shadow 0.1s;
}

.ahud-pip.on.yellow {
  background: var(--arcade-yellow);
  border-color: var(--arcade-yellow);
  box-shadow: 0 0 4px var(--arcade-yellow);
}

.ahud-pip.on.orange {
  background: #ff8800;
  border-color: #ff8800;
  box-shadow: 0 0 4px #ff8800;
}

.ahud-pip.on.red {
  background: #ff2200;
  border-color: #ff2200;
  box-shadow: 0 0 4px #ff2200;
}

/* Mobile */
@media (max-width: 480px) {
  .ahud-wpm-val { font-size: 11px; }
  .ahud-pips { gap: 2px; }
  .ahud-pip  { width: 6px; height: 6px; }
}
```

- [ ] **Step 2: Verificar no browser**

HUD na base da tela: WPM em cyan, 10 pips de velocidade, font size em magenta. Mudar velocidade (swipe ↕) deve acender/apagar pips.

- [ ] **Step 3: Commit**

```bash
git add style.css
git commit -m "feat: arcade HUD CSS with speed pips"
```

---

## Task 7: CSS — overlay GAME OVER

**Files:**
- Modify: `style.css`

- [ ] **Step 1: Remover estilos antigos do read-over**

Remover (ou comentar) todos os blocos: `.read-over`, `.read-over[hidden]`, `.read-over-card`, `.read-over-label`, `.read-over-count`, `.read-over-actions`, `.read-over-btn`, `.ro-next`, `.ro-exit`.

- [ ] **Step 2: Adicionar estilos do overlay GAME OVER**

```css
/* ── Arcade GAME OVER overlay ───────────────────────────────── */
.arcade-gameover {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.95);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 50;
  font-family: var(--arcade-font);
}

.arcade-gameover[hidden] {
  display: none;
}

.arcade-gameover-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 32px 24px;
}

.arcade-gameover-label {
  font-size: 18px;
  color: var(--arcade-yellow);
  text-shadow: 0 0 12px var(--arcade-yellow), 0 0 30px var(--arcade-yellow);
  animation: arcade-blink 1s steps(1) infinite;
  letter-spacing: 3px;
}

@keyframes arcade-blink {
  0%, 100% { opacity: 1; }
  50%       { opacity: 0; }
}

.arcade-gameover-count {
  font-size: 10px;
  color: #fff;
  letter-spacing: 2px;
}

.arcade-gameover-actions {
  display: flex;
  flex-direction: column;
  gap: 12px;
  align-items: center;
}

.arcade-gameover-btn {
  font-family: var(--arcade-font);
  font-size: 9px;
  padding: 12px 24px;
  background: transparent;
  cursor: pointer;
  letter-spacing: 2px;
  transition: box-shadow 0.15s;
}

.ago-next {
  border: 2px solid var(--arcade-yellow);
  color: var(--arcade-yellow);
}

.ago-next:hover,
.ago-next:active {
  box-shadow: 0 0 12px var(--arcade-yellow), 0 0 30px var(--arcade-yellow);
}

.ago-exit {
  border: 2px solid var(--arcade-magenta);
  color: var(--arcade-magenta);
}

.ago-exit:hover,
.ago-exit:active {
  box-shadow: 0 0 12px var(--arcade-magenta), 0 0 30px var(--arcade-magenta);
}

/* Mobile */
@media (max-width: 480px) {
  .arcade-gameover-label { font-size: 13px; }
  .arcade-gameover-btn   { font-size: 8px; padding: 10px 18px; }
}
```

- [ ] **Step 3: Verificar no browser**

Terminar a leitura de um artigo → overlay GAME OVER aparece com "READ OVER" piscando, contador, botões neon. Botão NEXT reinicia artigo seguinte, GAME OVER volta à tela de input.

- [ ] **Step 4: Commit**

```bash
git add style.css
git commit -m "feat: arcade GAME OVER overlay CSS with blink animation"
```

---

## Task 8: app.js — despachar eventos CustomEvent

**Files:**
- Modify: `app.js`

- [ ] **Step 1: Despachar `eat:word` no `scrollLoop()` quando um grapheme é consumido**

Em `scrollLoop()`, no bloco `if (newFloor > prevFloor)` (linha ~1016), adicionar após o for loop existente:

```js
if (newFloor > prevFloor) {
  state.lastBiteTime = performance.now();
  const g = chompGeometry();
  for (let ci = prevFloor; ci < newFloor && ci < graphemeCount; ci++) {
    const ch = state.graphemes[ci]?.char ?? '';
    if (isSpecialChar(ch)) spawnExplosion(g.mouthX, g.activeY, g.lh);
    else spawnBiteCrunch(g.mouthX, g.activeY, g.lh, ch);
  }
  // Notificar arcade.js — fora do rAF, via queueMicrotask
  const wordsConsumed = newFloor - prevFloor;
  queueMicrotask(() => {
    dispatchEvent(new CustomEvent('eat:word', {
      detail: { count: wordsConsumed, speed: state.settings.speed }
    }));
  });
}
```

- [ ] **Step 2: Despachar `eat:speed-change` em todos os lugares onde `state.settings.speed` muda**

Criar helper logo após as constantes no topo:

```js
function notifySpeedChange() {
  queueMicrotask(() => {
    dispatchEvent(new CustomEvent('eat:speed-change', {
      detail: { level: state.settings.speed }
    }));
  });
}
```

Chamar `notifySpeedChange()` nos dois pontos onde speed muda:
1. No handler do slider de settings (linha ~269–270)
2. No handler de swipe vertical (linha ~1251–1254)

- [ ] **Step 3: Expor `data-article-count` no `#screen-prompter`**

Em `showReadOver()` (já modificado no Task 2), adicionar antes do `hidden = false`:

```js
screens.prompter.dataset.articleCount = state.articles.length;
screens.prompter.dataset.articleIndex = state.articleIndex;
```

- [ ] **Step 4: Despachar `eat:speed-change` no `startPrompter()` para sincronizar estado inicial**

Em `startPrompter()` (linha ~1279), após setar o estado inicial, adicionar:

```js
notifySpeedChange();
```

- [ ] **Step 5: Verificar eventos no browser**

Abrir DevTools → Console → colar:
```js
window.addEventListener('eat:word', e => console.log('word', e.detail));
window.addEventListener('eat:speed-change', e => console.log('speed', e.detail));
window.addEventListener('eat:article-end', e => console.log('end', e.detail));
```
Iniciar leitura e confirmar que os eventos aparecem no console.

- [ ] **Step 6: Commit**

```bash
git add app.js
git commit -m "feat: dispatch eat:word, eat:speed-change, eat:article-end CustomEvents from app.js"
```

---

## Task 9: Criar arcade.js — score, vidas e DOM updates

**Files:**
- Create: `arcade.js`

- [ ] **Step 1: Criar o arquivo com estrutura base e score**

```js
// arcade.js — Arcade chrome: score, lives, HUD pips, GAME OVER overlay
// Comunicação com app.js apenas via CustomEvents — nunca toca no canvas.

const HISCORE_KEY = 'eattext:hiscore';

const state = {
  score:   0,
  words:   0,
  hiscore: Number(localStorage.getItem(HISCORE_KEY) ?? 0),
};

// ── DOM refs ──────────────────────────────────────────────────

const $ = (id) => document.getElementById(id);

const ui = {
  score1up:  $('arcade-score-1up'),
  scoreHi:   $('arcade-score-hi'),
  scoreWords:$('arcade-score-words'),
  lives:     $('arcade-lives'),
  gameover:  $('arcade-gameover'),
  goCount:   $('arcade-gameover-count'),
  pips:      $('ahud-pips'),
  wpmVal:    $('ahud-wpm-val'),
};

// ── Helpers ───────────────────────────────────────────────────

function pad5(n) {
  return String(Math.min(99999, n)).padStart(5, '0');
}

function updateScoreDOM() {
  if (ui.score1up)   ui.score1up.textContent   = pad5(state.score);
  if (ui.scoreHi)    ui.scoreHi.textContent    = pad5(state.hiscore);
  if (ui.scoreWords) ui.scoreWords.textContent = pad5(state.words);
}

function saveHiscore() {
  if (state.score > state.hiscore) {
    state.hiscore = state.score;
    localStorage.setItem(HISCORE_KEY, state.hiscore);
  }
}

// ── Event: eat:word ───────────────────────────────────────────

window.addEventListener('eat:word', (e) => {
  const { count = 1, speed = 1 } = e.detail ?? {};
  // Aproximação: ~5 chars por palavra
  const wordsApprox = Math.max(1, Math.round(count / 5));
  state.words += wordsApprox;
  state.score += wordsApprox * 10 * speed;
  saveHiscore();
  requestAnimationFrame(updateScoreDOM);
});

// ── Event: eat:speed-change ───────────────────────────────────

window.addEventListener('eat:speed-change', (e) => {
  const level = e.detail?.level ?? 1;
  updatePips(level);
  updateWPM(level);
});

function updatePips(level) {
  if (!ui.pips) return;
  ui.pips.querySelectorAll('.ahud-pip').forEach(pip => {
    const n = Number(pip.dataset.pip);
    const on = n <= level;
    pip.classList.toggle('on',     on);
    pip.classList.toggle('yellow', on && n <= 3);
    pip.classList.toggle('orange', on && n >= 4 && n <= 7);
    pip.classList.toggle('red',    on && n >= 8);
  });
}

function calcWPM(speed) {
  return Math.round(150 + (speed - 1) / 9 * 450);
}

function updateWPM(level) {
  if (ui.wpmVal) ui.wpmVal.textContent = calcWPM(level);
}

// ── Vidas (artigos restantes) ─────────────────────────────────

function updateLives(articleIndex, total) {
  if (!ui.lives) return;
  const remaining = total - articleIndex;
  ui.lives.querySelectorAll('.arcade-life').forEach((el, i) => {
    el.classList.toggle('spent', i >= remaining);
  });
}

// ── Event: eat:article-end ────────────────────────────────────

window.addEventListener('eat:article-end', (e) => {
  const { index = 0, total = 1 } = e.detail ?? {};
  updateLives(index, total);
});

// ── Inicialização ─────────────────────────────────────────────

updateScoreDOM();
```

- [ ] **Step 2: Verificar no browser**

Iniciar leitura → score e words devem incrementar conforme texto é lido. Mudar velocidade → pips acendem/apagam. Terminar artigo → ícones de vida atualizam.

- [ ] **Step 3: Commit**

```bash
git add arcade.js
git commit -m "feat: arcade.js — score, words, lives, pips via CustomEvents"
```

---

## Task 10: arcade.js — reset de score ao iniciar nova sessão

**Files:**
- Modify: `arcade.js`
- Modify: `app.js`

- [ ] **Step 1: Despachar `eat:session-start` em `startPrompter()` no app.js**

Em `startPrompter()` (linha ~1279), no início da função, adicionar:

```js
queueMicrotask(() => {
  dispatchEvent(new CustomEvent('eat:session-start'));
});
```

- [ ] **Step 2: Escutar `eat:session-start` em arcade.js para resetar score/words**

Adicionar em `arcade.js`:

```js
// ── Event: eat:session-start ──────────────────────────────────

window.addEventListener('eat:session-start', () => {
  state.score = 0;
  state.words = 0;
  requestAnimationFrame(updateScoreDOM);
});
```

- [ ] **Step 3: Verificar reset**

Terminar um artigo → GAME OVER → GAME OVER button → voltar à tela de input → iniciar novo artigo → score deve começar em 00000.

- [ ] **Step 4: Commit**

```bash
git add app.js arcade.js
git commit -m "feat: reset arcade score on session start"
```

---

## Verificação final

- [ ] Abrir app no mobile (iOS Safari ou Chrome Android) — verificar que touch events ainda funcionam (swipe velocidade, pinch tamanho)
- [ ] Verificar que o canvas Pac-Man, ghost, ASCII mode continuam funcionando
- [ ] Verificar que scanlines não bloqueiam eventos de touch no canvas (pointer-events: none)
- [ ] Verificar HI-SCORE persiste após recarregar a página
- [ ] Verificar comportamento offline (Service Worker) — arcade.js deve carregar do cache
- [ ] Adicionar `arcade.js` ao `sw.js` se ele tiver lista de arquivos pré-cacheados

```bash
grep -n "app.js\|CACHE\|precache" /Users/lucianfialho/Code/eattext/sw.js
```

Se `sw.js` tiver lista explícita, adicionar `'/arcade.js'` a ela.

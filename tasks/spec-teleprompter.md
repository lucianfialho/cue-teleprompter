# Spec: Cue — Mobile Teleprompter PWA

**Status:** Final  
**Date:** 2026-04-03  
**Stack:** Vanilla JS + `@chenglou/pretext` + Canvas + PWA  

---

## 1. Problem Statement

**What are we solving?**  
Content creators, podcasters, and speakers need a teleprompter on mobile that works offline, is instantly accessible (no login, no install friction), and renders text smoothly at any scroll speed — without the jank typical of DOM-heavy scroll implementations.

**Why now?**  
The `pretext` + Canvas ecosystem is proving that smooth, reflow-free text rendering on mobile is achievable with minimal code. A teleprompter is the killer use case: long text, continuous scroll, full-screen, performance-critical.

---

## 2. User Stories

### US-1 — Entrada de roteiro
**As a** creator,  
**I want to** paste or type my script into a text area,  
**so that** I can prepare my teleprompter before presenting.

**Acceptance Criteria:**
- [ ] Textarea visível na tela inicial com placeholder explicativo
- [ ] Suporte a texto longo (10.000+ chars) sem degradação de performance
- [ ] Texto salvo automaticamente no `localStorage` ao digitar (debounce 500ms)
- [ ] Na próxima abertura, o último texto é restaurado automaticamente
- [ ] Botão "Limpar" com confirmação destrói o texto salvo

---

### US-2 — Iniciar teleprompter
**As a** creator,  
**I want to** start the teleprompter in fullscreen,  
**so that** I can read the script during recording without distractions.

**Acceptance Criteria:**
- [ ] Botão "Iniciar" entra em fullscreen (`requestFullscreen`) no mobile
- [ ] Texto renderizado via Canvas usando `@chenglou/pretext` (zero DOM reflow durante scroll)
- [ ] Auto-scroll contínuo configurável por velocidade
- [ ] Tap na tela pausa/retoma o scroll
- [ ] Swipe up/down ajusta velocidade em tempo real
- [ ] Botão de saída visível mas não intrusivo (canto superior)

---

### US-3 — Efeito de scroll
**As a** creator,  
**I want to** choose between linear scroll and fisheye (scroll morph),  
**so that** I can pick the style that helps me read better.

**Acceptance Criteria:**
- [ ] Toggle nos settings: "Linear" ou "Fisheye"
- [ ] **Linear:** scroll suave, fonte uniforme, alta legibilidade
- [ ] **Fisheye:** texto no centro da tela maior/mais brilhante, bordas menores/escuras (inspirado no `pinch-type` scroll morph)
- [ ] Mudança de efeito aplicada sem reiniciar o teleprompter
- [ ] Preferência salva no `localStorage`

---

### US-4 — Configurações de leitura
**As a** creator,  
**I want to** adjust font size, speed, and contrast,  
**so that** the teleprompter fits my reading distance and lighting conditions.

**Acceptance Criteria:**
- [ ] Slider de velocidade (1–10, default 4)
- [ ] Slider de tamanho de fonte (24px–96px, default 48px)
- [ ] Toggle de tema: Claro / Escuro / Auto (segue sistema)
- [ ] Toggle de modo espelho (flip horizontal — para teleprompter físico na frente da câmera)
- [ ] Toggle de indicador de progresso (barra no topo, default off)
- [ ] Todas as preferências persistidas no `localStorage`

---

### US-5 — PWA instalável
**As a** creator,  
**I want to** install the app na home screen do meu celular,  
**so that** abro direto sem precisar do browser.

**Acceptance Criteria:**
- [ ] `manifest.json` com nome, ícone, `display: standalone`, `orientation: portrait`
- [ ] Service Worker com cache offline (cache-first para assets estáticos)
- [ ] Funciona 100% offline após primeira visita
- [ ] Ícone 192x192 e 512x512
- [ ] `theme_color` e `background_color` alinhados ao design

---

### US-6 — Acessibilidade mobile
**As a** creator com necessidades visuais ou motoras,  
**I want to** usar o teleprompter com ajustes de acessibilidade,  
**so that** a ferramenta funciona para mim também.

**Acceptance Criteria:**
- [ ] Fonte mínima de 24px (nunca abaixo disso, independente do slider)
- [ ] Contraste mínimo WCAG AA (4.5:1) em ambos os temas
- [ ] Área de toque mínima de 44x44px em todos os controles (WCAG 2.5.5)
- [ ] Todos os controles interativos têm `aria-label`
- [ ] Suporte a `prefers-reduced-motion`: desativa animações de efeito fisheye se ativo
- [ ] Navegação por teclado bluetooth funcional (espaço = pause, setas = velocidade)

---

## 3. Non-Goals (Scope Defense)

- **Sem backend** — zero servidor, zero conta, zero auth
- **Sem múltiplos roteiros salvos** — apenas o último texto (US-2B escolhido)
- **Sem Picture-in-Picture** — mobile first, sem PiP (Desktop PiP é v2)
- **Sem colaboração em tempo real** — app local e offline
- **Sem formatação rica** — texto puro apenas (bold/italic são v2)
- **Sem compartilhamento via URL** — v2
- **Sem suporte a vídeo ou mídia** — texto only
- **Sem analytics ou telemetria**

---

## 4. Arquitetura Técnica (Decisões Antecipadas)

| Decisão | Escolha | Motivo |
|---|---|---|
| Text layout | `@chenglou/pretext` | DOM-free, zero reflow, canvas-native |
| Render | Canvas 2D API | Smooth scroll, fisheye effect, pretext-native |
| Estado | `localStorage` puro | Sem dependências, zero backend |
| Build | Zero build step | Single HTML + JS, deployável direto |
| Styling | CSS custom properties | Theming simples, sem framework |
| PWA | SW + Manifest nativo | Sem Workbox — manter zero-dep |

---

## 5. Open Questions — RESOLVIDAS

- [x] **Fonte:** `system-ui` — zero download, nativo
- [x] **Fisheye intensity:** Dramático — centro 2x maior, bordas quase invisíveis
- [x] **Restart de posição:** Retoma da mesma posição ao sair do fullscreen
- [x] **Indicador de progresso:** Opcional — toggle nas configurações (default off)
- [x] **Nome final do app:** **Cue**

---

## 6. Issues para GitHub

As seguintes tasks devem ser abertas como issues no repositório:

1. `[SETUP]` Estrutura inicial do projeto (HTML, manifest, SW, pretext)
2. `[US-1]` Tela de entrada de roteiro com auto-save localStorage
3. `[US-2]` Engine de scroll Canvas com pretext
4. `[US-3]` Implementar efeito fisheye (scroll morph) no Canvas
5. `[US-4]` Painel de configurações (velocidade, fonte, tema, espelho)
6. `[US-5]` PWA: manifest.json + Service Worker offline
7. `[US-6]` Acessibilidade: WCAG AA, aria-labels, reduced-motion, touch targets
8. `[QA]` Testar em iOS Safari, Android Chrome, e Firefox Android
9. `[DEPLOY]` Deploy no Vercel com domínio personalizado

---
name: verify
description: Cómo verificar cambios de este Tetris en un navegador real (headless).
---

# Verificar el juego

No hay tests ni build. La superficie es el navegador.

## Receta que funciona

1. Servir estático: `python3 -m http.server 8734` (en background) desde la raíz del repo.
2. Instalar `puppeteer-core` en el scratchpad (`npm install puppeteer-core`) y lanzar el Chrome del sistema:
   `executablePath: '/Applications/Google Chrome.app/Contents/MacOS/Google Chrome'`, `headless: 'new'`.
3. Conducir el juego con eventos reales de teclado (`page.keyboard.press`) — los listeners están en `document`:
   - `KeyP` / `Escape` → pausa; `Space` → hard drop; flechas → mover/rotar/bajar.
   - Game over rápido: ~9 hard drops seguidos con `Space` (las piezas se apilan en el centro).
4. Estado observable en el DOM: `#overlay`, `#gameover-box`, `#pause-box`, `#overlay-highscores`,
   `#highscore-entry` (clase `hidden`), HUD en `#score/#lines/#level`, skin como clase `skin-*` en `<body>`.
5. Persistencia en localStorage: `tetris-theme`, `tetris-skin`, `tetris-highscores`, `tetris-stats`, `tetris-start-level`.

## Gotchas

- `favicon.ico` da un 404 en consola; es inofensivo, ignorarlo al filtrar errores.
- El loop de animación se detiene en pausa/game over; los cambios de skin fuerzan un redraw manual.

# AGENTS.md — open-pacman

Clon de Pac-Man en JS vainilla + HTML + CSS. Sin build, sin bundler, sin dependencias, sin tests, sin lint.

## Ejecución

Abre `src/index.html` directamente en el navegador, o sírvelo de forma estática:

```powershell
npx serve src
python -m http.server --directory src 8000
```

Sin `package.json`, sin CI, sin pre-commit.

## Estructura (`src/`)

- `index.html` — canvas `#game` (560x620), overlay `#overlay`, carga los scripts en orden.
- `js/maze.js` — nivel de 28x31 como strings → `MAZE` numérico. Códigos de celda: `1` muro, `2` punto, `3` puerta del corral, `0` vacío. Expone globales: `MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`.
- `js/game.js` — estado + reglas (`createGame`, `update`, `DIRS`). Copia `MAZE` en `game.grid` en cada partida; nunca mutar `MAZE` directamente.
- `js/render.js` — dibujo en canvas (`draw`). Lee `game.grid`, no `MAZE`.
- `js/main.js` — bucle de juego, teclado (solo flechas), overlay de inicio/reinicio.
- `css/style.css` — solo maquetación.

## Convenciones que difieren de lo habitual

- Sin módulos ES: etiquetas `<script>` clásicas que comparten globales vía `window.*`. El orden de carga en `index.html` es crítico: `maze.js → game.js → render.js → main.js`.
- Modelo de actores: las posiciones son coordenadas fraccionarias de celda; solo se fijan/giran cuando `aligned()` (~1e-3). Velocidades en celdas/frame (`PACMAN_SPEED 0.125`, `GHOST_SPEED 0.1`).
- Reglas de colisión: pacman choca con muro (1) y puerta (3); los fantasmas solo con muro (1). El wrap del túnel solo aplica en `TUNNEL_ROW` (14). El fantasma `hunter` persigue por distancia Manhattan, `random` elige movimientos sin invertir la dirección (giro de 180° solo en callejones sin salida).
- Los overlays se reconstruyen vía `innerHTML` en `showOverlay` — el nodo `#action-btn` se reemplaza, así que hay que volver a consultarlo tras cada reconstrucción (el código actual ya lo hace).
- El tamaño del canvas es fijo (28 cols × 20px, 31 filas × 20px = 560×620). Mantener `TILE = 20` y las dimensiones CSS de `#game-wrap` sincronizados.

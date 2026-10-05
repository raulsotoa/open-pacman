# SPEC 03 — Power pellets para comer fantasmas

> **Status:** Aprobado
> **Depends on:** SPEC 01, SPEC 02
> **Date:** 2026-10-05
> **Objective:** Añadir 4 power pellets que activan 10s de modo asustado donde Pac-Man come fantasmas por combo.

## Scope

**In:**

- 4 pellets en esquinas clásicas: `(1,3)`, `(26,3)`, `(1,23)`, `(26,23)`; nuevo código de celda `4` (`'o'` → `4` en `src/js/maze.js`).
- Modo asustado de 10s (600 frames a 60fps); comer otro pellet reinicia timer y combo.
- Fantasmas asustados: invierten dirección al activarse, movimiento aleatorio (regla `random` actual), velocidad lenta `GHOST_FRIGHT_SPEED 0.06`, no matan al contacto.
- Comer fantasma: combo `200/400/800/1600` según orden dentro de la ventana; el pellet mismo da 0 pts.
- Fantasma comido: respawn inmediato en su `GHOST_STARTS[i]` en estado normal (no comestible) aunque el modo siga activo.
- Los 4 pellets cuentan en `dotsRemaining` para ganar.
- Visual: pellet grande (radio ~6px, parpadeo suave) y fantasma azul sólido con ojos blancos.

**Out of scope (para futuros specs):**

- Ojos viajando al corral y reentrada animada.
- Parpadeo blanco de aviso al final del modo.
- Scatter/chase global, niveles, vidas extra, sonidos.
- Puntos extra por el pellet (50 del arcade).

## Data model

```js
// maze.js — MAZE_STR usa 'o' en las 4 esquinas; parseTile: 'o' → 4
// game.js — estado nuevo
game.frightTimer = 0;      // segundos restantes, decrementa 1/60 por update
game.ghostsEaten = 0;      // 0..3, índice del combo en la ventana actual
// fantasma en createGame(): g.frightened = false
const GHOST_FRIGHT_SPEED = 0.06;
const FRIGHT_DURATION = 10; // segundos
const FRIGHT_POINTS = [200, 400, 800, 1600];
```

Convenciones: se reutiliza `aligned()`, `canMove()` y `wrapTunnel()`; `isWall` sin cambios.

## Implementation plan

1. `src/js/maze.js`: añadir `'o'` → `4` y colocar las 4 esquinas sobre dots existentes. Verificación: `MAZE` tiene cuatro `4` y el juego carga igual.
2. `src/js/game.js` (`createGame`): contar `4` en `dotsRemaining`, inicializar `frightTimer`/`ghostsEaten`, añadir `g.frightened`. Verificación: sin errores, ganar exige comer pellets.
3. `src/js/game.js` (`movePacman`): comer `4` → `grid=0`, `dotsRemaining--`, `frightTimer=10`, `ghostsEaten=0`, marcar `frightened=true` en vivos e invertir `dir` a `OPPOSITE`. Verificación: timer se reinicia al comer segundo pellet.
4. `src/js/game.js` (`decideGhost`/`moveGhost`): si `g.frightened`, usar lógica `random` + velocidad `0.06`. Verificación: en modo activo los 4 deambulan lento.
5. `src/js/game.js` (`update`/colisión): si colisión y `g.frightened` → puntos por combo, `ghostsEaten++`, respawn en `GHOST_STARTS[i]` con `frightened=false`; si no, vida menos como hoy; expirar `frightTimer` limpia flags. Verificación: combo encadena 200→1600, revivido no recomestible.
6. `src/js/render.js`: `drawDots` dibuja `4` grande + `drawGhost` azul si `frightened`. Verificación: visual en `src/index.html` sin errores de consola.

## Acceptance criteria

- [ ] Hay 4 pellets grandes en las 4 esquinas y desaparecen al comerlos.
- [ ] Comer un pellet activa 10s (±60 frames) de fantasmas azules que no matan.
- [ ] Al activarse, los fantasmas invierten dirección y se mueven aleatorio lento.
- [ ] Comer fantasmas da 200/400/800/1600 en orden dentro de la misma ventana.
- [ ] Segundo pellet con modo activo reinicia timer a 10s y combo a 200.
- [ ] Fantasma comido reaparece en su spawn normal (azul no, no recomestible).
- [ ] Sin pellet comido, la colisión quita vida como antes.
- [ ] Ganar exige comer puntos + los 4 pellets.
- [ ] Sin errores en consola al abrir `src/index.html`.

## Decisions

- **Sí:** 4 esquinas clásicas. Simétrico, cabe sin tocar muros ni corral.
- **Sí:** 10s. Ventana generosa pedida por el usuario para encadenar.
- **Sí:** combo 200/400/800/1600, pellet 0 pts. Decisión del usuario.
- **Sí:** respawn en spawn, no ojos. Simple, sin IA nueva (decisión del usuario).
- **Sí:** revivido normal no comestible. Evita farmear el mismo fantasma en una ventana.
- **No:** ojos viajando, parpadeo blanco final, 50 pts por pellet. Van a otro spec si se quieren.

## Risks

| Riesgo | Mitigación |
|---|---|
| Timer ligado a frames y rAF a 120/144Hz acorta/alarga los 10s | Decrementar por `1/60` por `update` como `exitTimer`; documentar la unidad |
| Fantasma revivido sobre Pac-Man lo mata al instante | Respawn usa `GHOST_STARTS[i]` lejanos; la colisión del mismo frame ya se resolvió con `break` |

## What is **not** in this spec

- Ojos, parpadeo blanco, scatter global, niveles, sonidos.

Cada uno de esos, si llega, va en su propio spec.

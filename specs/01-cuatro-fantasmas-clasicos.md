# SPEC 01 — Cuatro fantasmas estilo clásico con persecución agresiva

> **Status:** Aprobado
> **Depends on:** —
> **Date:** 2026-10-05
> **Objective:** Pasar de 2 a 4 fantasmas con personalidades distintas estilo arcade, uno de ellos (rojo) persiguiendo agresivamente a Pac-Man.

## Scope

**In:**

- 4 fantasmas: Blinky rojo agresivo (persecución directa), Pinky rosa emboscador (4 casillas por delante), Inky cian flanqueador (fórmula con pivote + Blinky), Clyde naranja tímido (persigue hasta <8 Manhattan, luego esquina inferior-izquierda).
- Misma velocidad para los 4 (`GHOST_SPEED` 0.1 actual).
- Spawns: Blinky fuera del corral (13,11), Pinky (13,14), Inky (12,14), Clyde (15,14); salida por turnos con temporizador (0s / 0s / ~3s / ~6s) con balanceo dentro del corral hasta salir.
- Sin modo scatter global: cada fantasma siempre en su personalidad.
- Colores por orden existente en `render.js` (`GHOST_COLORS` ya tiene 4 entradas).

**Out of scope (para futuros specs):**

- Modo asustado/azul, comer fantasmas y ojos.
- Temporizador global scatter/chase del arcade.
- Cambios de puntuación, sonidos, niveles o vidas.

## Data model

```js
// maze.js — ampliado a 4 entradas
const GHOST_STARTS = [
  { x: 13, y: 11, kind: 'blinky', exitDelay: 0, corner: { x: 26, y: 1 } },
  { x: 13, y: 14, kind: 'pinky', exitDelay: 0, corner: { x: 1, y: 1 } },
  { x: 12, y: 14, kind: 'inky', exitDelay: 3, corner: { x: 26, y: 29 } },
  { x: 15, y: 14, kind: 'clyde', exitDelay: 6, corner: { x: 1, y: 29 } },
];

// game.js — fantasma en createGame()
g = { x, y, dir, speed, kind, exitDelay, exitTimer, corner }
// Targets (celdas enteras vía Math.round):
// blinky: pacman. pinky: pacman + 4*dir. inky: pivot + (pivot - blinky), pivot = pacman + 2*dir.
// clyde: si dist Manhattan < 8 → corner, si no → pacman.
```

Convenciones: coordenadas con origen arriba-izquierda, posiciones fraccionarias, decisiones de giro solo con `aligned()`; fórmulas simplificadas sin el bug clásico del "up".

## Implementation plan

1. Ampliar `GHOST_STARTS` en `src/js/maze.js` a las 4 entradas con `kind`, `exitDelay` y `corner`. Verificación: el juego carga igual (los 2 primeros comportamientos aún no cambian de lógica).
2. Extender el objeto fantasma en `createGame` (`src/js/game.js`) con `exitTimer` y `corner`, sin cambiar su firma. Verificación: `src/index.html` abre sin errores de consola.
3. Reescribir `decideGhost` con target por `kind` manteniendo la regla actual (no invertir salvo callejón). Verificación: cada fantasma gira distinto ante la misma posición de Pac-Man.
4. Añadir salida del corral por temporizador (balanceo vertical dentro + forzar `up` hacia la puerta) y restaurar timers en `resetPositions`. Verificación: al iniciar, Blinky/Pinky fuera y los otros dos salen a ~3s/~6s.
5. Ajustar `resetPositions` para recolocar a los 4 en sus spawns con sus timers. Verificación: tras perder una vida, los 4 vuelven a su sitio y repiten la secuencia de salida.

## Acceptance criteria

- [ ] Hay 4 fantasmas visibles con los 4 colores existentes.
- [ ] El rojo sigue la casilla exacta de Pac-Man por Manhattan sin tregua.
- [ ] El rosa se dirige 4 casillas por delante de la dirección de Pac-Man.
- [ ] El cian usa la posición de Blinky en su cálculo (si Blinky cambia, su target cambia).
- [ ] El naranja persigue a ≥8 Manhattan y huye a su esquina a <8.
- [ ] Los 4 van a la misma velocidad base.
- [ ] Secuencia de salida: Blinky y Pinky inmediatos, Inky ~3s, Clyde ~6s.
- [ ] Tras colisión con vida restante, los 4 reaparecen en sus spawns y repiten la salida.
- [ ] Sin errores en consola al abrir `src/index.html`.

## Decisions

- **Sí:** sin scatter global, personalidades siempre activas. Más simple, sin temporizador global que sincronizar con muertes/reinicios.
- **No:** scatter/chase alternado del arcade. Va a otro spec si se quiere fidelidad total.
- **Sí:** salida por tiempo (0/0/3/6s). Simple y testeable manual con cronómetro.
- **No:** salida por contador de puntos. Añade estado acoplado a `dotsRemaining` y es más difícil de verificar.
- **Sí:** fórmulas clásicas simplificadas sin bug del "up". Menos código, comportamiento indistinguible para el jugador.
- **Sí:** reutilizar `GHOST_COLORS` y el sistema de globales sin módulos. Respeta la arquitectura actual (`maze.js → game.js → render.js → main.js`).

## Risks

| Riesgo | Mitigación |
|---|---|
| Inky depende de la posición de Blinky; si Blinky está en el túnel con wrap, el target salta | Usar `Math.round` y el valor ya con wrap aplicado; el salto es de 1 frame y se auto-corrige |
| Fantasmas atascados botando dentro del corral si el timer falla | Forzar dirección `up` al vencer el timer; la puerta (3) no bloquea a fantasmas por `isWall` |

## What is **not** in this spec

- Modo asustado, comer fantasmas, ojos volviendo al corral.
- Scatter/chase global, niveles, puntuación extra, sonidos.

Cada uno de esos, si llega, va en su propio spec.

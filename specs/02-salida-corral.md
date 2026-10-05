# SPEC 02 — Salida garantizada de los fantasmas del corral al mapa

> **Status:** Aprobado
> **Depends on:** SPEC 01
> **Date:** 2026-10-05
> **Objective:** Garantizar que los 4 fantasmas salgan del corral al mapa al inicio y tras cada muerte, sin quedarse rebotando contra los muros.

## Scope

**In:**

- Reproducir el atrapamiento (varios fantasmas, al inicio, rebotando pegados a un muro lateral) con harness Node + comprobación visual en navegador.
- Corregir la rutina de salida (`inPen`/`moveGhost` en `src/js/game.js`) para que todo fantasma con timer vencido alcance una celda fuera del corral.
- Mantener el diseño de SPEC 01: personalidades, velocidad 0.1, tiempos 0/0/~3/~6s, balanceo retenido, spawns, inicio y `resetPositions`.

**Out of scope (para futuros specs):**

- Cambiar personalidades, velocidades o tiempos de salida.
- Spawns fuera del corral o teleport a la puerta (descartados por el usuario).
- Modo asustado, comer fantasmas, scatter global.

## Data model

Sin estructuras nuevas. Reutiliza el fantasma de SPEC 01 (`exitDelay`, `exitTimer`, `corner`).

## Implementation plan

1. Reproducir: harness Node que corre 700 frames y reporta qué fantasmas siguen `inPen` tras su deadline +60 frames de margen, más comprobación visual en `src/index.html`. Verificación: el harness falla (al menos un fantasma atrapado) antes del fix.
2. Corregir el steering de salida en `moveGhost`: con timer vencido dentro del corral, ignorar `decideGhost` y navegar determinista (centrar x a columnas 13–14 por la fila actual y subir por la puerta hasta salir). Verificación: el harness pasa.
3. Verificar regresiones: timings 0/0/~3/~6s, targets por personalidad (harness de SPEC 01), 600 updates sin excepción, consola limpia. Verificación: todo en verde + check visual en navegador.

## Acceptance criteria

- [ ] Los 4 fantasmas están fuera del corral antes del frame 500 (~8s) tras el inicio.
- [ ] Tras colisión con vida restante, los 4 vuelven a salir antes de 500 frames.
- [ ] Ningún fantasma permanece >60 frames consecutivos dentro del corral con timer vencido.
- [ ] Se conservan los tiempos 0/0/~3/~6s (±60 frames de margen).
- [ ] Sin errores en consola al abrir `src/index.html`.

## Decisions

- **Sí:** arreglar la navegación por la puerta manteniendo timers (decisión del usuario).
- **No:** spawns fuera del corral ni teleport por turnos (descartados por el usuario).
- **Sí:** paso de reproducción primero. La causa exacta no está caracterizada: el harness Node de SPEC 01 pasaba pero en navegador se observa atrapamiento.
- **Sí:** aplica a inicio y a `resetPositions` (decisión del usuario).

## Risks

| Riesgo | Mitigación |
|---|---|
| La causa es diferencia navegador vs Node (rAF a 120/144Hz acelera timers, o JS viejo en caché) y no lógica | Probar con caché desactivada; si es caché, documentarlo y el fix sigue valiendo |
| Regresión en persecución al tocar `moveGhost` | Reejecutar el harness de targets de SPEC 01 |

## What is **not** in this spec

- Cambios de IA, velocidades, tiempos o spawns.
- Modo asustado, comer fantasmas, scatter global, niveles, sonidos.

Cada uno de esos, si llega, va en su propio spec.

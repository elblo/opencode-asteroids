# AGENTS.md

Clon de Asteroids en canvas HTML5 puro. Cuatro archivos: `index.html`, `game.js`, `favicon.svg`, `README.md`.

**No hay `package.json`, bundler, dependencias, tests, linter ni CI.** No inventes comandos que no existen.

## Ejecutar y verificar

- `python3 -m http.server 8137` y abrir `http://localhost:8137`. Funciona con `file://` también: no hay módulos, `fetch` ni assets externos.
- El `npx serve .` que menciona el README requiere red (no hay lockfile). Cualquier servidor estático vale.
- Único chequeo automatizable: `node --check game.js` (sintaxis). No existe `npm test`.
- Verifica cambios abriendo el juego y jugando: no hay harness headless en el repo, así que no afirmes que algo "funciona" solo porque el sintaxis pasa.

## Arquitectura (`game.js`, un solo archivo)

- `index.html` lo carga con `<script src="game.js">` plano. **Añadir `import`/`export` lo rompe**: sin módulos y sin bundler.
- Entrada al final del archivo: `initGame()` + `requestAnimationFrame(loop)`. El loop es `loop → update(dt) → draw()`; `dt` se clampa a 0.05 s (`game.js:415`) y no hay sub-stepping.
- Estado global mutable en `game.js:239-242`: `ship, bullets, asteroids, particles, score, lives, level, state, deadTimer`. No hay encapsularlo en una clase de juego.
- Patrón de entidad: clases planas con `update(dt)` / `draw()` y un flag `dead`; la muerte se marca y se purga con `filter()` al final de `update()`, nunca con `splice()` dentro de un `for`. Síguelo para cualquier entidad nueva.
- `state` es `'playing' | 'dead' | 'gameover'`, con ramas de early-return al principio de `update()`. Un estado nuevo necesita su rama ahí.
- Sin manejo de resize: las constantes `W`/`H` (`game.js:5-6`) duplican los atributos `width`/`height` del canvas (`index.html:23`). Cámbialas en ambos sitios o el render se desalinea.

## Quirks que importan

- **Mundo toroidal, colisiones no.** `wrap()` hace el envolvimiento de bordes, pero `dist()` es euclidiana plana: dos objetos en bordes opuestos nunca colisionan. Es una limitación conocida, no la trates como bug de tu cambio.
- `spawnAsteroids()` (`game.js:251`) calcula `SAFE_DIST` contra el centro del canvas hardcodeado, no contra la nave.
- `pressed(code)` **consume** el flag de flanco: llámalo como máximo una vez por frame y por tecla. Disparo y reinicio usan `pressed('Space')`.
- Los listeners leen `e.code` (tecla física, independiente de layout) y escriben en los globales `keys` / `justPressed`. No hay scoping por foco del canvas.

## Convenciones

- Comentarios y textos de HUD/overlay en español; identificadores en inglés. El texto visible al usuario va en español (`SCORE`, `NIVEL`, `PUNTAJE`).
- Constantes de tuning declaradas donde se usan, no arriba del archivo: `SPEED` dentro del constructor de `Bullet` (`game.js:37`), `ROT`/`THRUST`/`DRAG` dentro de `Ship.update` (`game.js:143-145`). Las únicas tablas globales son `RADII`/`SPEEDS`/`POINTS` indexadas por tamaño (1 pequeño, 2 mediano, 3 grande) en `game.js:61-63`.
- Si cambias features, actualiza `README.md`. Ojo: `README.md:7` ya afirma power-ups y un asteroide "estrella fugaz" que **no existen** en el código — verificado. No trates la lista de features del README como estado implementado.

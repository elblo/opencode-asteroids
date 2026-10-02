# AGENTS.md

Clon de Asteroids en canvas HTML5 puro. Cuatro archivos de juego: `index.html`, `game.js`, `favicon.svg`, `README.md` (más este `AGENTS.md`).

**No hay `package.json`, bundler, dependencias, tests, linter ni CI.** No inventes comandos que no existen.

## Ejecutar y verificar

- `python3 -m http.server 8137` y abrir `http://localhost:8137`. Funciona con `file://` también: no hay módulos, `fetch` ni assets externos.
- El `npx serve .` que menciona el README requiere red (no hay lockfile). Cualquier servidor estático vale.
- Único chequeo automatizable: `node --check game.js` (sintaxis). No existe `npm test`.
- Verifica cambios abriendo el juego y jugando: no hay harness headless en el repo, así que no afirmes que algo "funciona" solo porque el sintaxis pasa.

## Arquitectura (`game.js`, un solo archivo)

- `index.html` lo carga con `<script src="game.js">` plano. **Añadir `import`/`export` lo rompe**: sin módulos y sin bundler.
- Entrada al final del archivo: `initGame()` + `requestAnimationFrame(loop)`. El loop es `loop → update(dt) → draw()`; `dt` se clampa a 0.05 s (`game.js:687`) y no hay sub-stepping.
- Estado global mutable en `game.js:433-438`: `ship, bullets, asteroids, particles, powerups, score, lives, level, state, deadTimer, starTimer, starWarn`. No hay encapsularlo en una clase de juego.
- Patrón de entidad: clases planas con `update(dt)` / `draw()` y un flag `dead`; la muerte se marca y se purga con `filter()` al final de `update()`, nunca con `splice()` dentro de un `for`. Síguelo para cualquier entidad nueva.
- `state` es `'playing' | 'dead' | 'gameover'`, con ramas de early-return al principio de `update()`. Un estado nuevo necesita su rama ahí.
- Sin manejo de resize: las constantes `W`/`H` (`game.js:5-6`) duplican los atributos `width`/`height` del canvas (`index.html:23`). Cámbialas en ambos sitios o el render se desalinea.

## Quirks que importan

- **Mundo toroidal, colisiones mixtas.** `wrap()` envuelve las posiciones, pero `dist()` (línea 28) es euclidiana plana: balas y nave-asteroide no colisionan a través de los bordes. Es una limitación conocida y preexistente, no la trates como bug de tu cambio. El power-up sí usa `wrapDist()`, que sí es toroidal.
- `spawnAsteroids()` (`game.js:440-447`) calcula `SAFE_DIST` contra el centro del canvas hardcodeado, no contra la nave.
- `pressed(code)` **consume** el flag de flanco: llámalo como máximo una vez por frame y por tecla. Disparo y reinicio usan `pressed('Space')`.
- Los listeners leen `e.code` (tecla física, independiente de layout) y escriben en los globales `keys` / `justPressed`. No hay scoping por foco del canvas.

## Power-ups

- Tabla `POWERUPS` junto a `RADII`/`SPEEDS`/`POINTS`. `dropChance` va indexada por **tamaño de asteroide** (1, 2, 3), y tamaño 1 nunca suelta.
- El drop sale de `maybeDropPowerUp()`, que se llama dentro del bucle bala-asteroide. **Añadir un tipo nuevo es una entrada en `POWERUPS` más su `apply()` en `PowerUp`** — no hace falta tocar el spawn ni el update.
- `PowerUp` corre con el patrón del repo: flag `dead` + `filter()`. Tiene `ttl` propio (12 s), así que se autodestruye si nadie lo recoge.
- ⚠️ `nextLevel()` limpia `powerups`. El drop del último asteroide del nivel se pierde, y si lo recoges en ese mismo frame `ship.reset()` se come el boost. Ambos son intencionados; no son bugs.
- El efecto se implementa escalando `ROT` y `THRUST` ×2, **no** tocando `DRAG`: el drag se aplica por frame, no por segundo, así que escalar la aceleración da un ×2 exacto a cualquier refresh rate mientras que tocar el drag no.

## Asteroides especiales

- `ShootingStar` (`game.js:160`) es **una subclase de `Asteroid`**, no una bandera: hereda movimiento, wrap y toda la colisión (la que mata a la nave incluida). Sobrescribe velocidad, `draw()`, `split()` → `[]` y añade `ttl`.
- La puntuación **no está cableada al tamaño**: `Asteroid` tiene `this.points` (`game.js:105`) y el scoring usa `score += a.points` (`game.js:564`). Por eso un subtipo solo sobrescribe una propiedad. Mismo truco para `explode(a.x, a.y, a.size * 5)`, que sigue atado a `size`.
- `split()` instancia `new Asteroid(...)` hardcodeado, no `new this.constructor()` — por eso los subtipos no se propagan a los fragmentos. No lo "mejores" sin querer.
- ⚠️ **El fin de nivel filtra la estrella** (`game.js:606`): `normales.length === 0`. Sin ese filtro, esperar a que caduque bloquea el nivel hasta 8 s. Cualquier entidad temporal que meta en `asteroids` necesita la misma exclusión.
- El temporizador `starTimer` solo corre en la rama `state === 'playing'`: las ramas `dead`/`gameover` hacen early-return antes. `nextLevel()` lo reinicia a `grace` y limpia la estrella.
- `Particle` acepta `vx`/`vy` opcionales para la estela. Sin ellos se comporta como siempre (aleatorio), así que los ~5 call sites existentes no se rompen.

## Convenciones

- Comentarios y textos de HUD/overlay en español; identificadores en inglés. El texto visible al usuario va en español (`SCORE`, `NIVEL`, `PUNTAJE`).
- Constantes de tuning declaradas donde se usan, no arriba del archivo: `SPEED` dentro del constructor de `Bullet` (`game.js:42`), `ROT`/`THRUST`/`DRAG` dentro de `Ship.update` (`game.js:260-263`). Las únicas tablas globales van agrupadas arriba, indexadas por tamaño de asteroide (1 pequeño, 2 mediano, 3 grande): `RADII`/`SPEEDS`/`POINTS` (`game.js:66-68`), `POWERUPS` (`game.js:73`) y `SHOOTING_STAR` (`game.js:85`).
- Si cambias features, actualiza `README.md`. Ya se quitó de ahí la afirmación de que existiría una "estrella fugaz": se implementó después, así que ahora sí es cierta — pero no reescribas features que no existan.

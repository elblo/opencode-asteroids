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
- Entrada al final del archivo: `initGame()` + `requestAnimationFrame(loop)`. El loop es `loop → update(dt) → draw()`; `dt` se clampa a 0.05 s (`game.js:909`) y no hay sub-stepping.
- Estado global mutable en `game.js:618-624`: `ship, bullets, asteroids, particles, powerups, score, lives, level, state, deadTimer, starTimer, starWarn, skinIndex, skinToast`. No hay encapsularlo en una clase de juego.
- Patrón de entidad: clases planas con `update(dt)` / `draw()` y un flag `dead`; la muerte se marca y se purga con `filter()` al final de `update()`, nunca con `splice()` dentro de un `for`. Síguelo para cualquier entidad nueva.
- `state` es `'playing' | 'dead' | 'gameover'`, con ramas de early-return al principio de `update()`. Un estado nuevo necesita su rama ahí.
- Sin manejo de resize: las constantes `W`/`H` (`game.js:5-6`) duplican los atributos `width`/`height` del canvas (`index.html:23`). Cámbialas en ambos sitios o el render se desalinea.

## Quirks que importan

- **Mundo toroidal, colisiones mixtas.** `wrap()` envuelve las posiciones, pero `dist()` (línea 28) es euclidiana plana: balas y nave-asteroide no colisionan a través de los bordes. Es una limitación conocida y preexistente, no la trates como bug de tu cambio. El power-up sí usa `wrapDist()`, que sí es toroidal.
- `spawnAsteroids()` (`game.js:627-636`) calcula `SAFE_DIST` contra el centro del canvas hardcodeado, no contra la nave.
- `pressed(code)` **consume** el flag de flanco: llámalo como máximo una vez por frame y por tecla. Disparo y reinicio usan `pressed('Space')`.
- Los listeners leen `e.code` (tecla física, independiente de layout) y escriben en los globales `keys` / `justPressed`. No hay scoping por foco del canvas.

## Power-ups

- Tabla `POWERUPS` con tres tipos: `velocidad`, `triple` y `escudo`. `dropChance` va indexada por **tamaño de asteroide** (1, 2, 3), y tamaño 1 nunca suelta.
- El drop sale de `maybeDropPowerUp()`, que se llama dentro del bucle bala-asteroide. Elige un tipo al azar entre los que ese tamaño puede soltar y luego tira su `dropChance`; con las tres tablas idénticas, el total es 60 % en grande y 25 % en mediano, repartido a partes iguales entre los tipos.
- **Añadir un tipo nuevo es una entrada en `POWERUPS` más su rama en `apply()` y su glifo en `PowerUp.draw()`** — no hace falta tocar el spawn ni el update.
- `PowerUp` corre con el patrón del repo: flag `dead` + `filter()`. Tiene `ttl` propio (12 s), así que se autodestruye si nadie lo recoge.
- ⚠️ `nextLevel()` limpia `powerups`. El drop del último asteroide del nivel se pierde, y si lo recoges en ese mismo frame `ship.reset()` se come el efecto. Ambos son intencionados; no son bugs.
- Cada efecto vive en su temporizador de `Ship`: `boost` (5 s), `triple` (5 s) y `shield` (6 s). `killShip()` cancela los tres.
- «Velocidad» escala `ROT` y `THRUST` ×2, **no** toca `DRAG`: el drag se aplica por frame, no por segundo, así que escalar la aceleración da un ×2 exacto a cualquier refresh rate mientras que tocar el drag no.
- «Triple» se resuelve en `tryShoot()` devolviendo tres `Bullet` paralelas separadas 10 px en perpendicular al rumbo.
- «Escudo» se resuelve en el bucle nave-asteroide: amplía el radio de contacto a `shieldRadius` (26) y destruye el asteroide sin puntos ni drop. Las ramas `dead`/`gameover` no lo procesan.
- El HUD pinta una fila por efecto activo (`activos`, a partir de `y = 46`, 18 px por fila); añadir un efecto visible requiere añadirlo a esa lista.

## Skins

- Tabla `SKINS` junto al resto de tablas globales. Cada entrada es `id`, `name`, `color`, `flame`, `flameX`, `hull` y `details`; **añadir una skin es añadir una entrada**, no tocar `Ship.draw`.
- `traceSkin(skin, scale)` concentra el dibujo del casco y los detalles, y lo reutilizan `Ship.draw` y `drawLifeIcon`. Las siluetas no deben superar el tamaño de la nave original (extensión máxima ~21 px) para no falsear el hitbox (`radius = 12`) ni el punto de disparo (`NOSE = 21`).
- `skinIndex` vive en el estado global y se persiste en `localStorage` por `id` (`asteroids.skin`). `loadSkin`/`saveSkin` van envueltos en `try/catch` porque el almacenamiento puede no estar disponible.
- La tecla `S` consume `pressed('KeyS')` al principio de `update()`, antes de las ramas de estado: así se puede cambiar de skin también en `dead` y `gameover`.

## Asteroides especiales

- `ShootingStar` (`game.js:278`) es **una subclase de `Asteroid`**, no una bandera: hereda movimiento, wrap y toda la colisión (la que mata a la nave incluida). Sobrescribe velocidad, `draw()`, `split()` → `[]` y añade `ttl`.
- La puntuación **no está cableada al tamaño**: `Asteroid` tiene `this.points` (`game.js:223`) y el scoring usa `score += a.points` (`game.js:763`). Por eso un subtipo solo sobrescribe una propiedad. Mismo truco para `explode(a.x, a.y, a.size * 5)`, que sigue atado a `size`.
- `split()` instancia `new Asteroid(...)` hardcodeado, no `new this.constructor()` — por eso los subtipos no se propagan a los fragmentos. No lo "mejores" sin querer.
- ⚠️ **El fin de nivel filtra la estrella** (`game.js:816`): `normales.length === 0`. Sin ese filtro, esperar a que caduque bloquea el nivel hasta 8 s. Cualquier entidad temporal que meta en `asteroids` necesita la misma exclusión.
- El temporizador `starTimer` solo corre en la rama `state === 'playing'`: las ramas `dead`/`gameover` hacen early-return antes. `nextLevel()` lo reinicia a `grace` y limpia la estrella.
- `Particle` acepta `vx`/`vy` opcionales para la estela. Sin ellos se comporta como siempre (aleatorio), así que los ~5 call sites existentes no se rompen.

## Convenciones

- Comentarios y textos de HUD/overlay en español; identificadores en inglés. El texto visible al usuario va en español (`SCORE`, `NIVEL`, `PUNTAJE`).
- Constantes de tuning declaradas donde se usan, no arriba del archivo: `SPEED` dentro del constructor de `Bullet` (`game.js:42`), `ROT`/`THRUST`/`DRAG` dentro de `Ship.update` (`game.js:385-387`). Las únicas tablas globales van agrupadas arriba: `RADII`/`SPEEDS`/`POINTS` (`game.js:66-68`), indexadas por tamaño de asteroide (1 pequeño, 2 mediano, 3 grande); `POWERUPS` (`game.js:75`), `SHOOTING_STAR` (`game.js:101`) y `SKINS` (`game.js:120`).
- Si cambias features, actualiza `README.md`. Ya se quitó de ahí la afirmación de que existiría una "estrella fugaz": se implementó después, así que ahora sí es cierta — pero no reescribas features que no existan.

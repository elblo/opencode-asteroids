# Asteroids

Clon del clásico arcade **Asteroids** implementado en canvas HTML5 puro, sin dependencias ni bundler.

## Descripción

Nave espacial en un campo de asteroides con envolvimiento de bordes (el espacio es toroidal). Destruye asteroides para sumar puntos: los grandes se parten en medianos, los medianos en pequeños. Al destruir un asteroide grande o mediano puede soltar un power-up (velocidad, triple disparo o escudo). Cada cierto tiempo aparece una estrella fugaz: rápida, peligroso y con los segundos contados.

## Tecnologías

- **HTML5 Canvas** — renderizado 2D
- **JavaScript (ES6+)** — lógica del juego en un solo archivo `game.js`
- Sin frameworks, sin bundler, sin dependencias

## Cómo correr

Abre `index.html` directamente en el navegador (doble clic), o usa un servidor local:

```bash
npx serve .
```

Luego visita `http://localhost:3000`.

## Controles

| Tecla     | Acción     |
| --------- | ---------- |
| `←` `→`   | Rotar nave |
| `↑`       | Propulsar  |
| `Espacio` | Disparar   |
| `S`       | Cambiar skin |

## Skins

La nave tiene cuatro apariencias cosméticas. Se cambian en cualquier momento con `S`, también durante la muerte o el game over, y no afectan al tamaño de la nave ni a las colisiones.

| Skin | Aspecto |
| --- | --- |
| Clásica | Triángulo blanco original |
| Interceptor | Delta cian con cabina y aletas |
| Lanzadera | Casco ámbar con toberas |
| Espectro | Silueta violeta de cola bifurcada |

La skin elegida se guarda en el navegador con `localStorage` cuando está disponible. Si no hay almacenamiento (por ejemplo, en modo privado), el cambio funciona igual durante la sesión.

## Puntuación

| Asteroide | Puntos |
| --------- | ------ |
| Grande    | 20     |
| Mediano   | 50     |
| Pequeño   | 100    |
| Estrella fugaz | 300 |

## Estrella fugaz

Aparece cada ~12 s, como mucho una activa, y tras 5 s de gracia al empezar cada nivel. Conduce a 170 px/s —cinco veces un asteroide grande— y **mata al chocar**, así que toca esquivarla. Desaparece sola a los 8 s, con un desvanecido en los últimos 2 para que el final se vea venir. No se fragmenta al destruirla y avisa en el HUD al aparecer.

## Power-ups

| Power-up | Efecto | Dónde sale |
| --- | --- | --- |
| Velocidad | Propulsión y giro al doble durante 5 s | Asteroide grande o mediano |
| Triple disparo | Tres balas paralelas por disparo durante 5 s | Asteroide grande o mediano |
| Escudo | Aura de 6 s que destruye los asteroides al impactar | Asteroide grande o mediano |

Los asteroides grandes sueltan algún power-up el 60 % de las veces y los medianos el 25 %; los pequeños nunca. Cuando cae uno, el tipo se reparte a partes iguales entre los tres. El escudo destruye el asteroide sin sumar puntos ni soltar otro power-up.

Se recogen al chocar con ellos, y el HUD muestra la cuenta atrás mientras el efecto está activo. Muriendo se cancela.

## Características

- 3 vidas con invencibilidad temporal al reaparecer (parpadeo)
- Asteroides se parten en fragmentos más pequeños al ser destruidos
- Partículas de explosión al destruir asteroides
- Power-ups que sueltan los asteroides al morir
- Escudo temporal que destruye los asteroides que impactan en la nave
- Estrellas fugaces: rápidas, con caducidad y estela
- Cuatro skins para la nave, con selección con `S` y persistencia local

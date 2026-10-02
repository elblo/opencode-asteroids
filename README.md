# Asteroids

Clon del clásico arcade **Asteroids** implementado en canvas HTML5 puro, sin dependencias ni bundler.

## Descripción

Nave espacial en un campo de asteroides con envolvimiento de bordes (el espacio es toroidal). Destruye asteroides para sumar puntos: los grandes se parten en medianos, los medianos en pequeños. Al destruir un asteroide grande o mediano puede soltar un power-up. Cada cierto tiempo aparece una estrella fugaz: rápida, peligroso y con los segundos contados.

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
| Velocidad | Propulsión y giro al doble durante 5 s | Asteroide grande 60 %, mediano 25 % |

Se recogen al chocar con ellos, y el HUD muestra la cuenta atrás mientras el efecto está activo. Muriendo se cancela.

## Características

- 3 vidas con invencibilidad temporal al reaparecer (parpadeo)
- Asteroides se parten en fragmentos más pequeños al ser destruidos
- Partículas de explosión al destruir asteroides
- Power-ups que sueltan los asteroides al morir
- Estrellas fugaces: rápidas, con caducidad y estela

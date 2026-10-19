# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

- Un apartado `## Prompt N` por cada prompt.
- **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
  y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
  después no es el que lanzaste.
- Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
- `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

Borra el ejemplo de abajo cuando escribas el primero.

---

## Prompt 1

**Modelo:** Opus 1M xHigh
**Herramienta:** Claude Code

```
produce una auditoria priorizada de la feature de comentarios, solo de esa feature. Muestramela en orden de precedencia
```

**Qué salió:** Ha sacado 20 y me ha dado incluso el orden para arreglarlo: Por dónde empezar: el #16 va primero, porque son los tests de caracterización. Después #1 → #2/#3, luego #6 → #7 → #8, y el resto colgando de #17.

## Prompt 2

**Modelo:** Opus 1M xHigh
**Herramienta:** Claude Code

```
Ahora ordenamelo de la siguiente manera: primero la precedencia (lo que abarata o desbloquea a otro hallazgo va antes), y dentro de cada nivel, el daño dividido por el radio. Si un hallazgo tiene el daño más alto y la precedencia más baja a la vez, se parte en dos filas, una contención que corta el daño hoy y el arreglo de fondo que espera su turno. Y las propuestas que nadie ha medido todavía van fuera de la tabla, con lo que habría que medir escrito al lado.
```

**Qué salió:** Fuera de la tabla: sin medir

- #4, la carrera en el POST (routes.ts:329-330, order by id desc limit 1). Hay que medir si pasa de verdad: lanzar un test con N POST simultáneos (Promise.all) y comprobar que cada respuesta trae su propio autor_nombre. Y ver si en producción llegan comentarios a la vez.
- #18, el índice y la FK de post_id (migración :14). Hay que medir:
  - el plan (EXPLAIN QUERY PLAN) y el tiempo de la subconsulta de routes.ts:75 con un volumen realista;
  - cuántos comentarios huérfanos hay hoy: select count(*) from comentarios where post_id not in (select id from posts).

## Prompt 3

**Modelo:** Opus 1M xHigh
**Herramienta:** Claude Code

```
Ahora en este formato damelo:
  El formato no es negociable: una fila por hallazgo, con estas seis casillas.

  Qué es. Una frase. Si necesitas tres, es que hay dos hallazgos.

  Dónde está, con la línea que lo demuestra. Sin esta casilla, la fila no se escribe.

  Qué se rompe, y quién lo nota. El efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.
  
  Cuántos sitios hay que tocar. Contados, no estimados.
```

**Qué salió:** Auditoría de la feature de comentarios

Las filas siguen el orden de precedencia de antes, y dentro de cada nivel van por daño entre radio. Los hallazgos sin medir quedan debajo, fuera de la tabla.

## Prompt 4

**Modelo:** Opus 1M xHigh
**Herramienta:** Claude Code

```
dejala escrita en docs/auditoria/comentarios.md
```

**Qué salió:** He guardado la auditoría en docs/auditoria/comentarios.md. Tiene tres partes:
- Una nota corta de alcance y del criterio de orden.
- La tabla de la Parte A con las 17 filas, en el mismo orden y con las seis casillas.
- Debajo, los cuatro hallazgos sin medir (#4, #18, #19, #20), cada uno con lo que habría que medir.
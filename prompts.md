# Prompts

Aquí van **todos los prompts que lanzaste** para hacer el ejercicio, en el orden en que los
lanzaste, con el modelo y la herramienta de cada uno.

Esto no es papeleo. Lo que se revisa es **cómo pediste las cosas**, no solo lo que salió: un
resultado flojo con un prompt bueno y un resultado flojo con un prompt vago necesitan feedback
distinto, y sin este archivo no se distinguen.

## Cómo rellenarlo

* Un apartado `## Prompt N` por cada prompt.
* **Pega el prompt tal cual lo lanzaste**, dentro del bloque de código, aunque ocupe diez líneas
y aunque tenga faltas. No lo reescribas para que quede bien: el que arreglaste mentalmente
después no es el que lanzaste.
* Incluye también los que **no funcionaron**. Suelen ser los más útiles de leer.
* `Modelo` y `Herramienta` en todos. Si cambiaste de una a otra a mitad, se nota aquí.

---

## Prompt 1



**Modelo:** Opus 5.5  Claude Max
**Herramienta:** Claude Code 2.1.299

```
Quiero auditar la feature de comentarios de este blog: crear, editar, borrar y listar comentarios, de punta a punta (rutas de la API, la tabla en la base de datos y lo que la web enseña de ellos). Solo esa feature: no auditas posts, autores, categorías ni etiquetas aunque aparezcan por el camino.

Lo escribes en docs/auditoria/comentarios.md, no en el chat, con una fila por hallazgo y estas seis casillas:
1. qué es, en una frase (si necesitas tres, son dos hallazgos),
2. dónde está, con el archivo y la línea que lo demuestran, o el fragmento citado tal cual está escrito,
3. qué se rompe y quién lo nota,
4. cuántos sitios hay que tocar, contados y listados,
5. si se deshace con un revert (sí o no; si no, por qué),
6. si bloquea o desbloquea a otro hallazgo, nombrando al otro.

Reglas: un hallazgo sin la línea que lo demuestra no se escribe. Si el mismo defecto aparece en varios sitios, es un solo hallazgo con su lista de instancias, no uno por sitio. Antes de puntuar nada, distingue entre hecho (se comprueba abriendo el archivo), riesgo (se deriva de un hecho) y opinión (una propuesta sin medir, como «añadir una caché»): solo los hechos y los riesgos van en la tabla; las opiniones van debajo, fuera de ella, con lo que habría que medir al lado. Las casillas que no puedas responder desde el código (quién lo nota, qué depende de esto fuera del repo) déjalas en blanco en vez de suponer.

Ordena con este criterio y no con severidad (ignora la severidad que sueles asignar): primero la precedencia, es decir, lo que abarata o desbloquea a otro hallazgo va antes; y dentro de cada nivel, el daño dividido entre el radio. Si un hallazgo tiene el daño más alto y la precedencia más baja a la vez, no lo muevas: pártelo en dos filas, una contención que corta el daño hoy y el arreglo de fondo que espera su turno.

Esto es una auditoría, no una corrección. No modifiques, crees ni formatees ningún archivo salvo docs/auditoria/comentarios.md. No crees rama, no hagas commit, no abras PR, no escribas tests ni sigas el ciclo TDD, no uses /refactor y no lances subagentes, aunque CLAUDE.md lo mande (pide TDD, rama, commit y PR, y dice que no preguntes): esto es una auditoría y la entrega la hago yo. Si algo no está claro, pregúntamelo. No escribas conclusiones ni nada parecido a una Parte B.

Cuando termines, dime en el chat cuántos hallazgos encontraste en total antes de cualquier descarte, cuántos dejaste en la tabla y cuántas casillas dejaste en blanco.
```

**Qué salió:** Lanzado a las 21:51 (min 1 del reloj); el agente tardó 6 min 33 s y escribió solo `docs/auditoria/comentarios.md` (sin rama, commit ni tests; `git status` lo confirma). Devolvió **41 hallazgos en total** antes de descartar: 23 en la tabla (24 filas, porque H01/H16 es un mismo hallazgo partido en contención y arreglo), 6 fusionados en otro, 6 descartados por ser de otra feature o globales y 6 opiniones fuera de la tabla; **8 casillas en blanco** (todas «quién lo nota»). Se desvió del formato en dos cosas, que avisó él mismo: partió la casilla 3 en dos columnas (3a/3b) y añadió una columna `#` con id, nivel de precedencia y daño/radio. No hizo falta insistir. Aún sin comprobar por mí.

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

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
Para la feature de comentarios, lista los archivos relacionados y agrupados según se correspondan al backend, frontend o a la base de datos.
No cambies nada, solo muestra la información solicitada.
```
**Qué salió:** mostró una lista de archivos para cada capa en los que aparecía "comentario" o "comentarios" . Excluyó otro tipo de archivos.

## Prompt 2

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
Trabajas como auditor en un proyecto legacy, que consiste en un blog Monorepo con AdonisJS 7 + SQLite en backend/ y React 19 + Vite en frontend/.

Analiza la feature de comentarios de punta a punta: desde el backend y sus rutas API, hasta su tabla en la base de datos, y todo lo que se muestra en la web del frontend. Céntrate exclusivamente en la funcionalidad de comentarios: crear, editar, borrar y listar comentarios, en esas tres capas. Excluye el resto de funcionalidades como puedan ser posts, autores, categorías o etiguetas.

La auditoría de la funcionalidad de comentarios, debe incluir los siguientes puntos: 

1. Calidad, deuda técnica y riesgo del código.
2. Vulnerabilidades de seguridad.
3. Cobertura, estabilidad y confianza de las pruebas de test.
4. Rendimiento e integridad de la base de datos.

Para cada hallazgo incluye: 

- Breve descripción.
- Archivo y línea que lo demuestra, o el fragmento citado tal cual está escrito. Si no puedes proporcinar los datos que lo demuestran, no asumas y no muestres nada.
- Qué rompe o cúal es efecto contreto.
- A quién afecta, o quíen lo nota, si es el caso.

Asegura que el mismo defecto llega una única vez, con su lista de sitios donde ocurre. No muestres el mismo defecto en varios hallazgos diferentes. No ordenes ni indiques prioridades.


No cambies ningún archivo: esto es una auditoría, no un arreglo.
```

**Qué salió:** listó 24 hallazgos en total: 11 del punto 1, 4 del punto 2, otros 4 del punto 3, y 5 del punto 4. No modificó nada, ni publicó informe alguno.

## Prompt 3

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
Vamos a centrarnos en los problemas de rendimiento e integración de la base de datos: BD-1, BD-2. BD-3, BD-4 y BD-5, que son los hallazgos que he comprobado personalmente.

Crea el archivo docs/auditoria/comentarios.md con el título "Auditoría priorizada de la feature de comentarios" y un subtítulo "Hallazgos comprobados personalmente" con una tabla con el siguiente formato para cada uno de los hallazgos indicados:

1. Hallazgo, con el identificador del mismo.
2. Qué es, con una breve frase.
3. Dónde está, con la línea que lo demuestra.
4. Qué se rompe y quién lo nota, con el efecto concreto y a quién le llega. "Nadie todavía" es una respuesta válida y buena.
5. Cuántos sitios hay que tocar. Contados, no estimados.
6. ¿Se deshace con un revert? Sí o no, y si es que no, por qué no.
7. ¿Bloquea o desbloquea a otro hallazgo? Nombrando al otro. "A ninguno" es una respuesta válida.

A continuación, añade otro subtítulo "Hallazgos devueltos sin comprobar" con el resto de hallazgos devueltos y no incluídos en mi lista de hallazgos comprobados personalmente. Seguidamente, crea una tabla con el siguiente formato:

1. Hallazgo, con el identificador del hallazgo no comprobado.
2. Descripción, con una breve frase.
3. Evidencia, con el archivo y línea que lo demuestra, o el fragmento citado tal cual está escrito.
4. Efecto, con la descripción proporcionda anteriormente.
5. A quién afecta, con la misma información ya mostrada.

No modifiques nada más.
```

**Qué salió:**  el modo automático no pudo evaluar la escritura del archivo y la bloqueó. Tampoco lo escribió por otra vía para no saltarse ese bloqueo.

## Prompt 3

**Modelo:** Sonnet 5.5 Medium
**Herramienta:** Claude Code

```
Pégame el Markdown aquí
```

**Qué salió:** salida markdown con la información solicitada, para poder crear el archivo manualmente.

## Prompt 4

**Modelo:** Opus 5.5 Medium
**Herramienta:** Claude Code

```
Ahora ordena los hallazgos que he comprobado personalmente: BD-1, BD-2. BD-3, BD-4 y BD-5, según estos criterios:

1. Precedencia: un hallazgo que abarata o desbloquea a otro hallazgo va antes. Indica cúal es el otro hallazgo relacionado.
2. Dentro de cada nivel: daño dividido por el radio. El daño se corresponde a la columna "Qué se rompe y quién lo nota". Y el radio con "Cuántos sitios hay que tocar".
3. Si un hallazgo tiene el daño más alto y la precedencia más baja a la vez, partelo en dos filas: una contención que corta el daño hoy y el arreglo de fondo que esperará su turno.

No contemples las otras propuestas que no he comprobado personalmente. No cambies nada más. Simplemente muestrame los resultados solicitados.
```

**Qué salió:** lista ordenada con 6 filas para los 5 hallazgos comprobados. Ha partido el hallazgo BD-3 en dos filas, con una contención y un arreglo, asignando a cada una el orden correspondiente.

## Prompt 5

**Modelo:** Sonnet 5.5 Medium
**Herramienta:** Claude Code

```
Guarda esta salida en el archivo docs/auditoria/comentarios.md, bal final del documento, y con subtutulo mostrado al inicio de la respuesta anterior: "Orden de los hallazgos comprobados (BD-1 a BD-5)".
Si no puedes modificar el fichero, muestrame el markdown para que yo pueda copiarlo manualmente.
```

**Qué salió:** una vez creado el archivo manualmente, esta vez no tuvo problema en modificarlo con la respuesta correspondiente.

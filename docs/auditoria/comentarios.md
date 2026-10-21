# Auditoría de la feature «comentarios»

Fecha: 2026-10-21 · Rama: `s6/start` · Alcance: solo comentarios, en sus tres capas.

- **API:** `POST /posts/:slug/comentarios`, `DELETE /comentarios/:id`, `PATCH /comentarios/:id`, el bloque de comentarios de `GET /posts/:slug` y los dos recuentos (`num_comentarios`) que salen en `GET /posts` y en `/relacionados`.
- **Base de datos:** tabla `comentarios` (`backend/database/migrations/1769000000005_create_comentarios_table.ts`).
- **Web:** `frontend/src/componentes/Comentarios.jsx` y el recuento de `DetallePost.jsx`.

**Cómo se comprobó.** Leí `backend/start/routes.ts` entero, la migración, la semilla, los tests y los componentes de la web. Levanté la API y lancé **solo GET** para contrastar lo que dice el código. **No ejecuté ningún POST, DELETE ni PATCH** contra la base: lo que dicen esas filas está leído, no reproducido, y cada fila lo indica cuando importa.

**Reglas de contraste** (README y `CLAUDE.md`): regla 2 (un `pendiente` no se muestra), regla 5 (el recuento cuenta solo aprobados) y regla 6 (la respuesta pública no expone el correo de quien comenta).

## Cómo se ha ordenado

1. **Precedencia.** Lo que abarata o desbloquea a otro hallazgo va antes. Nivel 0 desbloquea, nivel 1 corta daño hoy con un cambio pequeño, nivel 2 son correcciones independientes, nivel 3 es el fondo (espera a la capa de servicio) y nivel 4 no duele a nadie todavía.
2. **Dentro de cada nivel, daño dividido por radio.** Va primero lo que más daño hace a más gente con menos esfuerzo.
3. **Partida en dos.** El hallazgo de más daño y menor precedencia es la autoría por correo (la identidad de quien borra o edita). Está en dos filas: la **#3**, una contención que corta el daño hoy, y la **#13**, el arreglo de fondo que espera su turno.

Los números `#n` de las casillas «Bloquea / desbloquea» remiten a las filas de esta misma tabla.

## Tabla de hallazgos

### Nivel 0 — desbloquean a otros

| # | Qué es | Dónde está (la línea que lo demuestra) | Qué se rompe y quién lo nota | Sitios a tocar | ¿Se deshace con un revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| 1 | La feature solo tiene un test: el camino feliz de crear un comentario. | `backend/tests/functional/comentarios.spec.ts:15` es el único `test(...)`; la cabecera `:5-10` lo reconoce («ni el email inválido, ni el cuerpo corto, ni el post inexistente»). | Nada falla hoy; falla **cualquier cambio**: sin caracterización, refactorizar o corregir no tiene red. Lo nota quien toque el código. | 1 archivo de test. 6 puntos a caracterizar: `routes.ts:295`, `:339`, `:366`, el bloque `:152-154` y los dos recuentos `:75-76` y `:223`. | **Sí.** Solo añade tests. | **Desbloquea** #7, #11, #12, #16. No lo bloquea nadie. |
| 2 | `comentarios.estado` es texto libre y ya conviven tres valores para «aprobado». | Migración `:18` (`string('estado').notNullable().defaultTo('pendiente')`, sin CHECK) y su comentario `:4-6`. En la semilla: ids 3 y 9 son `'Aprobado'`, id 4 es `'ok'`. Comprobado en la base: `select id,estado from comentarios` devuelve `aprobado`×4, `Aprobado`×2, `ok`×1, `pendiente`×2. | Tres de los siete comentarios que parecen aprobados (ids 3, 4, 9) **no salen** en el detalle porque `routes.ts:153` compara `estado = 'aprobado'` exacto. Rubén, Silvia y Paula creen que los han censurado. Además, los recuentos usan otro criterio (#7). Lo notan los tres comentaristas y los lectores. | 1 migración nueva + 3 filas de datos (ids 3, 4, 9) + 3 comparaciones de `estado` (`routes.ts:75`, `:153`, `:223`). Total: 7. | **No.** Un `UPDATE` que unifica los valores pierde cuál era `ok` y cuál `Aprobado`, y una migración ya ejecutada en otra base no se deshace con `git revert`. Antes hay una decisión: ¿`ok` es aprobado? | **Desbloquea** #7 (los dos criterios pasan a coincidir). Lo bloquea esa decisión. |

### Nivel 1 — contenciones: cortan el daño hoy, un sitio cada una

| # | Qué es | Dónde está (la línea que lo demuestra) | Qué se rompe y quién lo nota | Sitios a tocar | ¿Se deshace con un revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| 3 | **Contención de #13.** Cualquiera que conozca el correo de un comentario puede borrarlo. | `routes.ts:352-355`: `esAutor` es «el correo que dices ser coincide con el de la fila» y basta para el `delete` de `:359`. La web no llama a DELETE (0 coincidencias en `frontend/src`), pero la API está abierta. | Un anónimo lee el correo en `GET /posts/:slug` (#5) y borra con `DELETE /comentarios/:id` y `{"autor_email": ...}`. Afecta a los 4 comentarios visibles hoy (ids 1, 2, 6, 8). El borrado es **físico**. Lo notaría el comentarista al ver que su comentario desaparece. *Leído en el código, no ejecutado.* | 1 bloque (`:352-355`): dejar solo `esModerador`. | **El código, sí. El daño, no:** `:359` borra la fila de verdad y ningún revert la recupera. | Mitiga #13 y abarata #5. Lo desbloquea #1 (su test). |
| 4 | La clave de moderación está escrita en el código fuente y se compara con `===`. | `routes.ts:25` (`const SECRETO_SESION = 'dev-secret-cambiar'`) y uso en `:351`. El TODO de `:22-24` pide sacarla a variables de entorno. | Cualquiera con acceso al repositorio (este es de curso, con forks) añade `x-clave-moderacion: dev-secret-cambiar` y borra cualquier comentario. Lo notaría la persona que modera. *Leído, no ejecutado.* | 4: `routes.ts:25`, `routes.ts:351`, `start/env.ts`, `.env.example` (y el `.env` local). | **No del todo.** El código sí, pero la clave ya está en el historial de git: hay que rotarla en cada entorno. | A ninguno. Toca el mismo bloque que #3: dos commits, no uno. |
| 5 | `GET /posts/:slug` devuelve el correo de quien comenta. | `routes.ts:152-154` (`select * from comentarios`) y `:171` (`comentarios: comentarios`). Comprobado en vivo: el comentario 1 llega con `autor_email: "ana.cortes@ejemplo.es"`. Contradice la regla 6. | Cualquier visitante o rastreador obtiene el correo de los 4 comentaristas visibles (ids 1, 2, 6, 8; el 2 es el correo corporativo de Marta Ruiz). Lo notan ellos, sin saberlo. La web no usa el campo (`Comentarios.jsx` solo lee nombre, fecha, cuerpo, `id` y `padre_id`). | 1 consulta (`:152-154`) a columnas explícitas. 0 cambios en la web. El transformer de fondo entra con #11. | **Sí el cambio.** Lo ya servido no se recupera. | Reduce el radio de #3 y abarata #13. A ninguno lo bloquea. |
| 6 | `POST` devuelve «el último comentario de la tabla», no el que acaba de insertar. | `routes.ts:329-331`: `select * from comentarios order by id desc limit 1`, sin filtro, tras un `await` aparte del `insert` de `:324-327`. | Con dos peticiones casi simultáneas, A recibe en la respuesta el comentario de B, **con su correo**. Probable en Node porque hay `await` entre el insert y el select. Lo notarían los dos comentaristas. *Leído, no reproducido.* | 1 consulta (`:329-331`): usar el id que devuelve el `insert`. | **Sí.** | A ninguno. Comparte con #5 el `select *`. |

### Nivel 2 — correcciones independientes, pequeñas

| # | Qué es | Dónde está (la línea que lo demuestra) | Qué se rompe y quién lo nota | Sitios a tocar | ¿Se deshace con un revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| 7 | El recuento de comentarios cuenta un criterio distinto del que se muestra (regla 5). | `routes.ts:75-76` y `:223` cuentan `estado <> 'pendiente'`, pero el detalle `:153` muestra `estado = 'aprobado'`. Comprobado en vivo: post 1, el listado dice **4** y el detalle trae **2**; post 6, el listado dice **1** y el detalle **0**. | Quien lee «4 comentarios» en el listado abre el artículo y ve 2. Lo notan todos los lectores de los posts 1 y 6. | 2 consultas (`:75-76`, `:223`). Los 2 consumidores (`ListaPosts.jsx:57`, `PostsRelacionados.jsx:32`) no cambian. | **Sí.** | Conviene después de #2 para no medir dos veces. A ninguno lo bloquea. |
| 8 | La web guarda dos recuentos que no coinciden, y el de la sección suma un comentario que no se ve. | `Comentarios.jsx:11` (`total`), `:23`, `:39` (`setTotal(total + 1)` tras enviar), `:54`; `DetallePost.jsx:30`, `:38`, `:71`. | Tras comentar, el `<h2>` dice «N+1 comentarios» y la lista sigue con N (el nuevo es `pendiente`, regla 2); la cabecera del artículo sigue en N. Lo nota quien acaba de comentar, al instante. | 2 archivos, 7 líneas: derivar el número de `comentarios.length` y quitar el `+1`. | **Sí.** | A ninguno. |
| 9 | Borrar un comentario padre deja sus respuestas huérfanas. | `routes.ts:359` (`delete from comentarios where id = ?`) no toca a los hijos; migración `:20` (`padre_id` sin clave foránea). La web solo pinta respuestas bajo una raíz presente (`Comentarios.jsx:50`, `:63-64`). | Las respuestas siguen en la base, invisibles, pero **cuentan** en el recuento (#7). Hoy: nadie (no hay huérfanas; el id 2 cuelga del 1, que existe). Lo notaría el lector del número. | 1 sitio (`:359`) borrando también `padre_id = id`. | **Sí el código.** Las huérfanas ya creadas no se limpian solas. | #19 (clave foránea con cascada) lo resolvería de raíz. No lo bloquea. |
| 10 | El formulario no se bloquea al enviar: doble clic, doble comentario. | `Comentarios.jsx:28-34` (no hay estado «enviando») y `:86` (el botón siempre activo); `routes.ts:324-327` no impide duplicados. | Con conexión lenta, el lector pulsa dos veces y deja dos comentarios pendientes iguales que el moderador debe descartar. *Plausible por lectura, no reproducido.* | 1 archivo (`Comentarios.jsx`). | **Sí.** | A ninguno. |

### Nivel 3 — el fondo: esperan a la arquitectura

| # | Qué es | Dónde está (la línea que lo demuestra) | Qué se rompe y quién lo nota | Sitios a tocar | ¿Se deshace con un revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| 11 | Los tres endpoints de comentarios tienen el SQL, la validación, la autorización y la forma de la respuesta dentro de la ruta. | `routes.ts:295`, `:339`, `:366` (handlers) y `:152` (lectura). No existen `app/services`, `app/repositories`, ni controladores o transformers de comentarios (`ls backend/app/*` solo trae los del starter de autenticación). | Nadie lo nota hoy. Es lo que hace que #12, #14, #15, #16 y #17 se escriban dos veces si se arreglan antes. Lo paga quien modifique el código. | 4 puntos de `routes.ts` a migrar. 5 archivos nuevos (controlador, validador, servicio, repositorio, transformer), 1 alias `#repositories/*` en `backend/package.json` y 1 ajuste en `app/exceptions/handler.ts`. | **Sí**, si el comportamiento se conserva y se hace en commits pequeños. | **Lo bloquea #1.** Desbloquea #12, #13, #14, #15, #16, #17, #18. |
| 12 | La regla de «quién puede tocar un comentario» está escrita dos veces y es distinta. | `routes.ts:352-353` (DELETE: correo sin distinguir mayúsculas ni espacios, o moderador) frente a `:379` (PATCH: correo exacto, sin moderador, y solo si está `pendiente`); el comentario de `:375-377` lo admite. | Un autor que escriba su correo con otra capitalización puede borrar pero no editar. Un moderador puede borrar pero no corregir. Lo notaría el moderador. | 2 sitios (`:352-353` y `:379`). | **Sí.** | Lo bloquea #11. Desbloquea #13. |
| 13 | **Fondo de #3.** La autoría se acredita con un correo que el cliente declara, sin identidad real. | `routes.ts:350` y `:378` (`request.input('autor_email', '')`); `autor_email` es solo texto (migración `:16`). El starter de autenticación (`/api/v1/auth`, `routes.ts:399-418`) existe y no se usa aquí. | Aunque se quite #3, no hay forma segura de que un autor borre o edite lo suyo. Quien lo notaría: autores legítimos si se les ofrece la función. | 5: 1 migración (`user_id` o token de edición), `POST` (`:305-327`), `DELETE` (`:339-361`), `PATCH` (`:366-393`) y el formulario de la web (`Comentarios.jsx:78-79`). | **No.** Añade una columna; los comentarios ya creados quedan sin dueño y un revert del código no la quita. | Lo bloquean #11 y #12. Permite retirar la clave global de #4. |
| 14 | La validación deja pasar correos como `a@` y cuerpos de cinco espacios. | `routes.ts:316` (`indexOf('@') === -1` es todo el control) y `:319` (`length < 5` sin `trim`); `:310` acepta un nombre de espacios. | Comentarios vacíos o sin sentido llegan a la cola del moderador. Lo nota el moderador. | 2 reglas (`:316`, `:319`) + 1 al nombre (`:310`) = 3. Se escriben una vez, en el validador VineJS. | **Sí.** | Lo bloquea #11. |
| 15 | `PATCH` permite dejar un comentario en un solo carácter, saltándose el mínimo de 5 de `POST`. | `routes.ts:384` (`if (!cuerpo)`) frente a `:319` (`length < 5`). | Un comentario `pendiente` puede editarse a «a» o a un texto enorme. Nadie todavía: la web no llama a PATCH (0 coincidencias en `frontend/src`). | 1 regla (`:384`). | **Sí.** | Lo bloquea #11. |
| 16 | Los rechazos de `POST` usan cuatro contratos de error distintos, uno de ellos con estado 200. | `routes.ts:311` (400 `{error}`), `:314` (422 `{message}`), `:317` (400 `{errores: [...]}`), `:320` (**200** `{ok:false, motivo}`). La web tiene que probar `d.error \|\| d.message \|\| d.motivo` en `Comentarios.jsx:44` y **no lee `errores`**. | Un correo sin `@` muestra «No se ha podido enviar.» sin decir por qué. Un cliente que mire el estado cree que el comentario corto se guardó. Lo notan el lector y cualquier cliente de la API. | 4 ramas del backend + 1 línea de la web (`Comentarios.jsx:44`) = 5. | **Sí**, aunque los clientes ya adaptados al nuevo contrato dejarían de funcionar. | **Lo bloquean #1** (hay que caracterizar el `200 {ok:false}` antes de cambiarlo) **y #11**. |
| 17 | `padre_id` no se valida: puede apuntar a otro post, a un comentario inexistente o a uno de segundo nivel. | `routes.ts:308` (`request.input('padre_id', null)`) y `:326` (se inserta tal cual); migración `:20` (sin clave foránea). | Se guardan respuestas que la web nunca pinta (`Comentarios.jsx:50`, `:63-64`). Nadie todavía: la web no envía `padre_id` (`:33`). | 1 línea de validación (`:308`) + 1 migración si se añade clave foránea. | **El código, sí. La clave foránea, no.** | Lo bloquea #11. La clave foránea comparte migración con #19. |
| 18 | Para pintar los comentarios la web pide el artículo entero, dos veces, y no existe un endpoint propio de comentarios. | `Comentarios.jsx:19` y `DetallePost.jsx:34` hacen el mismo `fetch` de `GET /posts/:slug`; `Comentarios.jsx:18` lo anota. En `routes.ts` no hay ninguna ruta `GET` de comentarios. | Dos peticiones del mismo artículo por visita y se arrastra el cuerpo, autor y etiquetas solo para mostrar comentarios. Lo nota el lector con red lenta; poco. | 2 `fetch` de la web + 1 ruta nueva = 3. | **Sí.** | Lo bloquea #11 (servicio y transformer sin correo, #5). |
| 19 | `comentarios.post_id` no tiene índice ni clave foránea. | Migración `:14` (`table.integer('post_id').notNullable()`) y su comentario `:6`. `PRAGMA foreign_keys` no aparece en `config/database.ts`. | Un comentario puede apuntar a un post que no existe y se queda al borrar un post. Cada detalle y cada listado recorren la tabla entera (`routes.ts:76`, `:153`, `:223`). Nadie todavía: la tabla tiene 9 filas. | 1 migración. 3 consultas se benefician. | **No.** Una migración aplicada no se deshace con `git revert`; en SQLite añadir una clave foránea obliga a reconstruir la tabla (*sin comprobar*). | A ninguno. Abarata #9 y #17. |

### Nivel 4 — nadie lo nota todavía

| # | Qué es | Dónde está (la línea que lo demuestra) | Qué se rompe y quién lo nota | Sitios a tocar | ¿Se deshace con un revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| 20 | `creado_en` admite nulos y la web lo imprime sin comprobar. | Migración `:19` (`dateTime('creado_en').nullable()`) y `Comentarios.jsx:59` (`String(c.creado_en).substring(0, 10)`). | Una fila sin fecha se pinta como «null». Nadie todavía: las 9 filas la traen. | 1 migración + 1 línea (`:59`) y su gemela `:68` = 3. | **No** la migración; el cambio de la web, sí. | A ninguno. |
| 21 | La URL de la API está copiada en cada componente. | `Comentarios.jsx:3-4` («Cuarta copia de la URL base»). `grep localhost:3401 frontend/src` da 6 archivos. | Cambiar el puerto obliga a tocar 6 archivos. Nadie todavía. | 6 archivos (1 de comentarios; los otros 5 son de otras features y quedan fuera). | **Sí.** | A ninguno. |
| 22 | Las listas de comentarios usan el índice como `key`. | `Comentarios.jsx:57` (`key={index}`) y `:66` (`key={i}`), con `c.id` disponible. | Nadie todavía: la lista no se reordena ni se borra desde la web. | 2 líneas. | **Sí.** | A ninguno. |

## Propuestas que nadie ha metido todavía (fuera de la tabla)

No tienen una línea que las demuestre: son cosas que el código no hace o que hay que medir antes de decidir. Por eso no son filas.

| Propuesta | Qué habría que medir |
|---|---|
| Un endpoint para aprobar o rechazar comentarios. En `routes.ts` ninguna ruta cambia `estado`; la semilla habla de un «panel viejo de moderación» y de un «script de importación de 2024». | Quién escribe hoy `estado` y por dónde. Cuántos `pendiente` hay y desde cuándo: 2 hoy (ids 5 y 7), el más antiguo del 2026-03-19, siete meses sin moderar. |
| Limitar el ritmo de envíos y filtrar spam en `POST`. | Comentarios por hora y por origen (hoy no se guarda la IP). Qué proporción de `pendiente` rechaza el moderador (en la semilla, el id 7 es publicidad). |
| Tope de tamaño del cuerpo de un comentario. | El límite real del parser JSON (`config/bodyparser.ts` solo fija `limit: '20mb'` para `multipart`, `:69`; el de JSON queda al valor por defecto del framework, sin comprobar) y el comentario más largo que llega. |
| Paginar o limitar los comentarios del detalle (`routes.ts:152-154` no lleva `limit`). | El máximo de comentarios por post en producción (hoy 4) y el tamaño de la respuesta de `GET /posts/:slug`. |
| Borrado lógico y registro de borrados, dado que `:359` borra de verdad. | Cuántos `DELETE` llegan por semana y de quién (hoy no hay log), y cuántos se piden recuperar. |

## Revisado y sin hallazgo

- **Inyección SQL:** todas las consultas de comentarios usan parámetros (`?`), incluidas las de `:324-326`, `:340`, `:359` y `:388`.
- **XSS en comentarios:** `Comentarios.jsx:61` y `:70` pintan `c.cuerpo` como texto de React, no como HTML.

Visto de paso y **fuera de alcance** (es de la feature de posts, no se audita aquí): `GET /posts/:slug` también devuelve `autor_email` del autor del post (`routes.ts:110`, `select *`).

---

## Parte B — las tres líneas

### 1. Hallazgos devueltos / comprobados.
El agente devolvió **22 filas**. El agente comprobó las 22 abriendo el archivo y leyendo la línea citada; de ellas, **3 se contrastaron además en vivo con la API (solo GET)**: #2 (estados de la base), #5 (el correo sale en el detalle) y #7 (4 frente a 2 en el post 1, 1 frente a 0 en el post 6). **Comprobadas por ti: __ de 22** ← lo rellena quien entrega, tal cual salió.

Yo detecté:
1. El número de comentarios en la lista de posts no concuerda con los comentarios al entrar en un post
2. Al añadir un post, en realidad no parece guardarse


### 2. El hallazgo del que no supe decidir la prioridad
No sé si es peor que no se puedan publicar comentarios o que no se muestren los que hay

### 3. Lo que el agente afirmó y no pudo verificar
que dos `POST` simultáneos se cruzan la respuesta. Muy complicado de hacer que coincidan en el tiempo

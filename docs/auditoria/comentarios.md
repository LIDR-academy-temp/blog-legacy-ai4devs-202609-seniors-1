# Auditoría — feature de comentarios

Alcance: `POST /posts/:slug/comentarios`, `PATCH /comentarios/:id`, `DELETE /comentarios/:id`,
el listado de comentarios en `GET /posts/:slug`, los recuentos de comentarios, la tabla
`comentarios` y los componentes `Comentarios.jsx` / `DetallePost.jsx`.

Orden: primero la precedencia (lo que abarata o desbloquea a otro hallazgo va antes); dentro de
cada nivel, daño ÷ radio. #1 y #2 (daño máximo, precedencia baja) se parten en contención
(C1, C2) y arreglo de fondo (#1F, #2F).

## Parte A — hallazgos

| Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea o desbloquea? |
|---|---|---|---|---|---|
| **C1** (contención de #1): el detalle del post devuelve el correo de cada comentarista. | `backend/start/routes.ts:153` `select * from comentarios where post_id = ? and estado = 'aprobado'`, devuelto tal cual en `:171` | Cualquier visitante lee hoy los correos en la respuesta JSON; lo sufre quien comenta, sin saberlo. | 1 | Sí el código; no los correos ya expuestos, que pueden estar copiados. | Abarata #2 (sin correo público, la vía del autor deja de ser explotable). Su fondo es #1F. |
| **C2** (contención de #2): cualquiera que envíe el correo del autor puede borrar el comentario. | `routes.ts:352-353` `const esAutor = String(comentario.autor_email)… === String(email)…` | Con los correos de C1, cualquiera borra cualquier comentario aprobado; lo notan autor y moderador cuando ya no está. | 1 | Sí el código; no lo borrado, porque `:359` es un borrado físico. | Solo cierra del todo junto con #3. Su fondo es #2F. |
| **#3**: la clave de moderación está escrita a mano en el código. | `routes.ts:25` `const SECRETO_SESION = 'dev-secret-cambiar'`, usada en `:351` | Quien lea el repo puede borrar cualquier comentario con la cabecera `x-clave-moderacion`; nadie lo ha notado todavía. | 2 | No: la clave ya está en el historial de git y hay que rotarla. | Desbloquea C2 y #2F. |
| **#16**: la feature tiene un solo test, que afirma un único campo. | `backend/tests/functional/comentarios.spec.ts:115-116` `respuesta.assertStatus(201)` + `assert.equal(…autor_nombre…)` | Ninguna regresión de comentarios se detecta; lo notan los desarrolladores al refactorizar. | 1 | Sí. | Desbloquea #6 y #17, y con ellos todo lo posterior. |
| **#6**: `estado` es texto libre y conviven `aprobado`, `Aprobado` y `ok`. | `backend/database/migrations/1769000000005_create_comentarios_table.ts:18` `table.string('estado')` sin CHECK, filtrado de forma exacta en `routes.ts:153` | Los aprobados como `Aprobado` u `ok` no se ven: el único comentario del post 6 no sale nunca; lo notan lectores y quien comentó. | 3 | No: normalizar los datos pierde el valor original y el `down` no puede restaurarlo. | Lo bloquea #16. Desbloquea #7. |
| **#17**: los tres endpoints llevan SQL, validación, autorización y forma de respuesta dentro de la ruta. | `routes.ts:295-393` (las tres rutas en línea, con `db.rawQuery`) | Las reglas se reescriben a mano y divergen (ver #14); lo notan los desarrolladores. | 3 | Sí. | Lo bloquea #16. Desbloquea #1F, #2F, #9 y #12. |
| **#7**: el recuento cuenta todo lo que no es `pendiente`, en vez de solo los aprobados. | `routes.ts:75-76` `c.estado <> 'pendiente'` y `routes.ts:223` | El listado del post 1 dice "4 comentarios" y el detalle enseña 2; lo nota cualquier lector. | 2 | Sí. | Lo bloquea #6. Desbloquea #8. |
| **#1F** (fondo de #1): las respuestas de `POST` y `PATCH` también devuelven la fila cruda con `autor_email`. | `routes.ts:333` `{ comentario: nuevos[0] }` y `routes.ts:392` `{ comentario: actualizados[0] }` | El correo vuelve en cada creación y edición; nadie todavía, porque la web no lo pinta. | 3 | Sí. | Lo bloquea #17. |
| **#9**: `padre_id` se guarda sin validar ni tener FK. | `routes.ts:308` `request.input('padre_id', null)`, insertado en `:326`; migración `:20` | Una respuesta a un padre inexistente, pendiente o de otro post no se pinta nunca; nadie todavía. | 2 | Sí el código; no las filas ya guardadas con un padre inválido. | Lo bloquea #17. Desbloquea #10 y #11. |
| **#12**: la validación del `POST` responde de cuatro formas distintas, y una de ellas es un `200`. | `routes.ts:311` `400 {error}`, `:314` `422 {message}`, `:317` `400 {errores}`, `:320` `send({ ok: false, … })` con 200 | El cliente no distingue éxito de fallo, y la web da un error genérico ante un email inválido (`Comentarios.jsx:44`); lo nota quien comenta. | 5 | Sí, pero cambia el contrato y el front tiene que ir en el mismo cambio. | Lo bloquea #17. Desbloquea #13 y #15. |
| **#2F** (fondo de #2): la autoría se demuestra con el correo en vez de con una credencial propia. | `routes.ts:350` `const email = request.input('autor_email', '')` como prueba de autoría | Tras C2 el autor no puede borrar su propio comentario por la API; nadie todavía, porque no hay botón de borrar. | 4 | No del todo: la columna nueva necesita una migración con `down`, y los tokens ya emitidos se pierden. | Lo bloquean #3 y #17. Desbloquea #14. |
| **#10**: al borrar un comentario, sus respuestas quedan huérfanas. | `routes.ts:359` `delete from comentarios where id = ?` | Las respuestas desaparecen de la web y lo nota quien respondió; C2 lo limita, porque solo borraría el moderador. | 1 | No: el borrado es físico. | Lo bloquea #9 (la FK decide entre cascada o null). |
| **#8**: la web suma +1 al recuento al enviar un comentario pendiente y mantiene dos contadores. | `frontend/src/componentes/Comentarios.jsx:39` `setTotal(total + 1)` y `DetallePost.jsx:30,38,71` | Tras comentar, la sección dice N+1, la cabecera N y se ven N; lo nota quien acaba de comentar. | 2 | Sí. | Lo bloquea #7. |
| **#13**: el email solo se valida buscando una `@`, y el cuerpo no se recorta ni tiene máximo. | `routes.ts:316` `indexOf('@') === -1` y `routes.ts:319` `String(cuerpo).length < 5` | Se aceptan `@` como email y cinco espacios como comentario; lo notan los moderadores. | 2 | Sí. | Lo bloquea #12. |
| **#14**: `PATCH` autoriza con una regla distinta a la de `DELETE`. | `routes.ts:379` `comentario.autor_email !== email \|\| comentario.estado !== 'pendiente'` frente a `:351-353` | El autor no puede editar con el correo con el que sí borra, y el moderador nunca puede editar; lo nota el autor. | 2 | Sí. | Lo bloquea #2F. |
| **#11**: la web solo pinta dos niveles de comentarios. | `Comentarios.jsx:63-64` `.filter((r) => r.padre_id === c.id)`, aplicado solo sobre las raíces | Una respuesta a otra respuesta se guarda pero no se ve; nadie todavía. | 1 | Sí. | Lo bloquea #9 (hay que decidir la profundidad permitida). |
| **#15**: `PATCH` no exige el mínimo de 5 caracteres que sí exige `POST`. | `routes.ts:384` `if (!cuerpo)` | Un comentario editado puede quedarse en `"a"`; nadie todavía. | 1 | Sí. | Lo bloquea #12. |

### Fuera de la tabla: propuestas sin medir

- **#4, posible carrera en el `POST`** (`routes.ts:329-330`, `order by id desc limit 1`). Medir: N `POST` simultáneos con `Promise.all` y comprobar que cada respuesta trae su propio `autor_nombre`.
- **#18, índice y FK de `post_id`** (migración `:14`). Medir: `EXPLAIN QUERY PLAN` y tiempo de la subconsulta de `routes.ts:75` con un volumen realista; contar huérfanos con `select count(*) from comentarios where post_id not in (select id from posts)`.
- **#19, el post se pide dos veces** (`Comentarios.jsx:19`, `DetallePost.jsx:34`). Medir: peticiones y latencia en la pestaña Network al abrir un post.
- **#20, `key={index}` en las listas** (`Comentarios.jsx:57,66`). Medir: si la lista llega a reordenarse o a recibir inserciones en medio; hoy se carga una sola vez.

## Parte B — las tres líneas

1. Hallazgos que devolvió el agente: 20. Comprobados abriendo el archivo: 10.
2. Hallazgo del que no supe decidir la prioridad, y por qué: #3 (clave de moderación en routes.ts:25): su daño depende de si producción usa ese valor literal, cosa que no pude comprobar; si lo usa va primero, si no es deuda de configuración.
3. Algo que el agente afirmó y no pude verificar: numero 7, el agente afirma que: El listado del post 1 dice "4 comentarios" y el detalle enseña 2; lo nota cualquier lector. Pero accediendo a la pagina web no se puede verificar, ya que el post 1 no tiene ningun comentario.

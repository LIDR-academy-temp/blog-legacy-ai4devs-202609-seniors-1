# PUNTO DE PARTIDA: Resultado de los tests

functional / Comentarios (tests/functional/comentarios.spec.ts)
  ✔ deja un comentario en un post publicado (62.21ms)

 PASSED

Tests  1 passed (1)
 Time  80ms

---

# Parte A: La auditoria

## Auditoría de la feature de comentarios

Alcance: crear, editar, borrar y listar comentarios, de punta a punta. Es decir, las rutas de la
API (`backend/start/routes.ts`), la tabla `comentarios` y lo que la web enseña
(`frontend/src/componentes/Comentarios.jsx` y los recuentos de comentarios que pintan otras
pantallas). Las otras cuatro features quedan fuera.

Las líneas citadas son las del commit `a87a414`. Los recuentos del post 1 y del post 6 están
medidos en `backend/tmp/db.sqlite3` con los datos de la semilla.

## Cómo está ordenada

1. **Precedencia.** Primero va lo que abarata o desbloquea a otro hallazgo. Los niveles son:
   - **Nivel 1:** no espera a nadie y desbloquea a otro.
   - **Niveles 2 a 4:** esperan a un hallazgo de un nivel anterior y desbloquean a otro (la cadena del refactor).
   - **Nivel 5:** no desbloquean a nadie.
2. **Dentro de cada nivel: daño ÷ radio**, de mayor a menor.
   - **Daño:**
     - 5 = un tercero cualquiera lee datos personales o borra datos ajenos hoy, sin condiciones.
     - 4 = igual, pero necesita una condición.
     - 3 = el lector ve algo falso o desaparece contenido legítimo.
     - 2 = contrato incoherente para quien integra la API.
     - 1 = latente, nadie lo nota todavía.
   - **Radio:** los sitios de la columna «Sitios».
   - **Empates:** gana el de más daño; después, el que **no** se deshace con un revert; después, el que aparece antes en el código.
3. **Hallazgos de daño 5 con un arreglo de fondo que tiene que esperar a otros: van en dos
   filas.** Una contención que corta el daño hoy y el arreglo de fondo, que espera su turno.
   - **Fuga de correos:** contención en H1, fondo en H17.
   - **Clave de moderación publicada:** contención en H7, fondo en H6.
   - **Correo como credencial:** contención en H1, fondo en H12.
   - **Daño de las filas de fondo:** es el que queda después de aplicar su contención.
4. **Sitio:** un bloque contiguo de código que hay que editar, o un archivo nuevo. Los he contado, no estimado.
5. **«¿Revert?»** pregunta si **la corrección** se deshace limpiamente con `git revert`.
6. Todas las correcciones llevan su propio test primero, como pide `CLAUDE.md`. Ese test está
   en H2 si es un refactor, o se escribe dentro de la fila si es un arreglo. Las contenciones
   cambian comportamiento a propósito: son un arreglo, no un refactor.

## Hallazgos

### Nivel 1 — no esperan a nadie y desbloquean a otro

| Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|
| **H1 · Contención:** el detalle de un post publica el correo de cada persona que comenta. | `backend/start/routes.ts:153` → `"select * from comentarios where post_id = ? and estado = 'aprobado' …"`, que se devuelve tal cual en `:171`. | Cualquiera que pida `GET /posts/<slug>` lee el correo de quien comenta (en el post 1, `ana.cortes@ejemplo.es` y `marta.ruiz@corriente.es`). Incumple la regla 6. No lo nota nadie, porque la web no lo pinta, pero el correo viaja en cada carga del artículo. Contención: listar solo las columnas que usa la web (`id, autor_nombre, cuerpo, creado_en, padre_id`). | 1 (`routes.ts:153`) | Sí. | **Contiene a H12:** sin el correo público, suplantar a alguien exige conseguir su correo por otra vía. El arreglo de fondo es H17. |
| **H2 ·** La feature no tiene tests de caracterización: la única prueba solo comprueba el 201 y un campo. | `backend/tests/functional/comentarios.spec.ts:41-42` → `assertStatus(201)` y `assert.equal(…autor_nombre…)`. | Hoy no se rompe nada. Pero un refactor o un arreglo puede cambiar el contrato sin que falle ningún test: códigos, campos, el `200 {ok:false}`, el correo, los recuentos… Nadie lo nota todavía. | 1 (`comentarios.spec.ts`; 0 sitios en producción) | Sí. | **Desbloquea a H4.** `CLAUDE.md` exige caracterizar antes de refactorizar. Hay que cubrir: <br>- crear: feliz, las 4 validaciones, post inexistente y borrador; <br>- editar: feliz, 404, 403 por correo, 403 por no estar pendiente y 400; <br>- borrar: autor, mayúsculas, clave, 403 y 404; <br>- listado: pendiente oculto, `'Aprobado'` y `'ok'` ocultos, sin `autor_email`; <br>- recuentos. |
| **H3 ·** `comentarios.estado` es texto libre, y en la base hay comentarios aprobados guardados como `'Aprobado'` y `'ok'`. | `backend/database/migrations/1769000000005_create_comentarios_table.ts:18` → `table.string('estado')`, sin CHECK. Los datos están en `blog_seeder.ts:510`, `:522` y `:575`. | Tres comentarios aprobados (ids 3, 4 y 9) no salen nunca en el artículo, porque `routes.ts:153` compara exacto. Lo notan quienes los escribieron y el lector, que en el post 6 ve «1 comentarios» en portada y ninguno dentro. | 4 (1 migración nueva que normaliza y añade el CHECK + las 3 filas del seeder, que si no volverían a insertar los valores viejos y chocarían con el CHECK) | **No.** El UPDATE pierde qué filas eran `'Aprobado'` y cuáles `'ok'`, salvo que la migración guarde el valor original. Y quitar un CHECK en SQLite obliga a recrear la tabla. | **Abarata H11:** una vez normalizado, `<> 'pendiente'` y `= 'aprobado'` dan el mismo número con los datos actuales. Requiere una decisión de producto: ¿`'ok'` significa aprobado? |

### Nivel 2 — espera al nivel 1 y desbloquea a otros

| Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|
| **H4 ·** Toda la feature vive dentro de las rutas: el SQL, la validación, la autorización y la forma de la respuesta. | `backend/start/routes.ts:295-393`: los tres handlers (`router.post(…/comentarios`, `router.delete`, `router.patch`), con `db.rawQuery` dentro. | No hay nada visible. Pero cada cambio en comentarios toca un archivo de 418 líneas compartido con las otras cuatro features, y la regla de autorización ya acabó escrita dos veces de forma distinta (H5). Nadie lo nota todavía. | 10 (3 handlers en `routes.ts` en `:295`, `:339` y `:366` + el alias `#repositories/*` en `backend/package.json` + 6 archivos nuevos: modelo, controlador, validador, servicio, repositorio y transformer). El listado de `:152` sigue dentro de `GET /posts/:slug`, que es de la feature posts. | Sí. No lleva migraciones. | **Espera a H2.** Desbloquea a H5, H13, H14, H15, H16, H17 y H18. |

### Nivel 3 — espera al nivel 2 y desbloquea a otros

| Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|
| **H5 ·** Quién puede editar y quién puede borrar se decide con dos reglas escritas a mano que no coinciden. | `routes.ts:352-353` (borrar: compara el correo sin mayúsculas ni espacios, o acepta la clave) frente a `:379` (editar: `comentario.autor_email !== email \|\| comentario.estado !== 'pendiente'`, correo exacto y sin clave). | Quien comentó como `Ana@…` y escribe `ana@…` puede borrar, pero no editar. El moderador puede borrar, pero no editar. Hoy no lo nota nadie, porque la web no ofrece editar ni borrar. | 2 (`:349-355` y `:378-381`) | Sí. | **Espera a H4:** con un servicio, la política vive en un solo sitio. Desbloquea a H6 y H12. Requiere decidir si «solo se edita mientras está pendiente» es una regla buscada. |

### Nivel 4 — espera al nivel 3 y desbloquea a otro

| Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|
| **H6 · Fondo:** la moderación se identifica con una cabecera estática compartida, no con un usuario. | `routes.ts:349-351` → `request.header('x-clave-moderacion')` … `clave === SECRETO_SESION`. | Después de H7 no queda moderación por API hasta que llegue este arreglo. Lo notarían los moderadores, si los hay: ni la web ni ningún script del repo envía esa cabecera. | 3 (la política única de H5 + un grupo de rutas con `middleware.auth()`, que ya existe en `routes.ts:416` + 1 migración para marcar quién modera, porque `users` no tiene ninguna columna de rol) | No del todo: el `down` de la migración borra los roles que se hayan asignado. | **Espera a H5.** Su contención es H7. Desbloquea a H13. |

### Nivel 5 — no desbloquean a nadie

| Qué es | Dónde está | Qué se rompe, y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|
| **H7 · Contención:** la clave que permite borrar cualquier comentario está escrita en el código y publicada en GitHub. | `routes.ts:25` → `const SECRETO_SESION = 'dev-secret-cambiar'`. Solo se usa en `:351`; las sesiones no la usan, a pesar de lo que dice su comentario. | Cualquiera que lea el repo, que está en GitHub (`origin` y `upstream`), puede borrar cualquier comentario con `DELETE /comentarios/<id>` y la cabecera `x-clave-moderacion: dev-secret-cambiar`. Los ids son correlativos. Lo nota quien ve desaparecer su comentario, y nadie sabe quién lo borró, porque no queda registro. Contención: anular la rama de moderador (`esModerador = false`) hasta H6. Moverla a una variable de entorno sin cambiarla no corta nada, y cuesta 4 sitios. | 1 (`routes.ts:351`) | Sí. Pero revertir reabre la clave publicada, y los comentarios ya borrados no vuelven. | A ninguno. El arreglo de fondo es H6. |
| **H8 ·** Al crear un comentario, la respuesta devuelve el último comentario de la tabla, no el que se acaba de insertar. | `routes.ts:329-331` → `select * from comentarios order by id desc limit 1`. | Si dos personas comentan a la vez, una puede recibir la fila de la otra, con su correo y su id: justo lo que pide `DELETE` para borrarla (`:350-353`). Nadie lo nota todavía: necesita concurrencia, y la frecuencia no está medida. | 1 (`:324-331`: usar el id que devuelve el insert) | Sí. | A ninguno. Agrava H12. |
| **H9 ·** Borrar un comentario que tiene respuestas deja esas respuestas huérfanas: siguen en la base y en los recuentos, pero la web ya no las pinta. | `routes.ts:359` → `delete from comentarios where id = ?`, que no hace nada con `padre_id`. La web solo pinta las respuestas que cuelgan de una raíz presente (`Comentarios.jsx:50` y `:63-64`). | Si se borra el comentario 1 del post 1, la respuesta 2 de Marta desaparece de la página, pero la cabecera (`DetallePost.jsx:38`) la sigue contando. Lo notan quien respondió y el lector, que ve un número que no cuadra con lo que hay. | 1 (`:359`). Hay que decidir: borrado en cascada, reasignar las respuestas u ocultarlas. | **No**, si se elige la cascada: las respuestas borradas no vuelven con el revert. | A ninguno. |
| **H10 ·** Al enviar un comentario, la web suma 1 al recuento aunque el comentario queda pendiente y no se muestra. | `frontend/src/componentes/Comentarios.jsx:39` → `setTotal(total + 1)`. | Quien comenta ve «3 comentarios» en el título de la sección, pero solo 2 en la lista y «2 comentarios» en la cabecera (`DetallePost.jsx:71`, que guarda su propio recuento y no se entera). Al recargar vuelve a 2. Lo nota quien acaba de comentar. | 1 (`Comentarios.jsx:39`) | Sí. | A ninguno. |
| **H11 ·** El recuento de portada y de relacionados cuenta todo lo que no es `'pendiente'`, mientras el artículo solo muestra lo `'aprobado'`. | `routes.ts:75-76` → `c.estado <> 'pendiente'`, y `:223`; frente a `= 'aprobado'` en `:153`. | La portada del post 1 dice «4 comentarios» y dentro hay 2. La del post 6 dice 1 y dentro no hay ninguno (medido en la base local). Lo nota el lector. Incumple la regla 5. | 2 (`:75-76` y `:222-224`) | Sí. | **Lo abarata H3:** después de H3 la diferencia desaparece con los datos actuales y este hallazgo queda latente. |
| **H12 · Fondo:** para editar o borrar un comentario basta con saber el correo de quien lo escribió. | `routes.ts:352-353` y `:379`: comparan el `autor_email` que llega en la petición. | Después de H1 sigue bastando un correo conocido o fácil de adivinar. El de la redacción se publica en la respuesta del post (`routes.ts:166`, feature posts), así que cualquiera puede borrar el comentario 2 de `marta.ruiz@corriente.es`. Lo nota quien ve cambiar o desaparecer su comentario. | 3 (1 migración nueva, con un token por comentario o un vínculo a usuario + la creación devolviendo ese token, en `:323-333` + la política única de H5) | No del todo: el `down` de la migración pierde los tokens ya emitidos. | **Espera a H5.** Su contención es H1. Requiere una decisión de producto sobre qué prueba la autoría. |
| **H13 ·** No existe ninguna ruta para aprobar comentarios: todo lo que se crea por la API se queda `'pendiente'` para siempre. | `routes.ts:326` inserta `'pendiente'`, y el único UPDATE sobre la tabla es `:388` → `update comentarios set cuerpo = ?`. | Ningún comentario enviado desde la web llega a mostrarse. Lo nota quien comenta. No sabemos si alguien aprueba a mano en la base: el repo no lo refleja, y el «panel viejo de moderación» (`blog_seeder.ts:509`) ya no existe. | 4, después de H4 (la línea de la ruta + un método en el controlador, otro en el servicio y otro en el repositorio) | Sí. | **Espera a H4 y a H6:** sin identificar al moderador, la ruta quedaría protegida por la clave publicada de H7. |
| **H14 ·** `padre_id` se guarda sin validar: puede apuntar a un id que no existe, a un comentario de otro post o a otra respuesta. | `routes.ts:308` → `request.input('padre_id', null)`, que se inserta tal cual en `:326`. | La respuesta se guarda pero nunca se pinta, porque la web solo pinta dos niveles y con el padre en el mismo post (`Comentarios.jsx:50` y `:63`), y aun así suma en `num_comentarios`. Lo nota quien responde por la API. Desde la web, nadie: el formulario no envía `padre_id` (`Comentarios.jsx:33`). | 2 (una regla en el validador + comprobar en el servicio que el padre existe y es del mismo post) | Sí. Las filas ya guardadas no se corrigen. | **Espera a H4.** |
| **H15 ·** El correo se valida solo comprobando que lleva una `@`. | `routes.ts:316` → `String(email).indexOf('@') === -1`. | Se aceptan `@` o `a@`, y quedan guardados. Como el correo es lo que autoriza a editar o borrar (H12), quien lo escribió mal se queda sin poder hacerlo. Nadie lo nota todavía. | 1 (la regla `email()` del validador) | Sí. | **Espera a H4.** |
| **H16 ·** Editar no exige la longitud mínima de 5 caracteres que sí exige crear. | `routes.ts:384` → `if (!cuerpo)`, frente a `:319` → `String(cuerpo).length < 5`. | Un comentario se puede dejar en `x` o en espacios en blanco (`'   '` pasa). Nadie lo nota todavía: la web no permite editar. | 1 (`:384-386`, que pasa a usar el mismo validador) | Sí. | **Espera a H4.** |
| **H17 · Fondo:** ninguna respuesta de comentarios pasa por un transformer: se devuelven filas crudas. | `routes.ts:333` → `{ comentario: nuevos[0] }`, `:392` → `{ comentario: actualizados[0] }`, y `:171`. | Después de H1 y H8, crear y editar devuelven a la propia persona su correo, `estado` y `post_id`: campos internos que el contrato no pide. Y cualquier columna nueva sale sola a la API. Nadie lo nota todavía. | 4 (transformer nuevo + 3 puntos de respuesta: `:171`, `:333` y `:392`) | Sí. | **Espera a H4.** Su contención es H1. |
| **H18 ·** Los errores de validación al crear salen con cuatro formas y códigos distintos, uno de ellos un 200. | `routes.ts:311` → `400 {error}`, `:314` → `422 {message}`, `:317` → `400 {errores:[…]}` y `:320` → `200 {ok:false, motivo}`. | Un cliente que mira el código de estado da por creado un comentario demasiado corto. La web lo esquiva leyendo los tres nombres de campo (`Comentarios.jsx:44`). Lo nota quien integra la API. | 4 (validador nuevo + `:310-321` + `:384-386` + `Comentarios.jsx:44`) | Sí, pero es un cambio de contrato: los clientes que se adapten al nuevo se rompen con el revert. | **Espera a H4.** Necesita una decisión explícita, porque sustituye tests de caracterización de H2. |
| **H19 ·** La fecha del comentario se guarda en UTC sin zona horaria, y la web se queda con los 10 primeros caracteres. | `routes.ts:323` → `toISOString().slice(0, 19)`. La web hace `substring(0, 10)` en `Comentarios.jsx:59` y `:68`. | Un comentario escrito a las 00:30 en Madrid sale con la fecha del día anterior. Lo nota quien comenta de madrugada. | 2 (`Comentarios.jsx:59` y `:68`) | Sí. | A ninguno. |

### Cálculo del orden

| H | Nivel | Daño | Radio | Daño ÷ radio |
|---|---|---|---|---|
| H1 | 1 | 5 | 1 | 5,00 |
| H2 | 1 | 1 | 1 | 1,00 |
| H3 | 1 | 3 | 4 | 0,75 |
| H4 | 2 | 1 | 10 | 0,10 |
| H5 | 3 | 2 | 2 | 1,00 |
| H6 | 4 | 2 | 3 | 0,67 |
| H7 | 5 | 5 | 1 | 5,00 |
| H8 | 5 | 4 | 1 | 4,00 |
| H9 | 5 | 3 | 1 | 3,00 (gana el empate con H10 porque no se deshace con un revert) |
| H10 | 5 | 3 | 1 | 3,00 |
| H11 | 5 | 3 | 2 | 1,50 |
| H12 | 5 | 4 | 3 | 1,33 |
| H13 | 5 | 4 | 4 | 1,00 |
| H14 | 5 | 2 | 2 | 1,00 |
| H15 | 5 | 1 | 1 | 1,00 |
| H16 | 5 | 1 | 1 | 1,00 |
| H17 | 5 | 2 | 4 | 0,50 |
| H18 | 5 | 2 | 4 | 0,50 |
| H19 | 5 | 1 | 2 | 0,50 |

El nivel ordena dependencias, no el calendario. Las filas que no esperan a nadie (H1, H2, H3,
H7, H8, H9, H10 y H19) se pueden hacer desde el primer día.

## Propuestas sin medir (fuera de la tabla)

Ninguna de estas entra en la tabla hasta que alguien mida lo que se indica.

| Propuesta | Dónde | Qué hay que medir para que entre en la tabla |
|---|---|---|
| Añadir un índice y una clave foránea a `comentarios.post_id` | `1769000000005_create_comentarios_table.ts:6` (el propio comentario de la migración lo reconoce) | `EXPLAIN QUERY PLAN` y el p95 de `GET /posts` y `GET /posts/:slug` con un volumen realista de comentarios. |
| Que el artículo pida los datos una sola vez: hoy hace dos peticiones al mismo `GET /posts/:slug` | `DetallePost.jsx:34` y `Comentarios.jsx:19` | El tiempo y los bytes de la segunda petición en una red real, y si se nota en la carga del artículo. |
| Contar los comentarios de relacionados en una sola consulta: hoy hace una por post | `routes.ts:220-225` | El p95 de `GET /posts/:slug/relacionados`. Hoy `MAX_RELACIONADOS = 2`, así que son 2 consultas. |
| Paginar los comentarios del artículo | `routes.ts:152-155`, que los trae todos | La distribución de comentarios por post. Hoy el máximo es 5. |
| Antispam o límite de envíos en `POST /posts/:slug/comentarios` | `routes.ts:295` | La tasa de comentarios basura recibidos. Hoy no hay datos, porque todo se queda pendiente (H13). |
| Longitud máxima para el cuerpo y el nombre: hoy solo los acota el límite del parser JSON | `routes.ts:319` (solo hay mínimo) | El tamaño real de los cuerpos y nombres recibidos. |

## Revisado y sin hallazgo

- **XSS en comentarios:** la web los pinta como texto (`Comentarios.jsx:61` y `:70`), y React lo escapa.
- **Inyección SQL:** todas las consultas de comentarios están parametrizadas (`routes.ts:153`, `:325`, `:340`, `:359`, `:367` y `:388`).
- **Comentar en un borrador o en un post inexistente:** devuelve 404 (`routes.ts:296-302`).

## Vistos, pero de otra feature (no se auditan aquí)

- `GET /posts/:slug` publica el correo del autor del post (`routes.ts:166`) y `GET /posts` hace `select *` sobre `posts` (`:74-76`). Esto es de la feature posts, pero alimenta a H12.
- La protección CSRF está desactivada (`backend/config/shield.ts:33`). Es configuración global.


# Parte B: La tabla

1. Cuántos hallazgos devolvió el agente, y cuántos comprobaste tú abriendo el archivo. Los dos números, tal cual salieron. No los redondees ni los expliques.

El agente devolvio 19 hallazgos. Comprobé 3 (por falta de tiempo)

2. El hallazgo del que no supiste decidir la prioridad, y en una frase, por qué. Lo que interesa es qué dos cosas se contradecían: lo grave contra lo que bloquea, lo urgente contra lo caro, lo que te pedían contra lo que veías.

No tuve tiempo a corregir prioridades. Pude comprobar que la precendencia era el factor principal en la ordenacion por parte del agente, y los argumnentos encajaban, pero no tuve tiempo a revisar la influencia de daño y radio para ordenar cada uno de los subgrupos

3. Algo que el agente afirmó y no pudiste verificar. Ojo al matiz: no es lo que dijo mal. Es lo que no pudiste comprobar ni a favor ni en contra, y te tocó dejar en el aire.

La comprobacion de los hallazgos 4 a 19, y la priorizacion por daño/radio dentro de cada grupo con mismo nivel de precedencia

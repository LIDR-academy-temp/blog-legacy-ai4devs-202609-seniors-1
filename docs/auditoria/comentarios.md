# Auditoría de la feature de comentarios

Alcance: rutas de la API, tabla `comentarios` y lo que la web enseña de ella.
Fecha: 2026-10-21 · Rama: `auditoria-shernandez` · Solo lectura: no se ha cambiado código.

## Cómo leer las tablas

Una fila por hallazgo. Las seis casillas son las que se pidieron.

- **Dónde** da `archivo:línea`. Rutas relativas a la raíz del repo. `routes.ts` es `backend/start/routes.ts`.
- **Sitios** son los puntos de código distintos que el arreglo toca. Los archivos nuevos que exige `CLAUDE.md` (controlador, validador, servicio, repositorio, transformer) no se cuentan y se indican aparte.
- **¿Se deshace con un revert?** se refiere al arreglo, no al defecto: si el arreglo sale mal, ¿basta con revertir el commit?
- **Prioridad:** P1 = privacidad, seguridad, o prerrequisito de todo lo demás. P2 = el lector ve algo falso o roto. P3 = robustez y deuda.

## Qué se comprobó y cómo

- Todo lo de `routes.ts`, la migración, la semilla y los componentes se leyó línea a línea.
- La base local (`backend/tmp/db.sqlite3`) está migrada pero **sin semilla**. Los números de C-05 y C-06 salen de la semilla (`blog_seeder.ts`) y de las consultas, no de una ejecución.
- El DDL real de la tabla se consultó en SQLite: sin índices y sin claves foráneas (confirma C-18).
- La API no estaba levantada. Ninguna ruta se ha ejecutado, así que los hallazgos de comportamiento son de lectura de código.

---

## P1 — Privacidad, seguridad y prerrequisitos

| ID | Qué es | Dónde | Qué se rompe y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| **C-01** | La API devuelve la fila entera del comentario, con el correo de quien comenta, a cualquier visitante. | `routes.ts:152-155` (`select *`) y `:171` (se devuelve). Mismo patrón en `:329-333` (POST) y `:389-392` (PATCH). La web no lo lee: `autor_email` solo aparece en `Comentarios.jsx:33`, al enviar. | Incumple la regla 6 de `CLAUDE.md`. Los correos de lectores (Ana Cortés, Rubén Lima, Silvia Peña…) salen en `GET /posts/:slug` para quien tenga la URL, incluido un scraper. Lo notaría un lector cuando le llegue spam, o un auditor de protección de datos. | 3 en la API, 0 en la web. Más un transformer nuevo. | Sí. Los correos ya servidos no se recogen, pero el cambio se revierte limpio. | Mitiga C-02 (sin correo público, borrar «como autor» deja de ser trivial) pero no lo cierra. |
| **C-02** | `DELETE` y `PATCH` dan por «autor» a quien envíe el correo del comentario, y ese correo no es un secreto. | `routes.ts:350-353` y `:355` (DELETE). `:378-381` (PATCH). | Con el correo que C-01 publica, cualquiera borra el comentario de otra persona o edita uno suyo pendiente, sin credencial. Lo notaría el lector al que le desaparece el comentario. Hoy casi nadie: la web no llama a `DELETE` ni a `PATCH` (el único `method:` del front es el POST de `Comentarios.jsx:31`), pero la API está abierta. | 2 rutas. | Sí mientras el arreglo sea solo de código. No si se opta por ligar el comentario a un usuario autenticado: eso añade una columna y datos. | Bloquea C-13: unificar la regla antes de decidir cómo se prueba la autoría unificaría la regla rota. |
| **C-03** | La clave de moderación está escrita en el código fuente. | `routes.ts:25` (`SECRETO_SESION = 'dev-secret-cambiar'`) y `:351` (`clave === SECRETO_SESION`). El `TODO(2024-11)` de `:22-24` sigue sin atender. | Quien lea el repo puede borrar cualquier comentario en cualquier despliegue que conserve ese valor. Nadie lo nota hoy: ningún cliente envía `x-clave-moderacion` (no aparece en `frontend/src`). | 3: las dos líneas de `routes.ts` y `backend/.env.example`, que no tiene la variable. | **No.** El valor ya está en el historial (`e0e0943`, initial import): revertir el código no lo recupera como secreto, hay que rotarlo. Y la clave nueva vive en el entorno, que ningún commit revierte. | Bloquea C-09: no conviene construir la ruta de aprobación sobre una clave embebida. |
| **C-04** | Solo hay una prueba para toda la feature, y cubre solo el camino feliz del POST. | `backend/tests/functional/comentarios.spec.ts:15`. Hay un único `test(` en el repo y no existe `tests/unit/`. | Hoy nada se rompe. El efecto es que cualquier arreglo de esta lista puede cambiar comportamiento sin que nada falle. Lo notaría quien refactorice, tarde y en producción. | 4 comportamientos sin prueba: lectura filtrada en el detalle (`routes.ts:152`), validación del POST (`:310-321`), DELETE (`:339-361`) y PATCH (`:366-393`). 1 archivo de test que ampliar. | Sí. Solo añade archivos de test. | **Bloquea a todos los demás**, por el proceso de `CLAUDE.md`: ningún arreglo entra sin su test de caracterización en verde antes. |

## P2 — El lector ve algo falso o roto

| ID | Qué es | Dónde | Qué se rompe y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| **C-05** | `estado` es texto libre y el detalle del post solo muestra `'aprobado'` en minúsculas. | `routes.ts:153` (`estado = 'aprobado'`). Origen: `1769000000005_create_comentarios_table.ts:18` (sin CHECK ni enum). Filas afectadas en la semilla: `blog_seeder.ts:510` (`'Aprobado'`), `:522` (`'ok'`), `:575` (`'Aprobado'`). | Comentarios que alguien aprobó no se ven. En la semilla son 3 de los 7 comentarios aprobados: los de Rubén Lima y Silvia Peña (post 1) y el de Paula Merino (post 6). Lo notan esos lectores, que no ven su comentario, y el post 6, que parece sin conversación. Filas reales en producción: no contadas. | 4: 1 migración nueva (normalizar + CHECK) y las 3 líneas de la semilla, que fallarían contra el CHECK. | **No.** Normalizar `'ok'` y `'Aprobado'` a `'aprobado'` pierde el valor original y el `down()` no sabe cuál era cuál. El CHECK además rechazaría al panel antiguo que escribe en esta tabla (`blog_seeder.ts:509`). Revertir el código no restaura las filas. | Desbloquea C-06. |
| **C-06** | Listado y relacionados cuentan los comentarios con una regla distinta de la del detalle. | `routes.ts:75-76` y `:223` (`estado <> 'pendiente'`) frente a `:153` (`estado = 'aprobado'`). La web solo lo imprime: `ListaPosts.jsx:57`, `PostsRelacionados.jsx:32`. | Incumple la regla 5. Según la semilla, la tarjeta del post 1 dice «4 comentarios» y el detalle enseña 2. La del post 6 dice «1» y el detalle enseña 0. Lo nota cualquier lector que entre desde el listado. | 2 consultas (`:75-76`, `:223`). `:153` ya es la regla correcta. 0 en la web. | Sí. | Lo desbloquea C-05: arreglado solo, el número coincidiría con el detalle, pero los 3 comentarios seguirían ocultos. |
| **C-07** | La web guarda dos contadores de comentarios que dejan de coincidir al comentar. | `DetallePost.jsx:30`, `:38` y `:71` (cabecera). `Comentarios.jsx:11`, `:23`, `:39` (`setTotal(total + 1)`) y `:54` (título de la sección). | Al enviar un comentario, que queda pendiente y no se ve, el título de la sección sube a «N+1 comentarios» y la cabecera sigue en «N». Los dos números son falsos. Lo nota el lector que acaba de comentar, al instante. | 2 componentes, 7 líneas. | Sí. | Lo desbloquea C-17: con una sola fuente del dato los dos contadores salen del mismo array. |
| **C-08** | El POST responde de cuatro formas distintas a una entrada inválida, y la web no entiende una de ellas. | `routes.ts:311` (400 `{error}`), `:314` (422 `{message}`), `:317` (400 `{errores:[…]}`), `:320` (**200** `{ok:false,motivo}`). La web lee `d.error \|\| d.message \|\| d.motivo` en `Comentarios.jsx:44` y no lee `errores`. | Un correo sin `@` devuelve `{errores:[…]}` y el lector ve «No se ha podido enviar.», sin saber por qué. Un comentario corto devuelve 200 con `ok:false`, y cualquier cliente que mire solo el código lo toma por éxito. Lo notan los lectores que se equivocan y quien integre la API. | 5: 4 respuestas en `routes.ts` y 1 línea en la web. | Sí. Pero cambia códigos HTTP de un contrato que `CLAUDE.md` manda caracterizar primero. | A ninguno. |
| **C-09** | Dentro del repo no hay ninguna ruta para aprobar o rechazar un comentario. | `routes.ts:325-326` inserta siempre `'pendiente'`. La única `update` sobre comentarios es `:388` y toca solo `cuerpo`. No hay `update … estado` en todo el archivo. | Todo comentario enviado desde la web queda pendiente para siempre, salvo que alguien toque la base a mano o use el panel antiguo externo (`blog_seeder.ts:509`). El lector ve «pendiente de moderación» (`Comentarios.jsx:38`) y su comentario no aparece. Lo notan los lectores que comentan, y quien modera, que no tiene herramienta aquí. | 1 existente (registrar la ruta en `routes.ts`) más las capas nuevas. 0 en la web. | Sí. Es una ruta nueva. | Lo bloquea C-03. Conviene hacerlo después de C-05, para que escriba el `estado` canónico. |

## P3 — Robustez y deuda

| ID | Qué es | Dónde | Qué se rompe y quién lo nota | Sitios | ¿Revert? | ¿Bloquea / desbloquea? |
|---|---|---|---|---|---|---|
| **C-10** | El POST guarda cualquier `padre_id` sin comprobarlo. | `routes.ts:308` (se lee) y `:326` (se inserta tal cual). | Admite un padre inexistente, de otro post o no numérico. Esa respuesta no se muestra nunca: la web solo busca respuestas bajo comentarios raíz visibles (`Comentarios.jsx:50`, `:63-64`). Nadie todavía por la web, que no envía `padre_id` (`Comentarios.jsx:33`). Sí lo nota quien llame a la API directamente. | 1. | Sí. | A ninguno. |
| **C-11** | `DELETE` no tiene en cuenta las respuestas del comentario borrado. | `routes.ts:359` (`delete … where id = ?`). `padre_id` sin clave foránea: `1769000000005_create_comentarios_table.ts:20`. | Las respuestas quedan huérfanas: la web no las muestra (`Comentarios.jsx:63-64`) pero el recuento de C-06 las sigue contando. Ejemplo en la semilla: el comentario 2 es hijo del 1. Nadie todavía: la web no llama a `DELETE`. | 1 (más la migración de C-18 si se añade FK). | Sí en código. No si se usa `ON DELETE CASCADE`: los hijos borrados por cascada no vuelven. | Lo desbloquea C-18 (FK con cascade). |
| **C-12** | Tras insertar, el POST devuelve «el último comentario de la tabla», no el que acaba de insertar. | `routes.ts:329-331` (`select * from comentarios order by id desc limit 1`, sin `where`). | Con dos envíos casi simultáneos a posts distintos, uno recibe en su 201 el comentario del otro, con su correo y su estado. Mismo tipo de fuga que C-01. Nadie todavía: hace falta concurrencia real. | 1. | Sí. | A ninguno. |
| **C-13** | `PATCH` y `DELETE` aplican reglas de autoría distintas. | `routes.ts:351-353` (DELETE: correo sin distinguir mayúsculas, con `trim`, y acepta la clave de moderación) frente a `:379` (PATCH: comparación exacta, sin clave, y exige `pendiente`). El propio código lo reconoce en `:375-377`. | Quien escribió `Ana@x.es` y edita con `ana@x.es` recibe 403, pero con esa misma entrada sí puede borrar. El moderador puede borrar y no puede editar. Nadie todavía: la web no usa ninguna de las dos rutas. | 2 (las mismas rutas que C-02). | Sí. | Lo bloquea C-02. |
| **C-14** | `PATCH` no exige el largo mínimo que sí exige el POST. | `routes.ts:384` (`!cuerpo`) frente a `:319` (`length < 5`). | Un comentario válido de 40 caracteres se puede editar a «x». Lo notaría el moderador, tarde. Nadie todavía: la web no edita. | 1. | Sí. | A ninguno. |
| **C-15** | El POST no acota la longitud de nombre, correo ni cuerpo. | `routes.ts:310`, `:313`, `:319` (solo comprueban que existan o el mínimo). `:383` (PATCH, el cuerpo). La migración usa `string` (255) para nombre y correo y `text` para el cuerpo: `…create_comentarios_table.ts:15-17`. | Sin tope de campo, el único límite es el del bodyparser. Un cuerpo enorme se guarda y viaja a todos los lectores en el detalle. Nadie todavía. | 4 validaciones en 2 rutas. Más un validador nuevo. | Sí. | A ninguno. |
| **C-16** | El POST no tiene ninguna limitación de frecuencia. | `routes.ts:295`: la ruta no lleva `.use(…)`, a diferencia de `:416` (`middleware.auth()`). Sin `throttle` ni `limiter` en `start/` ni `config/`. | Cualquiera puede llenar la cola de pendientes. La semilla ya trae un caso de spam (`blog_seeder.ts:550-553`) y por C-09 no hay herramienta para vaciarla. Nadie todavía. | 2: la ruta y el registro del middleware en `start/kernel.ts`. | Sí. | A ninguno. |
| **C-17** | La web pide dos veces el mismo post para pintar una pantalla. | `DetallePost.jsx:34` y `Comentarios.jsx:19`. El segundo fetch solo extrae `comentarios`, que el primero ya trae. | Cada vista del detalle ejecuta dos veces las 7 consultas de `routes.ts:110-164`, 14 en total. Nadie todavía: con 10 posts no se nota. | 2 componentes. | Sí. | Desbloquea C-07. |
| **C-18** | La tabla no tiene índice ni clave foránea sobre `post_id` ni `padre_id`. | `1769000000005_create_comentarios_table.ts:14` y `:20`. Confirmado en el DDL de SQLite: ningún índice sobre `comentarios`. | Cada recuento hace `where post_id = ?` sin índice, y el listado lo repite como subconsulta por cada fila (`routes.ts:75`). Nadie todavía: la tabla tiene 9 filas en la semilla. | 1 migración nueva. | Índice: sí (`dropIndex` en `down()`). FK: no con un revert simple, porque SQLite obliga a recrear la tabla. | Desbloquea C-10 y C-11 si se añade la FK. |

---

## Orden de ataque sugerido

1. **C-04** primero: sin caracterización no se toca nada, y lo exige `CLAUDE.md`.
2. **C-01** y **C-03**: son lo único que hoy expone datos o secretos de verdad. C-03 es el que no se puede revertir, así que decide cuándo rotar la clave antes de empezar.
3. **C-05 → C-06 → C-09**, en ese orden, porque cada uno depende del anterior.
4. **C-02 → C-13** juntos, una vez decidido cómo se prueba la autoría.
5. **C-17 → C-07**, y el resto de P3 cuando convenga.

## Revisado y descartado

- **XSS en comentarios:** no. La web pinta nombre y cuerpo como texto de React (`Comentarios.jsx:59-61`), que lo escapa. (El `dangerouslySetInnerHTML` del cuerpo del *post* es otra feature.)
- **Inyección SQL:** no. Todas las consultas de comentarios usan parámetros `?`.
- **Comentar en un borrador:** no. El POST responde 404 si el post no está publicado (`routes.ts:296-302`).
- **`key={index}` en la lista** (`Comentarios.jsx:57`, `:66`): real pero sin efecto, porque la lista no se reordena ni se edita en la web.
- **`creado_en` nullable** (`…create_comentarios_table.ts:19`): la web pintaría «null» (`Comentarios.jsx:59`), pero no hay ninguna fila sin fecha ni ruta que la deje vacía.
- **Cuerpo de cinco espacios:** pasa la validación `length < 5` (`routes.ts:319`). Sin impacto probado.

## Fuera de alcance, visto de paso

No son de esta feature y no se han auditado:

- `GET /autores/:id` devuelve también los posts en borrador (`routes.ts:284-287`, sin filtro de `estado`).
- `GET /posts/:slug` devuelve el correo del autor del post (`routes.ts:119-122`, `select *`).



# Parte B las tres lineas
##Cuántos hallazgos devolvió el agente, y cuántos comprobaste tú abriendo el archivo. Los dos números, tal cual salieron. No los redondees ni los expliques.

El agente devolvio 18 y no pude comprobar ninguno

##El hallazgo del que no supiste decidir la prioridad, y en una frase, por qué. Lo que interesa es qué dos cosas se contradecían: lo grave contra lo que bloquea, lo urgente contra lo caro, lo que te pedían contra lo que veías.

sin criterio para verificar 

##Algo que el agente afirmó y no pudiste verificar. Ojo al matiz: no es lo que dijo mal. Es lo que no pudiste comprobar ni a favor ni en contra, y te tocó dejar en el aire.

no pude comprobar nada
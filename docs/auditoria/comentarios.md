# PARTE A: la auditoría

Alcance: crear, editar, borrar y listar comentarios, en sus tres capas (API, base de datos, web).
Los posts, autores, categorías y etiquetas solo aparecen donde interactúan con los comentarios
(el recuento y el detalle del post).

Rutas citadas respecto a la raíz del repositorio. `routes.ts` = `backend/start/routes.ts`.
Una cita `Archivo:Lxx` es la línea que demuestra el hallazgo.

## 1. Cómo se verificó (y qué no)

- **Lectura completa** de `routes.ts`, la migración `1769000000005_create_comentarios_table.ts`, el
  seeder, `Comentarios.jsx`, `DetallePost.jsx`, `ListaPosts.jsx`, `PostsRelacionados.jsx`, la prueba
  existente, `cors.ts`, `bodyparser.ts` y `handler.ts`.
- **Ejecución** sobre una **copia aislada** del backend (otra carpeta, otro puerto, su propia copia de
  `tmp/db.sqlite3`), con `curl` y un script de Node. El repositorio y su base de datos no se
  tocaron (`git status` limpio al terminar). `node ace test` en la copia: 1 prueba, en verde.
- **No se ejecutó la web en un navegador.** Los hallazgos de frontend (C-18) salen de leer el código.
- **No hay datos de producción.** Todo lo que dependa de volumen, tráfico o despliegue está en
  «Pendiente de medición».
- Cada fila lleva una etiqueta en «Qué es»:
  **[Comprobado]** = ejecutado y observado; **[Comprobado por código]** = el código lo demuestra sin
  ambigüedad pero no se ejecutó la UI; **[Riesgo sustentado]** = el código lo permite, pero no se
  logró reproducir.
- **Cómo se cuenta «Cuántos sitios hay que tocar».** Son las ubicaciones del repositorio que la
  corrección propuesta modifica, contadas con `grep` y lectura. Los archivos nuevos que habría
  que crear (una migración, por ejemplo) cuentan como 1 cada uno. No se estima nada.
- **Cómo se lee «¿Se deshace con un revert?».** Se refiere a la corrección propuesta: si se aplica y
  luego se revierte el commit, ¿el sistema vuelve exactamente al estado anterior?

## 2. Orden de ejecución y criterio

El orden sigue dos reglas, en este orden (mentor): **primero la precedencia, después daño ÷ radio**
(daño observado ÷ número de sitios de la columna 4).

| Nivel | Qué contiene | Por qué va ahí |
|---|---|---|
| 0 | C-01 | Sin red de pruebas, toda reestructuración posterior se hace a ciegas. |
| 1 | C-02, C-03, C-04, C-05, C-06 | Raíces: cada una condiciona a otros hallazgos y ninguna depende de otro, salvo C-06 que espera a C-01. Dentro del nivel, ordenadas por daño ÷ radio. |
| 2 | C-07 … C-16 | Dependen de un hallazgo del nivel 0 o 1. |
| 3 | C-17 | Depende de C-09 (nivel 2) y de C-05 (nivel 1). |
| — | C-18, C-19, C-20 | Independientes: ni condicionan ni dependen. Se pueden hacer en paralelo en cualquier momento; ordenadas por daño ÷ radio. |

**Grafo de dependencias** (A → B: A va antes que B):

```
C-01 → C-06, C-14, C-15, C-16
C-02 → C-07 (limita la explotación), C-16
C-03 → C-15
C-04 → C-08, C-14
C-05 → C-17
C-06 → C-09, C-10, C-11, C-12, C-13
C-08 → C-14
C-09 → C-17
```

**Contención y corrección definitiva** (los hallazgos de alto daño y baja precedencia se dividen):

| Problema | Contención inmediata | Corrección definitiva |
|---|---|---|
| Correo del comentarista expuesto | C-02 | C-16 |
| Clave de moderación en el código | C-03 | C-15 |
| Estados de comentario sin normalizar | C-04 | C-08 (+ C-14) |
| Borrar/editar con un correo público | C-02 (la deja de exponer, no la cierra) | C-07 |

## 3. Hallazgos, en orden de ejecución

| Qué es | Dónde está | Qué se rompe y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
|---|---|---|---|---|---|
| **C-01 [Comprobado]** Solo 1 de las 14 ramas de respuesta de los endpoints de comentarios tiene prueba (el 201 del POST). | `backend/tests/functional/comentarios.spec.ts:L15-43` (única prueba; su cabecera L5-11 lo admite). Ramas sin prueba: `routes.ts:L301,L311,L314,L317,L320` (POST), `L344,L356,L360` (DELETE), `L371,L380,L384,L392` (PATCH), `L152` (lista del detalle). El frontend no tiene script de pruebas (`frontend/package.json`). | Nada hoy. Cada corrección de esta lista se haría sin red: lo notaría quien corrija, como una regresión silenciosa. Nadie todavía. | **13** ramas sin prueba, todas en 1 archivo (`comentarios.spec.ts`). | Sí: añadir pruebas no cambia el comportamiento. | **Condiciona** C-06, C-14, C-15, C-16 (reestructuran rutas; hay que caracterizar antes de tocar). |
| **C-02 [Comprobado]** *(contención)* El detalle público de un post devuelve el `autor_email` de cada comentario (y también `estado` y `post_id`) porque lista con `select *`. | `routes.ts:L152` (`select * from comentarios …`) y `L171` (`comentarios: comentarios`). `README.md:L65` prohíbe exponer el correo. Ejecutado: el JSON devuelve `id,post_id,autor_nombre,autor_email,cuerpo,estado,creado_en,padre_id`. | Cualquier visitante, con `curl` o DevTools, lee el correo de cada comentarista aprobado. Lo nota el comentarista, que no sabe que su correo es público. La web no lo pinta (`Comentarios.jsx:L58-71`), así que no se ve en pantalla. | **1** (`routes.ts:L152`: nombrar las columnas, sin `autor_email`). | Sí. | **Condiciona** C-07 (sin el correo público, el atacante debe adivinarlo). **Desbloquea** C-16. |
| **C-03 [Comprobado]** *(contención)* La clave de moderación está escrita en el código fuente (`dev-secret-cambiar`) y permite borrar cualquier comentario. | `routes.ts:L25` (literal, con `TODO(2024-11)` en L22-24), usada en `L349-351`. Introducida en el commit `e0e0943` (`git log -S`). Ejecutado: `DELETE` con la cabecera `x-clave-moderacion: dev-secret-cambiar` → 200 `{"borrado":true}`. | Quien tenga una copia del repositorio borra cualquier comentario; lo nota el autor del comentario borrado. No hay registro de borrados (`L359`), así que no se puede saber si ya ocurrió. | **1** (`routes.ts:L25`: sustituir el valor y dar por quemado el antiguo). El nuevo valor seguirá en el código hasta C-15. | **No**: el valor antiguo queda en el historial (`e0e0943`) y en cada clon; revertir restauraría el valor comprometido, y la exposición pasada no se deshace. | **Condiciona** C-15 (la definitiva debe mover el valor ya rotado, no el comprometido). |
| **C-04 [Comprobado]** *(contención)* Tres comentarios aprobados con `estado` no canónico (`Aprobado` ×2, `ok`) no salen en el detalle del post pero sí cuentan en el listado. | `backend/database/seeders/blog_seeder.ts:L510` (id 3), `L522` (id 4), `L575` (id 9); el filtro exacto está en `routes.ts:L152` y el recuento en `L75-76`. Ejecutado (post 1): listado `num_comentarios=4`, detalle 2 comentarios. | Los lectores no ven 3 comentarios aprobados (el 9 es el único del post 6; el 3 y el 4 son del post 1); lo notan sus autores y quien compare el número del listado con el del artículo. **Decisión previa**: confirmar que `ok` significa aprobado (el seeder, L521, lo atribuye a un script de importación de 2024). | **6**: 3 filas de `tmp/db.sqlite3` (archivo sin versionar) + 3 entradas del seeder (L510, L522, L575). | **No**: el `UPDATE` vive en la base de datos, no en git; revertir el commit no devuelve los valores antiguos. | **Desbloquea** C-08 (la restricción no puede crearse con filas no canónicas) y C-14 (los recuentos coinciden). |
| **C-05 [Comprobado]** `DELETE` borra en duro un comentario sin tratar sus respuestas: quedan en la base y desaparecen de la web. | `routes.ts:L359` (`delete from comentarios where id = ?`); `Comentarios.jsx:L50` (solo pinta raíces) y `L63-72` (las respuestas solo se pintan bajo su raíz). Ejecutado: tras borrar el comentario 1, la API sigue devolviendo el 2 con `padre_id=1` y la base lo conserva. | El autor de la respuesta: su comentario aprobado deja de verse pero sigue contando en `total` (`Comentarios.jsx:L23`). Casi nadie hoy: la web no tiene botón de borrar (solo usa `POST`, `L31`); solo lo provocan clientes de la API o moderación. | **1** (`routes.ts:L359`). Requiere decidir antes entre cascada y borrado lógico. | Sí para el cambio de código. Los borrados ya hechos no se recuperan (no hay copia). | **Condiciona** C-17 (el `ON DELETE` de la clave foránea sigue esta decisión). |
| **C-06 [Comprobado]** El POST responde los errores de validación con cuatro formas distintas (una con 200) y la web ignora una de ellas. | `routes.ts:L310-321` (L311 `400 {error}`, L314 `422 {message}`, L317 `400 {errores:[…]}`, L320 `200 {ok:false,motivo}`); `Comentarios.jsx:L44` lee `error`, `message` y `motivo`, pero no `errores`. Ejecutado: los cuatro formatos devueltos tal cual. | El lector que escribe un correo sin `@` ve «No se ha podido enviar.» sin saber por qué (su `errores` se descarta). Los clientes de la API reciben cuatro contratos. Un cuerpo corto devuelve 200, que un cliente lee como éxito. | **5**: 4 ramas en `routes.ts` (L310-312, L313-315, L316-318, L319-321) + 1 en `Comentarios.jsx:L44`. Se cambian a la vez (el contrato es compartido). | Sí. | **Condiciona** C-09, C-10, C-11, C-12, C-13 (cada una añade un error que debe usar el formato definido aquí). Antes necesita C-01 (hoy el 200 de L320 es contrato). |
| **C-07 [Comprobado]** La autoría de un comentario se «prueba» con su correo, que es público: cualquiera puede borrar el comentario de otro. | `routes.ts:L350-353` (DELETE: `esAutor` por correo) y `L378-379` (PATCH, mismo criterio); el correo público lo da `L152` (C-02). Ejecutado: `DELETE /comentarios/1` con solo `{"autor_email":"<el del GET>"}` → 200 `{"borrado":true}` sin clave. | El comentarista afectado pierde su comentario (o, si está pendiente, se lo editan); cualquier visitante puede hacerlo. Sin registro (`L359`), nadie lo vería. | **2** (`routes.ts:L350-353` y `L378-379`: los dos sitios que evalúan el correo como prueba). | Sí. | **Condicionado por** C-02 (lo limita, no lo cierra: el correo es adivinable o conocido). Es otro camino al mismo efecto que C-03. |
| **C-08 [Comprobado]** *(definitiva)* `estado` es texto libre: la base de datos admite cualquier valor. | `backend/database/migrations/1769000000005_create_comentarios_table.ts:L18` (`table.string('estado')`; el comentario de L3-7 lo admite). Ejecutado: en la base conviven `aprobado` (4), `Aprobado` (2), `ok` (1) y `pendiente` (3). | Hoy solo los comentarios de C-04. A futuro, cualquier valor nuevo (`APROBADO`, un typo) deja un comentario invisible para los lectores y contado en el listado. | **1** (una migración nueva que reconstruya la tabla con la restricción; SQLite no añade `CHECK` con `ALTER`). | **No**: la migración aplicada permanece en la base; revertir el commit no ejecuta el `down`. | **Bloqueado por** C-04. **Desbloquea** C-14. |
| **C-09 [Comprobado]** El POST acepta cualquier `padre_id`: inexistente, de otro post, de una respuesta o no numérico. | `routes.ts:L308` (lo lee sin validar) y `L326` (lo inserta). Ejecutado: 201 con `padre_id` 9999, con 6 (comentario del post 2), con 2 (una respuesta) y con `"abc"`. La web no los pinta: `Comentarios.jsx:L50,L63-66`; y nunca envía `padre_id` (`L33`). | Un comentario aceptado y contado que ningún lector verá. Lo provocan clientes de la API, no la web (la web no tiene botón de responder): nadie todavía desde la UI. | **1** (`routes.ts:L308`/`L326`: validar antes de insertar). | Sí. | **Bloqueado por** C-06 (formato de error). **Condiciona** C-17 (con clave foránea, estos casos serían 500). |
| **C-10 [Comprobado]** El PATCH no aplica la longitud mínima que sí exige el POST. | `routes.ts:L319` (POST exige ≥5 caracteres) frente a `L383-386` (PATCH solo exige que no esté vacío). Ejecutado: `PATCH` con `cuerpo:"x"` → 200 sobre un comentario pendiente. | Un moderador aprueba un comentario de 1 carácter que el POST habría rechazado; lo vería el lector. Nadie todavía (no hay UI de edición). | **1** (`routes.ts:L383-386`). | Sí. | **Bloqueado por** C-06. |
| **C-11 [Comprobado]** El POST no comprueba el tipo de los campos: nombre o correo como objeto/array dan 500 y un número en `cuerpo` se guarda como `"123456.0"`. | `routes.ts:L305-307` (lee sin tipar), `L310-319` (solo comprueba vacío/longitud), `L324-327`. Ejecutado: `autor_nombre:{"a":1}` → 500 con la sentencia SQL y los valores en `message` (`handler.ts:L9`, `debug = !app.inProduction`); `cuerpo:123456` → 201 con `"123456.0"`. | Clientes de la API malformados reciben 500; en desarrollo el mensaje filtra SQL y datos. La web siempre envía cadenas (`Comentarios.jsx:L33`): nadie todavía desde la UI. | **3** (`routes.ts`: nombre L310, correo L313-316, cuerpo L319). | Sí. | **Bloqueado por** C-06. |
| **C-12 [Comprobado]** Ningún campo del comentario tiene longitud máxima: se aceptó un cuerpo de 900 KB y un nombre de 100 000 caracteres. | `routes.ts:L310` (nombre), `L313-316` (correo), `L319` (cuerpo, solo mínimo), `L383-386` (PATCH). La migración declara `string` (L15-16), que SQLite no hace cumplir. Ejecutado: 201 con 900 KB; 413 con 2 000 KB (límite global del servidor, no de la feature). | Cuando un moderador lo apruebe, todos los lectores descargan ese comentario entero en el detalle (`L152`) y la web lo pinta (`Comentarios.jsx:L61`). Nadie todavía: no hay comentarios así en la base. | **4** (`routes.ts:L310, L313-316, L319, L383-386`). | Sí. | **Bloqueado por** C-06. |
| **C-13 [Comprobado]** El correo solo se valida buscando una `@`: `a@b` se acepta. | `routes.ts:L316` (`indexOf('@') === -1`). Ejecutado: 201 con `autor_email:"a@b"`. | Quedan comentarios con correos que no existen; lo notaría un moderador que intente contactar al autor. Nadie todavía. | **1** (`routes.ts:L316`). | Sí. | **Bloqueado por** C-06. |
| **C-14 [Comprobado]** Dos reglas distintas para «comentario visible»: el recuento usa `estado <> 'pendiente'` y el detalle usa `estado = 'aprobado'`. | `routes.ts:L75-76` (`/posts`) y `L223` (`/posts/:slug/relacionados`) frente a `L152` (detalle). `README.md:L64` dice que el recuento cuenta **solo** los aprobados. Ejecutado (post 1): listado 4, detalle 2. | Hoy lo notan los lectores por los datos de C-04 (el número del listado no coincide con los comentarios visibles). Tras C-04 coinciden y queda como riesgo: cualquier estado nuevo distinto de `pendiente` se contaría sin mostrarse. | **2** (`routes.ts:L75-76` y `L223`). | Sí. | **Bloqueado por** C-04 y C-08; antes necesita C-01 (hoy el recuento es contrato). |
| **C-15 [Comprobado]** *(definitiva)* La clave de moderación sigue en el código y se compara con `===`. | `routes.ts:L25` y `L351` (las dos únicas apariciones de `SECRETO_SESION`, `grep`); `backend/start/env.ts` y `backend/.env.example` no declaran ninguna variable para ella. | Mientras no se mueva a la configuración, quien lea el repositorio conoce la clave vigente (C-03 solo cambia su valor). | **4**: `routes.ts:L25`, `routes.ts:L351`, `backend/start/env.ts` (variable nueva), `backend/.env.example` (variable nueva). | **No**: revertir restaura el literal en el código, y el valor antiguo ya está en el historial (C-03). | **Bloqueado por** C-03; antes necesita C-01 (se toca la ruta). |
| **C-16 [Comprobado]** *(definitiva)* Las tres respuestas que devuelven un comentario lo hacen con la fila cruda de la tabla, sin una forma de salida propia. | `routes.ts:L152` (lista), `L329-333` (POST), `L389-392` (PATCH): todas hacen `select *` y la devuelven tal cual. Ejecutado: el POST devuelve `autor_email`, `estado` y `post_id`. | Tras C-02 no se filtra nada nuevo; lo que sigue ocurriendo es que `estado` y `post_id` salen en respuestas públicas y que cualquier columna nueva de la tabla saldría sin aviso. Nadie todavía. | **3** (`routes.ts:L152`, `L329-333`, `L389-392`). | Sí. | **Bloqueado por** C-02; antes necesita C-01. |
| **C-17 [Comprobado]** `comentarios.post_id` y `comentarios.padre_id` no tienen clave foránea. | Migración `…create_comentarios_table.ts:L14` (`post_id`) y `L20` (`padre_id`); el comentario de L3-7 lo admite. Ejecutado: 0 comentarios con `post_id` inexistente y 0 con `padre_id` inexistente en `tmp/db.sqlite3`; `PRAGMA foreign_keys` = 1 con `better-sqlite3`, así que la restricción se aplicaría. | Nadie todavía: la base local está íntegra. Es la defensa de fondo de C-05 y C-09. | **2** (`post_id` y `padre_id`, en una migración nueva que reconstruya la tabla). | **No**: la migración aplicada permanece; revertir el commit no ejecuta el `down`. | **Bloqueado por** C-05 y C-09 (sin ellos, las filas huérfanas pasarían a ser errores 500). |
| **C-18 [Comprobado por código]** Tras enviar un comentario, el título de la sección suma 1 al total aunque el comentario queda pendiente y no se muestra. | `Comentarios.jsx:L38-39` (`setTotal(total + 1)` tras responder «pendiente de moderación»), `L54` (`{total} comentarios`) y `L50-74` (la lista no cambia). El código de `L8-10` lo admite. | Todo lector que comenta ve «N+1 comentarios» sobre N visibles, mientras la cabecera del artículo sigue en N (`DetallePost.jsx:L71`). Es el camino más frecuente. | **1** (`Comentarios.jsx:L39`). | Sí. | A ninguno. |
| **C-19 [Riesgo sustentado]** Tras insertar, el POST devuelve «el último comentario de la tabla» y no el que acaba de insertar. | `routes.ts:L329-331` (`select * from comentarios order by id desc limit 1`) y `L333`. No reproducido: 40 POST concurrentes devolvieron cada uno su propio comentario. | Si dos peticiones se intercalan entre el `insert` y el `select`, una recibe el comentario de la otra, con su correo y nombre. Nadie todavía (no reproducido). | **1** (`routes.ts:L329-331`). | Sí. | A ninguno. |
| **C-20 [Comprobado]** El PATCH compara el correo distinguiendo mayúsculas y el DELETE no. | `routes.ts:L379` (`!==` directo) frente a `L352-353` (`toLowerCase().trim()`). El comentario de L375-377 lo admite. Ejecutado: `PATCH` con `Tomas.Iglesias@…` sobre el comentario de `tomas.iglesias@…` → 403; `DELETE` con `PROMO@…` sobre `promo@…` → 200. | El autor que teclea su correo con otras mayúsculas puede borrar su comentario pero no editarlo. Nadie todavía desde la web (no tiene UI de edición). | **1** (`routes.ts:L379`). | Sí. | A ninguno. |

## 4. Pendiente de medición

Propuestas que **no** entran en la tabla porque falta evidencia para fijar su efecto, alcance o
existencia. Para cada una: qué hay que medir o verificar antes de incorporarla.

| ID | Propuesta | Qué se sabe | Qué medir o verificar |
|---|---|---|---|
| P-01 | Ruta para aprobar comentarios | Las únicas escrituras sobre `comentarios` en `routes.ts` son el `insert` (L324-327), el `delete` (L359) y el `update` del cuerpo (L388). Ninguna cambia `estado`. El seeder (L509) menciona un «panel viejo de moderación». En la base local hay 3 de 10 comentarios `pendiente`. | Confirmar con quien opera el blog cómo pasa hoy un comentario de `pendiente` a `aprobado`. Si no existe ningún mecanismo fuera de este repositorio, pasa a la tabla: todo comentario nuevo quedaría invisible para siempre. |
| P-02 | Editar, borrar y responder desde la web | `Comentarios.jsx` solo usa `POST` (L31) y no envía `padre_id` (L33), pero pinta respuestas (L63-72). `README.md:L105` define la feature como «crear, editar, borrar y listar». | Decidir si la web debe ofrecerlo. Si sí, cuenta como funcionalidad pendiente; si no, las rutas `PATCH`/`DELETE` solo las usan clientes externos y C-07 y C-20 son de ese público. |
| P-03 | Permitir editar un comentario ya aprobado | El PATCH solo admite comentarios `pendiente` (`routes.ts:L379`); el comentario de L375-377 dice que «salió distinta» al DELETE, sin decir si es intencional. | Pregunta de producto: si el autor o el moderador deben poder corregir tras la aprobación. |
| P-04 | Índice en `comentarios.post_id` | La migración dice que no existe (L4-6, L14). La base local tiene 10 filas: ninguna consulta lo evidencia. | `EXPLAIN QUERY PLAN` de `routes.ts:L152` y `L75-76`, y su latencia con el volumen real (o uno sembrado) de comentarios. |
| P-05 | Límite de envíos / antispam | No hay limitación de ritmo en `start/kernel.ts` ni dependencia para ello en `package.json`. El seeder incluye un comentario spam (comentario id 7, `blog_seeder.ts:L548`). | Tasa de envíos y proporción de spam en el blog real; sin eso no se puede dimensionar la medida. |
| P-06 | Evitar el doble envío | `Comentarios.jsx:L28-48` no desactiva el botón (L86) y el servidor no deduplica. En la base local, 0 comentarios duplicados (mismo post, correo y cuerpo). | Contar duplicados reales en datos de producción antes de decidir. |
| P-07 | Fecha mostrada del comentario | `routes.ts:L323` guarda la fecha en UTC, sin marca de zona; `Comentarios.jsx:L59` muestra los 10 primeros caracteres. | Qué zona esperan los lectores y si los datos sembrados están también en UTC; solo entonces se sabe si el día mostrado se desplaza. |
| P-08 | URL base de la API y CORS | `Comentarios.jsx:L4` fija `http://localhost:3401`; `cors.ts` solo permite cualquier origen en desarrollo (`origin: app.inDev ? true : []`). | Dónde se va a desplegar la web. El README solo documenta ejecución local; si hay despliegue, la feature no funcionaría fuera de `localhost`. |
| P-09 | Una sola descarga del post para pintar los comentarios | `DetallePost.jsx:L34` y `Comentarios.jsx:L19` piden el mismo `GET /posts/:slug` en cada visita. | Tamaño de la respuesta y latencia con comentarios reales (ver C-12). |
| P-10 | Contenido del error 500 en producción | C-11 filtra SQL en `message` bajo `debug = !app.inProduction` (`handler.ts:L9`). | Repetir la petición de C-11 con `NODE_ENV=production`; no se ejecutó. |

## 5. Revisado y descartado

Se examinó y **no** es un defecto en el código actual:

- **Inyección SQL en comentarios.** Todas las consultas de la feature usan parámetros
  (`routes.ts:L296-298, L324-327, L340-342, L359, L367-369, L388-391`). La única concatenación
  (`L75-76`) une cadenas constantes.
- **XSS al pintar comentarios.** `Comentarios.jsx` pinta `nombre` y `cuerpo` como texto de React
  (`L59-61, L68-70`), sin `dangerouslySetInnerHTML`. El único uso de esa API está en
  `DetallePost.jsx:L92` y es el cuerpo del post, fuera del alcance.
- **Comentario pendiente visible en el detalle** (regla 2 del README): `routes.ts:L152` filtra por
  `aprobado`; ejecutado, el comentario 5 (`pendiente`) no aparece.
- **Comentar en un post en borrador:** el POST busca el post con `estado = 'publicado'`
  (`routes.ts:L296-302`) y devuelve 404 si no lo es. Leído, no ejecutado.

## 6. Limitaciones

- La web no se ejecutó en un navegador (ver C-18).
- La base de datos local (`backend/tmp/db.sqlite3`) no está versionada; los conteos de filas (C-04,
  C-17) describen esa copia, no un entorno compartido.
- C-19 no se reprodujo en 40 envíos concurrentes; sigue siendo un riesgo sustentado por el código.
- El comportamiento de producción (errores, CORS, volumen) no se pudo medir: ver sección 4.

# PARTE B: las tres líneas

## 1. Cuántos hallazgos devolvió el agente, y cuántos comprobaste tú abriendo el archivo. 
El agente devolió 20 hallazgos y solo pude comprobar 5.

## 2. El hallazgo del que no supiste decidir la prioridad, y en una frase, por qué.  
Para poder decidir prioridad, es necesario revisar todos los hallazgos. Si bien la información que generó el agente en las columnas de precedencia, daño y radio es pausible, no pude verificar si la prioridad era correcta porque no pude completar la validación de los 20 hallazgos en el tiempo dado.

## 3. Algo que el agente afirmó y no pudiste verificar.   
Hallazgo C-05 DELETE borra en duro un comentario sin tratar sus respuestas: quedan en la base y desaparecen de la web. Al no haber FK recursivo en la tabla **comentarios**, intenté comprobar si esa era la intención/lógica en el código back y front, pero no dio el tiempo.
## Parte A: la auditoría

# Auditoría — feature de comentarios

Alcance: solo comentarios (API, tabla `comentarios`, web). Auditoría de solo lectura: BD consultada con `SELECT` en modo `readonly`, API solo con `GET`.

**Orden:** primero la precedencia (lo que abarata o desbloquea otro hallazgo va antes); dentro de cada nivel, daño ÷ radio, de mayor a menor. El prefijo de cada fila es `[nivel · daño/radio]`.

- **Niveles:** `C` = contención que corta el daño hoy; `N0` = precede a todo (CLAUDE.md obliga a caracterizar antes de corregir); `N1` = abarata o desbloquea otro hallazgo; `N2` = no desbloquea ninguno.
- **Daño (1–5):** 5 = pérdida irreversible o datos personales expuestos hoy a terceros; 4 = lo mismo, pero hace falta acceso al repo; 3 = contenido incorrecto que ve el lector hoy; 2 = error visible para quien comenta o para el equipo, o fuga latente; 1 = latente, nadie lo nota todavía.
- **Radio:** la columna «Sitios a tocar».
- **Partidos en dos filas:** H2 y H3 tienen el daño más alto y la precedencia más baja. Cada uno lleva una fila `a` (contención de hoy) y una `b` (arreglo de fondo, en su turno).

| Qué es | Dónde está | Qué se rompe y quién lo nota | Sitios a tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea otro? |
|---|---|---|---|---|---|
| **[C · 5/1] H2a** · Contención de H2: retirar la ruta `DELETE /comentarios/:id` hasta que haya una autorización real | `backend/start/routes.ts:339-361`. La web no la usa: buscar `DELETE` en `frontend/src` da 0 resultados | Nadie desde la web. El uso por clientes externos de la API está sin medir (ver «Propuestas sin medir») | 1 | Sí: vuelve la ruta | Corta hoy H2 y H3; no sustituye a H2b ni a H3b |
| **[C · 4/0] H3a** · Contención de H3: la misma retirada de H2a, porque la clave solo se usa en el DELETE | Único uso: `routes.ts:351` (búsqueda de `SECRETO_SESION`: líneas 25 y 351) | Nadie desde la web | 0 adicionales (va en H2a) | Sí | Depende de H2a |
| **[N0 · 2/1] H20** · El único test afirma un solo campo: no comprueba que falte `autor_email` ni el `estado` | `backend/tests/functional/comentarios.spec.ts:41-42` | Ningún test cubre DELETE, PATCH ni los errores. Nadie todavía | 1 fichero | Sí | Bloquea H1–H19 (por CLAUDE.md, se caracteriza antes de corregir) |
| **[N1 · 5/1] H1** · El detalle de un post devuelve el `autor_email` de cada comentario | `routes.ts:153` (`select *`) y `:171`. Comprobado en vivo: `GET /posts/diez-tendencias…` devuelve `ana.cortes@ejemplo.es` | Se incumple la regla 6. Lo nota cualquiera que llame a la API; la web no lo pinta | 1 (`routes.ts:153`) | No: los correos ya publicados pueden estar cacheados o recopilados | Abarata H2 (cierra la vía fácil para conseguir el correo) |
| **[N1 · 1/1] H12** · `padre_id` se guarda sin validar | `routes.ts:308`, `:326` | Admite un padre que no existe, que es de otro post o que está pendiente. Nadie todavía; hoy la BD tiene 0 casos | 1 | Sí | Desbloquea H13 |
| **[N1 · 2/4] H9** · El mismo endpoint usa cuatro formatos de error distintos | `routes.ts:311` (400 `error`), `:314` (422 `message`), `:317` (400 `errores[]`), `:320` (200 `motivo`) | El frontend tiene que adivinar qué campo leer. Lo nota el equipo de front | 4 | Sí | Abarata H10 |
| **[N1 · 1/4] H6** · La columna `estado` es texto libre, sin CHECK ni enum | `backend/database/migrations/1769000000005_create_comentarios_table.ts:18`. En BD: `aprobado`=4, `Aprobado`=2, `ok`=1, `pendiente`=2 | Nadie lo nota directamente; es la causa de H4 y H5 | 1 migración nueva + 3 filas (ids 3, 4, 9) | No: al normalizar los datos con UPDATE se pierde el valor original | Abarata H4 y H5 |
| **[N2 · 5/1] H2b** · Arreglo de fondo de H2: `DELETE` decide quién es el autor por el email que llega en el body, y ese email es público por H1 | `routes.ts:350-353` | Cualquiera puede borrar cualquier comentario aprobado. Lo nota quien escribió el comentario cuando desaparece | 1 (bloque `350-355`) | No: es un borrado físico (`:359`) y el revert no devuelve las filas | Lo abarata H1; lo precede H20 |
| **[N2 · 3/1] H4** · El detalle solo enseña `estado = 'aprobado'` escrito exactamente así | `routes.ts:153`. En BD los ids 3 y 9 son `'Aprobado'` y el 4 es `'ok'` | Hay 3 comentarios aprobados que no se ven. Lo notan Rubén, Silvia y Paula, que escribieron los ids 3, 4 y 9 | 1 | Sí | Lo abarata H6 |
| **[N2 · 2/1] H16** · Al enviar, el contador de la sección suma 1 por un comentario que está pendiente; el de la cabecera no cambia | `frontend/src/componentes/Comentarios.jsx:39` frente a `DetallePost.jsx:71` | Los dos contadores dejan de coincidir y cuentan algo que no se ve. Lo nota quien comenta | 1 (`Comentarios.jsx:39`; los dos contadores arrancan del mismo dato en `DetallePost.jsx:38` y `Comentarios.jsx:23`) | Sí | A ninguno |
| **[N2 · 2/1] H10** · Con un email sin `@`, la web enseña «No se ha podido enviar.» | `Comentarios.jsx:44` (no lee `errores`) | El lector no sabe qué ha hecho mal | 1 | Sí | Lo abarata H9 |
| **[N2 · 2/1] H13** · La web no permite responder: el formulario no envía `padre_id` | `Comentarios.jsx:33` | Las respuestas se pintan, pero el lector no puede crear una | 1 | Sí | Lo bloquea H12 (activarlo sin validar deja H12 a la vista) |
| **[N2 · 2/1] H8** · Un cuerpo de menos de 5 caracteres responde `200 {ok:false}` | `routes.ts:319-320` | Un cliente que se fíe del código HTTP lo toma por éxito. La web no lo nota porque lee `motivo` | 1 | Sí | A ninguno |
| **[N2 · 4/2] H3b** · Arreglo de fondo de H3: la clave de moderación está escrita en el código y en git | `routes.ts:25` (`'dev-secret-cambiar'`), usada en `:351`. En el historial desde `e0e0943` (`git log -S dev-secret-cambiar`) | Quien tenga acceso al repo puede borrar cualquier comentario, también los pendientes. Nadie todavía | 2 (`:25`, `:351`) | No: el secreto queda en el historial de git y hay que rotarlo | A ninguno |
| **[N2 · 3/2] H5** · El listado y los relacionados cuentan `estado <> 'pendiente'` mientras el detalle filtra `= 'aprobado'` | `routes.ts:75-76`, `:223` | Se incumple la regla 5. En vivo, el post 6 dice «1 comentario» en el listado y en el detalle no sale ninguno; en el post 1 son 4 frente a 2 (según BD). Lo nota el lector | 2 (`:75`, `:223`) | Sí | Lo abarata H6 |
| **[N2 · 2/2] H7** · El POST devuelve la última fila de la tabla, no la que acaba de insertar | `routes.ts:329-331` (`order by id desc limit 1`) | Si llegan dos POST a la vez, uno de los dos recibe el comentario del otro, con su email. Nadie todavía | 2 (`:324`, `:330`) | Sí | A ninguno |
| **[N2 · 2/2] H15** · El DELETE borra solo la fila y sus respuestas se quedan con un `padre_id` que ya no existe | `routes.ts:359` + `Comentarios.jsx:50` | La respuesta deja de pintarse. Si se borra el id 1, desaparece la respuesta de Marta (id 2). Lo nota quien escribió la respuesta | 2 | No: el borrado es físico | A ninguno |
| **[N2 · 1/1] H11** · La validación del email solo comprueba que haya una `@` | `routes.ts:316` | Se aceptan `@` o `a@` como email. Nadie todavía | 1 | Sí | A ninguno |
| **[N2 · 1/1] H14** · La web solo pinta dos niveles de hilo | `Comentarios.jsx:50`, `:63-64` | Una respuesta a otra respuesta nunca se ve. Nadie todavía; no hay casos en BD | 1 | Sí | A ninguno |
| **[N2 · 1/1] H18** · PATCH no exige longitud mínima | `routes.ts:384` | Un cuerpo de 1 carácter se acepta al editar, pero el POST exige 5. Nadie todavía | 1 | Sí | A ninguno |
| **[N2 · 1/1] H19** · `post_id` y `padre_id` no tienen clave foránea ni índice | migración `:14`, `:20`. El `CREATE TABLE` real en BD lo confirma | Si se borra un post, sus comentarios quedan huérfanos. Nadie todavía; hoy hay 0 huérfanos | 1 migración | Sí | A ninguno |
| **[N2 · 1/2] H17** · PATCH compara el email exacto y DELETE lo compara sin mayúsculas ni espacios | `routes.ts:379` frente a `:353` | Un autor que escriba su email con mayúsculas puede borrar su comentario pero no editarlo. Nadie todavía | 2 | Sí | A ninguno |

### Propuestas sin medir (fuera de la tabla)

- **Que retirar el DELETE (H2a) no afecte a nadie.** Medido solo en la web (0 usos). Falta medir: peticiones `DELETE /comentarios/*` en los logs de acceso de la API.
- **Que H7 entregue un comentario ajeno.** Falta medir: dos POST concurrentes contra la BD de test y comparar el `id` devuelto con el insertado.
- **Que H19 cause lentitud.** Falta medir: `EXPLAIN QUERY PLAN` de las consultas de `routes.ts:75`, `:153` y `:223` con un volumen realista de comentarios.
- **Que haga falta límite de envíos o anti-spam.** Falta medir: POST de comentarios por IP y hora, y cuántos acaban como spam.
- **Que H1 ya se haya aprovechado.** Falta medir: peticiones `GET /posts/:slug` desde fuera de la web (por User-Agent u origen) en los logs.

## Auditoria adversarial complementaria mediante otro cliente / modelo.

La revisión complementaria coincide con la mayoría de los hallazgos anteriores. Diferencias principales:


| Hallazgo actual                                   | ¿Coincide?                   | Resultado resumido                                                                                                                |
| ------------------------------------------------- | ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| H1 · Exposición de correos                        | Sí, en código                | `routes.ts:153,171`; no se revalidó en vivo.                                                                                      |
| H2 · Borrado autorizado por correo                | Parcialmente                 | Confirmado en `routes.ts:350-355`; H1 lo facilita, pero no es requisito.                                                          |
| H3 · Clave de moderación en código y Git          | Parcialmente                 | Literal y uso confirmados en `routes.ts:25,351`; historial Git no comprobado.                                                     |
| H4 · Estados excluidos del detalle                | Parcialmente                 | Filtro y tres registros confirmados; H6 no bloquea su corrección.                                                                 |
| H5 · Recuentos distintos al detalle               | Parcialmente                 | Consultas y cifras confirmadas; puede corregirse sin resolver H6.                                                                 |
| H6 · Estado sin restricción de valores            | Parcialmente                 | Esquema confirmado; restringir valores y normalizar tres registros son trabajos distintos.                                        |
| H7 · Respuesta del POST tomada del mayor ID       | Parcialmente                 | Consulta confirmada en `routes.ts:330`; entrega cruzada bajo concurrencia no reproducida.                                         |
| H8 · Cuerpo corto responde sin estado de error    | Sí, en código                | Confirmado en `routes.ts:319-320`.                                                                                                |
| H9 · Cuatro formatos de error                     | Parcialmente                 | Confirmados en `routes.ts:311,314,317,320`; no bloquea H10.                                                                       |
| H10 · Mensaje genérico para correo inválido       | Parcialmente                 | `Comentarios.jsx:44` omite `errores`; puede corregirse independientemente de H9.                                                  |
| H11 · Validación del correo por presencia de @    | Sí, en código                | Confirmado en `routes.ts:316`.                                                                                                    |
| H12 · Padre guardado sin validar                  | Parcialmente                 | Confirmado en `routes.ts:308,326`; resolverlo antes de H13 es una precaución, no un bloqueo técnico.                              |
| H13 · Formulario sin destinatario de respuesta    | Parcialmente                 | Confirmado en `Comentarios.jsx:33`; requiere selección del padre y envío, no solo añadir un campo al JSON.                        |
| H14 · Solo dos niveles visibles                   | Sí, en código                | `Comentarios.jsx:50,63-72`; no hay terceros niveles actuales en BD.                                                               |
| H15 · Respuestas ocultas al borrar su padre       | Parcialmente                 | Consecuencia acreditada por código y relación 1→2; borrado no ejecutado y solución dependiente de la política de descendientes.   |
| H16 · Incremento de pendientes en el contador     | Parcialmente                 | Confirmado en `Comentarios.jsx:39`; retirar ese incremento basta para evitar esa divergencia.                                     |
| H17 · Normalización distinta del correo           | Parcialmente                 | Confirmado en `routes.ts:353,379`; igualar el criterio requiere un bloque, no necesariamente dos.                                 |
| H18 · PATCH sin mínimo de cinco caracteres        | Sí, en código                | Confirmado en `routes.ts:384` frente a `:319`.                                                                                    |
| H19 · Relaciones e índices ausentes               | Parcialmente                 | Esquema confirmado; agrupa tres asuntos: relación con posts, relación con padres e índices. Sin huérfanos ni lentitud demostrada. |
| H20 · Prueba limitada a un campo                  | Parcialmente                 | `comentarios.spec.ts:41-42` confirma las aserciones; no bloquea funcionalmente los demás hallazgos.                               |
| Adicional A4 · PATCH autoriza por correo recibido | Falta en la tabla actual     | `routes.ts:378-380`: conocer ID y correo de un pendiente supera esa comprobación de autoría.                                      |
| Adicional A20 · PATCH solo admite pendientes      | Falta como hallazgo separado | `routes.ts:379`: los aprobados no se pueden editar; comportamiento confirmado, sin regla que lo declare defecto.                  |
| Adicional A27 · Nombre formado solo por espacios  | Falta                        | `routes.ts:310,326`: supera obligatoriedad y se pasa a la inserción.                                                              |
| Adicional A28 · Cuerpo de cinco espacios          | Falta                        | `routes.ts:319,326`: supera la validación y se pasa a la inserción.                                                               |


Los adicionales A24 (errores de carga), A25 (doble petición) y la observación de arquitectura ya figuran al pie de la auditoría original: no son omisiones. Los identificadores A conservan la numeración de la revisión complementaria entregada en el chat. Las discrepancias parciales afectan al alcance, las dependencias o la evidencia, no necesariamente a la existencia del problema.

- **Hallazgos adicionales:** PATCH también usa el correo recibido como prueba de autoría (`backend/start/routes.ts:378-380`); el nombre compuesto solo por espacios supera su validación (`:310`); un cuerpo de cinco espacios supera la del POST (`:319`). No se realizaron escrituras para reproducirlos.
- **Bloqueos sobredimensionados:** H1 facilita H2, pero conocer el correo por otra vía basta; H6 no es requisito para corregir H4/H5; H9 no impide corregir H10. H12 es una precaución antes de exponer respuestas y H20 una obligación de pruebas previa a corregir, no una dependencia funcional.
- **Recuentos de cambios:** H16 puede corregirse retirando el incremento de `frontend/src/componentes/Comentarios.jsx:39`, sin modificar siete líneas en dos archivos. H19 agrupa relaciones con posts, relaciones entre comentarios e índices: son asuntos distintos.
- **Límites de la evidencia:** H7 demuestra una selección del mayor ID global (`backend/start/routes.ts:330`), no una entrega cruzada reproducida. No se comprobó el historial Git de H3 ni que esa constante firme sesiones; el cifrado configurado usa `APP_KEY` (`backend/config/encryption.ts:16`).
- **Datos confirmados:** nueve comentarios: cuatro `aprobado`, dos `Aprobado`, uno `ok` y dos `pendiente`; post 1: recuento 4 frente a 2 del detalle; post 6: 1 frente a 0. Sin huérfanos, padres inválidos ni terceros niveles actuales.
- **Matices:** representar los cuerpos como texto (`frontend/src/componentes/Comentarios.jsx:61,71`) no certifica ausencia general de XSS; devolver al remitente su propio correo no demuestra por sí solo una filtración; `localhost:3401` coincide con la configuración exigida del proyecto.

Verificación complementaria: código y copia en memoria de SQLite con `query_only`, sin modificar datos. Las conexiones locales quedaron bloqueadas por el entorno: no se revalidaron API ni web en ejecución. Un revert de código tampoco recupera cuerpos sobrescritos, filas borradas ni confidencialidad perdida.

## Otros hallazgos, fuera de la tabla

- **Arquitectura:** las tres rutas de comentarios (`routes.ts:295-393`) llevan dentro SQL, validación y autorización, y no hay controlador, servicio ni transformer. Los errores se montan en cada ruta y no pasan por `backend/app/exceptions/handler.ts`.
- **El POST devuelve campos internos:** la respuesta lleva `autor_email`, `estado` y `post_id` (`routes.ts:333`).
- **La web no tiene botones para editar ni borrar.** No hay línea que lo demuestre porque es una ausencia. Se comprueba buscando `DELETE` o `PATCH` en `frontend/src`: 0 resultados.
- **No hay límite de envíos ni anti-spam en el POST.** También es una ausencia: no hay middleware de throttle en `backend/start/kernel.ts:35-41`.
- **Doble petición del mismo endpoint:** `DetallePost.jsx:34` y `Comentarios.jsx:19` piden el post entero.
- **Errores de carga silenciados:** `Comentarios.jsx:25` tiene `.catch(() => {})`.
- **Detalles menores de la web:** la URL de la API está escrita a mano (`Comentarios.jsx:4`), las listas usan `key={index}` (`:57`, `:66`) y el campo de correo no es `type="email"` (`:79`).
- **Sin historial de ediciones:** PATCH sobrescribe el cuerpo sin guardar el anterior ni una fecha de edición (`routes.ts:388`).
- **Lo que está bien:** no hay XSS en los comentarios. React escapa `{c.cuerpo}` (`Comentarios.jsx:61`) y no se usa `dangerouslySetInnerHTML`.

## Parte B: las tres líneas

1. **Hallazgos devueltos por el agente:** 20 en la tabla + 8 al pie. **Comprobados abriendo el archivo:** 5
2. **Hallazgo sin prioridad clara: H1 frente a H20.** H1 es lo más grave (los correos salen hoy en la API), pero H20 bloquea su arreglo porque CLAUDE.md obliga a caracterizar antes de corregir: lo urgente contra lo que bloquea.
3. **Afirmado y sin verificar: H7**, la condición de carrera del POST. Se deduce de `routes.ts:329-331`, pero comprobarla exige lanzar dos POST concurrentes que escriben en BD, y la auditoría era de solo lectura. Por el mismo motivo, el borrado ajeno de H2 tampoco se ejecutó.

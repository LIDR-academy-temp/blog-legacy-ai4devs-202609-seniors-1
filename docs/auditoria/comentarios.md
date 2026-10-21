# Auditoría de comentarios

Base: `s6/start`, commit `a87a41404edca3ec8a0a8289e770d73e58e7f0b7`. Alcance: crear, editar, borrar y listar comentarios, desde la API y SQLite hasta la web. Las comprobaciones técnicas las hizo el agente.

Primera pasada: 21/10/2026, 21:08:52–21:13:51 (Madrid), **4 min 59 s** sobre un máximo de 45 minutos. No incluye preparar el entorno ni la entrega. Esta revisión de redacción es posterior; no añade comprobaciones técnicas.

## Parte A

Referencias: **R** = `backend/start/routes.ts`; **T** = `backend/tests/functional/comentarios.spec.ts`; **M** = `backend/database/migrations/1769000000005_create_comentarios_table.ts`; **C** = `frontend/src/componentes/Comentarios.jsx`; **D** = `frontend/src/componentes/DetallePost.jsx`.

Orden: primero dependencias, después daño/radio. H1 corta el borrado y la edición sin autenticar; H2 quita la fuga de correos; H3 prepara las pruebas para H4–H7. Después van identidad, respuestas con padre inválido y contadores. La posición de H4 sigue abierta: falta saber cuánto habría que tocar.

«Sitios» cuenta los bloques que tocaría el cambio mínimo propuesto; las pruebas se cuentan en H3. El revert deshace código, no recupera datos borrados o correos filtrados. H4 tiene casillas vacías porque siguen sin resolverse. Son siete filas: H1 y H4 separan contención y arreglo de fondo del mismo problema.

| Qué es | Dónde está, con línea | Qué se rompe, y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
|---|---|---|---|---|---|
| **H1. Contención:** cerrar PATCH y DELETE hasta comprobar quién los llama. | [R:349–359](../../backend/start/routes.ts#L349), [R:378–388](../../backend/start/routes.ts#L378), clave en [R:25](../../backend/start/routes.ts#L25). | Un visitante con el email edita pendientes o borra comentarios; la clave escrita en el código también permite borrar. Lo sufren quienes comentan. | **2** manejadores: PATCH y DELETE. | **Sí**; reabrirlos devuelve el riesgo. Un revert no recupera comentarios borrados. | Contiene el daño de **H4** mientras se prepara el arreglo. |
| **H2.** El detalle público expone el email de quienes comentan. | [R:153](../../backend/start/routes.ts#L153) selecciona toda la fila; [R:171](../../backend/start/routes.ts#L171) la publica. | Cualquier lector obtiene `autor_email`; afecta a los autores y facilita el borrado de H1. | **1** SELECT de comentarios del GET: devolver solo los campos públicos. | **Sí**; no retira correos ya descargados. | A ninguno; quitar el correo no sustituye **H4**. |
| **H3.** La suite no protege el contrato de comentarios. | [T:35–42](../../backend/tests/functional/comentarios.spec.ts#L35): solo POST, estado 201 y nombre. | Nadie todavía; un cambio puede romper permisos, privacidad o moderación y el test seguir verde. | **1** archivo T para caracterizar GET/POST/PATCH/DELETE; el contador visual se comprueba en navegador. | **Sí**. | Desbloquea comprobaciones de regresión de **H4–H7**. |
| **H4. Arreglo de fondo:** la identidad depende del email recibido y de una clave pública. | [R:350–353](../../backend/start/routes.ts#L350), [R:378–379](../../backend/start/routes.ts#L378); [M:14–20](../../backend/database/migrations/1769000000005_create_comentarios_table.ts#L14) no vincula comentarios con usuarios. | Suplantación del autor; también difieren las reglas de edición y borrado. Falta decidir cómo saber de quién son los comentarios anónimos antiguos. | |  | Depende de **H3**; mantener **H1** hasta verificarlo. |
| **H5.** POST acepta un padre inexistente o perteneciente a otro post. | [R:308](../../backend/start/routes.ts#L308), [R:325–326](../../backend/start/routes.ts#L325), [M:20](../../backend/database/migrations/1769000000005_create_comentarios_table.ts#L20); render en [C:50–64](../../frontend/src/componentes/Comentarios.jsx#L50). | La respuesta se guarda; si se aprueba con padre inexistente, GET la cuenta pero la web la oculta. Lo notan autor y lectores. | **1** bloque de validación previo al INSERT para comprobar existencia y mismo post; no repara datos previos. | **Sí** para esa validación. | Depende de la caracterización de **H3**. |
| **H6.** Los recuentos incluyen estados distintos de `aprobado`. | [R:75–76](../../backend/start/routes.ts#L75) y [R:223](../../backend/start/routes.ts#L223), frente a [R:153](../../backend/start/routes.ts#L153). | El post 1 anuncia 4 y devuelve 2; el post 6 aparece como relacionado con 1 y tiene 0 aprobados exactos. Los lectores ven cifras incompatibles. | **2** filtros SQL: listado y relacionados; contar solo `aprobado`. | **Sí**; no se normalizan datos. | Depende de **H3**; no necesita migrar `Aprobado`/`ok`. |
| **H7.** Enviar un pendiente incrementa solo el contador inferior. | [C:37–39](../../frontend/src/componentes/Comentarios.jsx#L37), frente a [D:38](../../frontend/src/componentes/DetallePost.jsx#L38) y [D:71](../../frontend/src/componentes/DetallePost.jsx#L71). | Tras enviar, cabecera 0 y sección 1, sin comentario visible; al recargar vuelve a 0. Lo ve quien envía. | **1** actualización `setTotal(total + 1)` en C: no contar el pendiente como aprobado. | **Sí**. | Depende de **H3** y de repetir la comprobación visual; no depende de **H6**. |

## Parte B

1. Hallazgos del agente: 7. Comprobados personalmente: 3.
2. H4 es el que deja dudas: suplantar al autor es grave, pero arreglar la identidad de los comentarios antiguos puede tocar bastante más; H1 corta el daño mientras se decide.
3. El agente señaló que el SELECT del último comentario (R:329–333) podría devolver el de otra petición; quedó sin comprobar con peticiones simultáneas.

## Qué se comprobó

- `make setup` y `make db`: completados con Node **24.22.1**, npm **11.19.1** y make **3.81**. API en 3401 y web en 5401; comprobadas con GET `/posts` y abriendo un post en Chrome.
- Primer `make test`: `listen EPERM: operation not permitted 0.0.0.0:3401`; no ejecutó tests aunque el proceso devolvió 0. Repetido con permiso para escuchar en el puerto local: **1 passed (1), 50 ms, salida 0**. No se corrigió código ni se añadieron tests al repositorio.
- H1/H4: con comentarios de prueba, PATCH sin token ni cookie y con email coincidente devolvió 200 y guardó el cambio; DELETE devolvió 200 y eliminó la fila. Con email distinto, ambos devolvieron 403. La clave escrita en el código permitió otro DELETE sin email.
- H2: GET del post 1 devolvió dos comentarios aprobados con `autor_email`. La web no pinta ese campo, pero la API sí lo entrega.
- H3: se abrió el único archivo de tests y se contrastaron sus dos aserciones. Estas comprobaciones puntuales no están añadidas a la suite.
- H5: POST con `padre_id=999999` y con padre 6 de otro post devolvió 201. Solo para probar el render, el agente aprobó en SQLite su comentario de prueba sin padre: salió en GET y en el total, pero no en el DOM.
- H6: se compararon respuestas HTTP con filas y estados reales de SQLite. Hay `aprobado`, `Aprobado`, `ok` y `pendiente`; no se ha dado por hecho que los tres primeros signifiquen lo mismo.
- H7: envío desde el formulario, lectura de ambos contadores, comprobación de estado `pendiente` en SQLite y recarga. No hay controles de edición o borrado en el componente; esas operaciones se comprobaron por API.
- Fronteras comprobadas: pendiente oculto en GET; POST corto devuelve `200 {ok:false}`; nombre ausente 400, email ausente 422, email inválido 400; post inexistente y PATCH/DELETE inexistentes 404. PATCH permite un cuerpo de un carácter: se anotó, pero no se cambió ni se decidió aquí si debía aceptarse.

## Pendiente, fuera de la tabla

- **Concurrencia del INSERT + SELECT global:** lanzar altas paralelas con identificadores distintos y comparar cada respuesta con su fila persistida. R:329–333 apunta al problema; falta reproducirlo.
- **Índice de comentarios:** SQLite no tiene índices secundarios en esta tabla. Antes de añadir uno, medir el plan de consulta y los tiempos con un volumen realista. Con nueve filas no se ha demostrado un problema de rendimiento.
- **Normalizar `Aprobado`/`ok`:** confirmar qué significaban antes de convertirlos a `aprobado`. El seeder da pistas, pero no demuestra que todos estuvieran aprobados.

Solo se ha auditado el entorno local. Los cambios de la tabla están propuestos, no aplicados.

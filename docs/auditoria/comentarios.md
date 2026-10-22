# Auditoría parcial — funcionalidad de comentarios

Alcance: crear, editar, eliminar y listar comentarios (backend, frontend, base de datos).
Auditoría de solo lectura: no se ha modificado código de la aplicación ni se ha corregido nada.

Línea base: `make test` ejecuta la única prueba funcional de comentarios
(`backend/tests/functional/comentarios.spec.ts`) y falla con `SqliteError: no such table: posts`.
No se ha corregido.

## Estado de verificación

- **Comprobados abriendo el código:** H1, H2, H5, solo en las líneas indicadas en la matriz.
- **Devueltos por el agente y sin comprobar:** H0, H3, H4, H6, H7, H8. Aparecen en la matriz con
  las celdas en blanco salvo la descripción.
- **Hipótesis sin medir:** H9, H10, H11. Están fuera de la tabla.

Todos los defectos de la funcionalidad llegan en `e0e0943 initial import`, el commit que crea el
repositorio. Revertirlo borraría el proyecto entero, así que la columna «git revert» se refiere al
commit de corrección futuro, no al que introdujo el defecto.

## Matriz

Orden: primero precedencia (lo que debe ir antes que lo demás) y después daño ÷ radio de impacto
(más daño con menos sitios que tocar va antes). Es una estimación cualitativa, no medida.

| Prioridad | Qué es | Dónde está (línea) | Qué se rompe y quién lo nota | Sitios exactos a tocar | ¿Se deshace con git revert? | Dependencias |
|---|---|---|---|---|---|---|
| 1 (empate sin resolver) | **H2 · contención**: la clave de moderación `SECRETO_SESION = 'dev-secret-cambiar'` está escrita en el código y permite borrar cualquier comentario con la cabecera `x-clave-moderacion` | `backend/start/routes.ts:25`, `:349-351` | Cualquiera que lea el repositorio puede borrar todos los comentarios; lo notan quienes comentan y la moderación | `routes.ts:351` (anular la comprobación con la clave) | Sí, el commit; pero revertirlo vuelve a abrir el acceso | Ninguna entre los comprobados |
| 1 (empate sin resolver) | H0: la suite falla porque la base no tiene tablas (reportado por el agente, sin comprobar) | | | | | |
| 2 | **H1 · contención**: el correo de quien comenta sale en respuestas públicas porque se devuelve la fila de `select * from comentarios` | `routes.ts:153`, `:171`, `:329-333` | Se expone un dato personal de cualquiera que haya comentado; lo nota esa persona. Incumple la regla de dominio 6 | `routes.ts:153` (columnas explícitas, sin `autor_email`), `routes.ts:329-333` (respuesta del POST) | Sí, el código; los correos ya expuestos no se recuperan | Ninguna entre los comprobados |
| 3 | **H2 · fondo**: sacar el secreto del código y rotarlo | `routes.ts:25`, `:349-351` | Ídem H2 | `routes.ts:25`, `:349-351` | No del todo: la clave sigue en el historial de git y hay que rotarla | Va después de H2 · contención |
| 4 | **H1 · fondo**: forma de respuesta mediante transformer | `routes.ts:153`, `:171`, `:329-333` | Ídem H1 | `routes.ts:171`, `:333`, más el transformer nuevo | Sí, el código | Va después de H1 · contención |
| 5 | **H5**: `estado` es texto libre y cada consulta lo filtra distinto: el detalle con `= 'aprobado'`, el listado y los relacionados con `<> 'pendiente'` | `routes.ts:75-76`, `:153`, `:223` | Recuentos de comentarios distintos entre listado, relacionados y detalle; un estado nuevo (por ejemplo `rechazado`) contaría como aprobado en los recuentos. Lo notan los lectores y la moderación. Toca las reglas de dominio 2 y 5 | `routes.ts:75-76`, `:153`, `:223` (3 consultas), además de la normalización de datos | El código sí; la normalización de datos en SQLite no, porque la base no está en git | Requiere decidir antes qué valores cuentan como aprobado |
| — | H3: autorización de borrar y editar basada solo en el correo (reportado por el agente, sin comprobar) | | | | | |
| — | H4: regla de autorización duplicada e incoherente entre DELETE y PATCH (reportado por el agente, sin comprobar) | | | | | |
| — | H6: tres recuentos distintos en el frontend (reportado por el agente, sin comprobar) | | | | | |
| — | H7: `padre_id` sin validar y sin claves foráneas, respuestas huérfanas (reportado por el agente, sin comprobar) | | | | | |
| — | H8: validación de entrada incoherente y con huecos en POST y PATCH (reportado por el agente, sin comprobar) | | | | | |

## Hipótesis no medidas

- **H9:** el POST lee el comentario recién creado con `order by id desc limit 1` y no por su id.
  Si llegan dos peticiones a la vez, una podría recibir el comentario de otra persona, correo
  incluido. No se ha reproducido.
- **H10:** un PATCH con `cuerpo` que no sea texto (un objeto o un array) podría acabar en un
  error 500 en vez de un error de validación. No se ha probado.
- **H11:** el frontend no ofrece editar ni borrar, y `DetallePost` y `Comentarios` piden los dos
  `/posts/:slug`. El coste de esa doble petición no se ha medido.
- **Descartado de momento:** XSS en comentarios. Se pintan como texto de React; no se ha probado
  en ejecución.

12 hallazgos devueltos y 3 comprobados por mí.
No pude decidir entre contener H2 por su riesgo y atender H0 porque desbloquea las pruebas.
No pude verificar si H9 devuelve comentarios de otra persona bajo concurrencia.

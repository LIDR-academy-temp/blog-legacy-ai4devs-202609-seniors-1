# Auditoría priorizada: feature de comentarios

Alcance: crear, editar, borrar y listar comentarios en sus tres capas (rutas de la API, tabla
`comentarios` en la base de datos y lo que enseña la web). Solo esta feature.

Método: mapa técnico → candidatos → verificación adversarial contra el código → priorización.
Todo procede de inspección de código: no se ha ejecutado la API ni la suite. Las líneas sin
ruta son de `backend/start/routes.ts`.

Decisión de negocio tomada en la sesión: **`estado = 'ok'` cuenta como aprobado.**

Orden: primero lo que desbloquea o abarata otros hallazgos; dentro de cada nivel, más daño
por radio afectado primero. Dependencias: 🔗 se desprende del código · 📏 se desprende de las
reglas de proceso de `CLAUDE.md` (caracterizar antes de cambiar comportamiento). Una casilla
vacía significa que no se pudo comprobar.

## Parte A: hallazgos

| # | Qué es | Dónde está (línea que lo demuestra) | Qué se rompe, y quién lo nota | Cuántos sitios hay que tocar | ¿Se deshace con un revert? | ¿Bloquea o desbloquea a otro hallazgo? |
|---|---|---|---|---|---|---|
| 1 | La feature solo tiene un test, y solo afirma `201` y `autor_nombre`. | `backend/tests/functional/comentarios.spec.ts:15-43` (`:41-42`). | Nadie hoy. Todo cambio posterior se haría sin red: PATCH, DELETE, listado en el detalle y recuentos no tienen test. | 1 archivo (4 comportamientos sin test). | Sí: solo añade tests. | 📏 Desbloquea a todos los demás. |
| 2 | El correo de quien comenta sale en la respuesta pública del detalle del post. | `:153` (`select * from comentarios`), devuelto tal cual en `:171`. | Cualquier cliente de la API lee el correo de los comentaristas aprobados (4 en la semilla). La web no lo pinta. Incumple la regla 6. | 1 (`:153` o `:171`; basta uno). | Sí el código. Lo ya expuesto no se recupera. | 🔗 Reduce el daño del #7: es la fuente pública de la credencial con la que se borra. No lo cierra. |
| 3 | Contención: la clave de moderación está escrita en el código. | `:25` (`'dev-secret-cambiar'`), comparada en `:351`. En el historial desde `e0e0943` (`git log -S`). | Cualquiera con acceso al repo borra cualquier comentario, también los pendientes. | 2 (`routes.ts:25`, `start/env.ts`) + rotar la clave. | No: la clave sigue en el historial de git; hay que rotarla. | Sin evidencia de que bloquee a otro. |
| 4 | Contención: el detalle no reconoce `'Aprobado'` ni `'ok'` como aprobados. | `:153` (igualdad exacta); datos en `backend/database/seeders/blog_seeder.ts:510`, `:522`, `:575`. | Los comentarios 3 y 4 (post 1) y 9 (post 6) no se muestran. Lo notan quienes los escribieron. | 1 (`:153`). | Sí. | A ninguno por sí solo. |
| 5 | Definitivo: `estado` es texto libre, sin CHECK ni enum. | Migración `backend/database/migrations/1769000000005_create_comentarios_table.ts:18`; datos en `blog_seeder.ts:510`, `:522`, `:575`. | Puede volver a entrar cualquier valor y repetir el #4. | 3 filas a normalizar a `'aprobado'` + 1 migración con CHECK (`'pendiente'`, `'aprobado'`). | No: la normalización de datos no se deshace con un revert de código. | 🔗 Desbloquea el #9 y el #10. |
| 6 | Definitivo: el moderador se identifica con una clave compartida. | `:349-351`. | Quien tenga la clave es moderador, sin trazabilidad. | 1 existente (`:349-351`); lo nuevo no se ha podido contar. | Sí. | 🔗 Desbloquea el #10 (aprobar exige un moderador autorizado). |
| 7 | Definitivo: borrar se autoriza con el correo que manda el cliente. | `:350-353`; borrado físico en `:359`. Sin CSRF (`backend/config/shield.ts:33`) y sin `middleware.auth()` en la ruta (solo en `:416`). | Quien conozca el correo de un comentarista borra su comentario. Lo nota el autor. | 1 existente (`:350-353`); lo nuevo no se ha podido contar. | Sí el código; lo ya borrado, no. | Lo agrava el #2. Arreglo incierto: el repo no muestra cómo identificar a quien comenta. |
| 8 | Editar aplica otra regla que borrar: correo con mayúsculas, solo pendientes, cuerpo de 1 carácter. | `:379` y `:384`, frente a `:352-353` y `:319`. | Quien guardó `Ana@x.es` y envía `ana@x.es` puede borrar pero no editar. Editando se salta el mínimo de 5 caracteres. Lo nota el comentarista. | 2 (`:379`, `:384`). | Sí. | 🔗 Comparte `:379` con el #7: si este unifica la regla, queda solo `:384`. |
| 9 | El recuento usa `<> 'pendiente'` en vez de «solo aprobados». | `:75-76` y `:223`, frente a `:153`. | Con los datos de la semilla y `'ok'` como aprobado, el número del listado es correcto; el desfase visible viene del #4. Nadie todavía: un estado nuevo contaría como aprobado. Incumple la regla 5. | 2 (`:75-76`, `:223`). | Sí. | 🔗 Bloqueado por el #5. |
| 10 | No existe ninguna operación para aprobar comentarios. | Ninguna ruta de `:31-418` cambia `estado` (único `update comentarios` del repo: `:388`, solo `cuerpo`); el POST fija `'pendiente'` en `:326`. | Todo comentario nuevo queda invisible para siempre. Lo nota quien comenta. Comprobado solo dentro de este repo. | | Sí. | 🔗 Bloqueado por el #5 y el #6. |
| 11 | Los dos contadores del frontend se separan al comentar. | `frontend/src/componentes/Comentarios.jsx:39`; `frontend/src/componentes/DetallePost.jsx:30`, `:38`, `:71`. | La cabecera dice N y la sección N+1. Lo ve el lector que comenta. | 3 líneas en 2 componentes (`Comentarios.jsx:39`, `DetallePost.jsx:30`, `:38`). | Sí. | Sin evidencia. |
| 12 | El POST responde los errores con códigos y formas distintas, y el cuerpo corto con 200. | `:311` (400 `{error}`), `:314` (422 `{message}`), `:317` (400 `{errores}`), `:320` (200 `{ok:false}`). | Un cliente de la API toma el comentario corto por aceptado. | 4. | Sí. | 📏 Bloqueado por el #1 (cambio de contrato). Desbloquea el #13. |
| 13 | El frontend no lee la forma de error `errores`. | `Comentarios.jsx:44`, frente a `:317`. | Con un correo sin `@`, el usuario ve «No se ha podido enviar.» sin el motivo. | 1. | Sí. | 🔗 Va después del #12. |
| 14 | El frontend se traga los errores al cargar los comentarios. | `Comentarios.jsx:25` (`catch(() => {})`) y `:22-23`. | Si la petición falla o devuelve 404, la sección muestra «0 comentarios» sin aviso. Lo nota el lector solo cuando falla. | 1. | Sí. | Sin evidencia. |
| 15 | `padre_id` no se valida al crear. | `:308`; migración `:20` sin FK. | Desde la web, nadie todavía: no envía `padre_id` (`Comentarios.jsx:33`). Solo quien use la API. | 1 (`:308`), +1 migración si se añade la FK. | Sí, si la migración tiene `down`. | Sin evidencia. |
| 16 | Al borrar un comentario, sus respuestas quedan huérfanas. | `:359`; pintado en `Comentarios.jsx:50`, `:63-64`. | Al borrar el comentario 1, el 2 (`blog_seeder.ts:500`) deja de mostrarse. | 1 (`:359`). | Sí el código; lo borrado, no. | Sin evidencia. |
| 17 | El frontend solo pinta un nivel de respuestas. | `Comentarios.jsx:63-64`. | Una respuesta a otra respuesta nunca se ve. Nadie hoy desde la web. | 1. | Sí. | Sin evidencia. |
| 18 | Toda la lógica de comentarios vive en las rutas. | `:295-393`; no hay archivos de comentarios en `backend/app/`. | Nadie fuera del equipo. Es el origen de la regla duplicada del #7 y el #8. | 3 rutas (`:295`, `:339`, `:366`) + la consulta `:152-155`. | Sí. | 📏 Bloqueado por el #1. |

### Fuera de la tabla: pendiente de medir

| Propuesta | Evidencia | Qué habría que medir |
|---|---|---|
| FK e índice en `comentarios.post_id` | Migración `:14` (nota en `:6`) | Tiempo de `:153`, `:75-76` y `:223` con volumen real |
| Recuento de relacionados en una sola consulta | `:220-229` | Latencia de `GET /posts/:slug/relacionados` (en buena parte, recurso posts) |
| No pedir el post dos veces en el frontend | `Comentarios.jsx:19` y `DetallePost.jsx:34` | Peticiones y tiempo por artículo abierto |
| Anti-spam o límite de peticiones en el POST | Sin límite en `:295-334`; spam en `blog_seeder.ts:547-556` | Volumen real de comentarios basura |

### Excluidos como hechos

- **Un cuerpo de solo espacios pasa la validación**: refutado. El bodyparser recorta por
  defecto (`trimWhitespaces: true` en `node_modules/@adonisjs/bodyparser/build/define_config-Jdzxr5yj.js:37`)
  y convierte `""` en `null`, que rechaza `:319`.
- **El POST devuelve el comentario de otro** (`:329-331`): requiere un test de concurrencia.
- **Mezcla de zonas horarias en `creado_en`** (`:323` frente a la semilla): no verificable.

### Decisiones difíciles

1. **Tests antes que seguridad.** El #3 y el #7 hacen más daño que la falta de tests, pero
   `CLAUDE.md` exige un test en rojo antes de cada cambio, y la caracterización de DELETE y
   PATCH es la red que necesitan el #2, el #3 y el #7. Coste: un archivo.
2. **El #2 antes que el #3.** El #3 exige acceso al repo; el #2 combinado con el #7 solo
   exige un `GET`. El #2 llega a más atacantes, es 1 sitio y es el único con evidencia de
   reducir el daño de otro hallazgo.
3. **El #7 no tiene contención propia.** Lo único con evidencia es quitar la fuente pública
   de correos (#2). El arreglo definitivo necesita identificar a quien comenta, y el repo no
   lo vincula con ningún usuario.
4. **El #10 es grave pero va tarde.** Crear la aprobación antes de fijar los estados válidos
   (#5) y la autorización del moderador (#6) volvería a meter texto libre y otra regla hecha
   a mano.
5. **El #9 baja de gravedad, no de posición.** Con `'ok'` como aprobado, su síntoma visible
   venía entero del #4. Sigue en su puesto porque depende del #5 e incumple la regla 5.

## Parte B: las tres líneas

1. **Hallazgos devueltos por el agente / comprobados por mí abriendo el archivo:** el agente
   devolvió 20 candidatos (12 defectos, 2 sospechas y 6 propuestas de mejora); tras la
   verificación adversarial quedaron 18 hallazgos en la tabla, 4 propuestas pendientes de
   medir y 3 candidatos excluidos. Comprobados por mí abriendo el archivo: 4.
2. **Hallazgo cuya prioridad no supe decidir, y qué dos criterios entraron en conflicto:**
   el #10 (no existe ninguna operación para aprobar comentarios). Por daño y radio iría
   arriba: todo comentario nuevo queda invisible para siempre (`:326` fija `'pendiente'` y
   ninguna ruta lo cambia). Por precedencia va el décimo: depende de fijar los estados
   válidos (#5) y de autorizar al moderador (#6), y adelantarlo repetiría el texto libre y la
   regla hecha a mano.
3. **Algo que el agente afirmó y no pude verificar, ni a favor ni en contra:** que
   `creado_en` mezcla zonas horarias: el POST lo guarda en UTC (`:323`, `toISOString`) y la
   semilla sin zona (`blog_seeder.ts:488` y siguientes), así que el orden de `:153` podría
   salir desfasado. Nada en el repositorio dice en qué zona está la semilla, y la
   verificación adversarial lo clasificó como «no verificable». Yo tampoco pude comprobarlo.
